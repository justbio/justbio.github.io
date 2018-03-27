---
layout: post
category: "zabbix"
title:  "MARIADB5.5（MYSQL）的PARTITIONING设置 FOR ZABBIX3.0"
---

用zabbix的同学都知道，一台服务器监视几百几千台服务器，一个服务器几十个item，长年下来数据量是很惊人的。  
而zabbix自带的housekeeping功能，默认状态下的删除速度完全跟不上数据增长的速度。 
而一旦把删除量加大，CPU磁盘又开始报警了，严重的情况下，监视都会中断。
<!-- more -->

为了防止这种问题，我们采用了mariaDB（mysql）的partitioning功能。  
partitioning功能的开启最好是在服务器搭建的初期就设计并配置好。  
一旦服务器运行了一段时间，partitioning的配置做是也可以做，但是会比较麻烦。

众所周知，Zabbix里面数据量最大的表就是history和trends， 
我们只针对这两张表做partitioning，其他的交给housekeeper就足够了。 
*zabbix服务器为新装的服务器，除了表结构建立好了以外，没有任何数据。

登入mysql：

>mysql zabbix

按顺序执行以下sql文

 >Alter table history_text drop primary key, add index (id), drop index history_text_2, add index history_text_2 (itemid, id);

 >Alter table history_log drop primary key, add index (id), drop index history_log_2, add index history_log_2 (itemid, id);

创建存储过程partition_create 
``` 
DELIMITER $$  
CREATE PROCEDURE `partition_create`(SCHEMANAME varchar(64), TABLENAME varchar(64), PARTITIONNAME varchar(64), CLOCK int)  
BEGIN  
    DECLARE RETROWS INT;  
    SELECT COUNT(1) INTO RETROWS  
    FROM information_schema.partitions  
    WHERE table_schema = SCHEMANAME AND   
    table_name = TABLENAME AND partition_description >= CLOCK;  
    IF RETROWS = 0 THEN  
        SELECT CONCAT( "partition_create(", SCHEMANAME, ",", TABLENAME, ",", PARTITIONNAME, ",", CLOCK, ")" ) AS msg;  
        SET @sql = CONCAT( 'ALTER TABLE ', SCHEMANAME, '.', TABLENAME, ' ADD PARTITION (PARTITION ', PARTITIONNAME, ' VALUES LESS THAN (', CLOCK, '));' );  
        PREPARE STMT FROM @sql;  
        EXECUTE STMT;  
        DEALLOCATE PREPARE STMT;  
    END IF;
END$$  
DELIMITER ;
```

创建存储过程partition_drop   
```
DELIMITER $$
CREATE PROCEDURE `partition_drop`(SCHEMANAME VARCHAR(64), TABLENAME VARCHAR(64), DELETE_BELOW_PARTITION_DATE BIGINT)
BEGIN
    /*
        SCHEMANAME = The DB schema in which to make changes
        TABLENAME = The table with partitions to potentially delete
        DELETE_BELOW_PARTITION_DATE = Delete any partitions with names that are dates older than this one (yyyy-mm-dd)
    */
    DECLARE done INT DEFAULT FALSE;
    DECLARE drop_part_name VARCHAR(16);
    /*
        Get a list of all the partitions that are older than the date
        in DELETE_BELOW_PARTITION_DATE.  All partitions are prefixed with
        a "p", so use SUBSTRING TO get rid of that character.
    */
    DECLARE myCursor CURSOR FOR
        SELECT partition_name
        FROM information_schema.partitions
        WHERE table_schema = SCHEMANAME AND table_name = TABLENAME AND CAST(SUBSTRING(partition_name FROM 2) AS UNSIGNED) < DELETE_BELOW_PARTITION_DATE;
        DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    /*
       Create the basics for when we need to drop the partition.  Also, create
       @drop_partitions to hold a comma-delimited list of all partitions that
       should be deleted.
    */
    SET @alter_header = CONCAT("ALTER TABLE ", SCHEMANAME, ".", TABLENAME, " DROP PARTITION ");
    SET @drop_partitions = "";
    /*
       Start looping through all the partitions that are too old.
    */
    OPEN myCursor;
    read_loop: LOOP
        FETCH myCursor INTO drop_part_name;
        IF done THEN
            LEAVE read_loop;
        END IF;
        SET @drop_partitions = IF(@drop_partitions = "", drop_part_name, CONCAT(@drop_partitions, ",", drop_part_name));
    END LOOP;
    IF @drop_partitions != "" THEN
        /*
           1. Build the SQL to drop all the necessary partitions.
           2. Run the SQL to drop the partitions.
           3. Print out the table partitions that were deleted.
        */
        SET @full_sql = CONCAT(@alter_header, @drop_partitions, ";");
        PREPARE STMT FROM @full_sql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
        SELECT CONCAT(SCHEMANAME, ".", TABLENAME) AS `table`, @drop_partitions AS `partitions_deleted`;
    ELSE
        /*
           No partitions are being deleted, so print out "N/A" (Not applicable) to indicate
           that no changes were made.
        */
        SELECT CONCAT(SCHEMANAME, ".", TABLENAME) AS `table`, "N/A" AS `partitions_deleted`;
    END IF;
END$$
DELIMITER ;
```

创建存储过程partition_maintenance
```
DELIMITER $$
CREATE PROCEDURE `partition_maintenance`(SCHEMA_NAME VARCHAR(32), TABLE_NAME VARCHAR(32), KEEP_DATA_DAYS INT, HOURLY_INTERVAL INT, CREATE_NEXT_INTERVALS INT)
BEGIN
    DECLARE OLDER_THAN_PARTITION_DATE VARCHAR(16);
    DECLARE PARTITION_NAME VARCHAR(16);
    DECLARE OLD_PARTITION_NAME VARCHAR(16);
    DECLARE LESS_THAN_TIMESTAMP INT;
    DECLARE CUR_TIME INT;
    CALL partition_verify(SCHEMA_NAME, TABLE_NAME, HOURLY_INTERVAL);
    SET CUR_TIME = UNIX_TIMESTAMP(DATE_FORMAT(NOW(), '%Y-%m-%d 00:00:00'));
    SET @__interval = 1;
    create_loop: LOOP
        IF @__interval > CREATE_NEXT_INTERVALS THEN
            LEAVE create_loop;
        END IF;
        SET LESS_THAN_TIMESTAMP = CUR_TIME + (HOURLY_INTERVAL * @__interval * 3600);
        SET PARTITION_NAME = FROM_UNIXTIME(CUR_TIME + HOURLY_INTERVAL * (@__interval - 1) * 3600, 'p%Y%m%d%H00');
        IF(PARTITION_NAME != OLD_PARTITION_NAME) THEN
            CALL partition_create(SCHEMA_NAME, TABLE_NAME, PARTITION_NAME, LESS_THAN_TIMESTAMP);
        END IF;
        SET @__interval=@__interval+1;
        SET OLD_PARTITION_NAME = PARTITION_NAME;
    END LOOP;
    SET OLDER_THAN_PARTITION_DATE=DATE_FORMAT(DATE_SUB(NOW(), INTERVAL KEEP_DATA_DAYS DAY), '%Y%m%d0000');
    CALL partition_drop(SCHEMA_NAME, TABLE_NAME, OLDER_THAN_PARTITION_DATE);
END$$
DELIMITER ;
```

创建存储过程partition_verify
```
DELIMITER $$
CREATE PROCEDURE `partition_verify`(SCHEMANAME VARCHAR(64), TABLENAME VARCHAR(64), HOURLYINTERVAL INT(11))
BEGIN
    DECLARE PARTITION_NAME VARCHAR(16);
    DECLARE RETROWS INT(11);
    DECLARE FUTURE_TIMESTAMP TIMESTAMP;
    SELECT COUNT(1) INTO RETROWS
    FROM information_schema.partitions
    WHERE table_schema = SCHEMANAME AND table_name = TABLENAME AND partition_name IS NULL;
    IF RETROWS = 1 THEN
        SET FUTURE_TIMESTAMP = TIMESTAMPADD(HOUR, HOURLYINTERVAL, CONCAT(CURDATE(), " ", '00:00:00'));
        SET PARTITION_NAME = DATE_FORMAT(CURDATE(), 'p%Y%m%d%H00');
        SET @__PARTITION_SQL = CONCAT("ALTER TABLE ", SCHEMANAME, ".", TABLENAME, " PARTITION BY RANGE(`clock`)");
        SET @__PARTITION_SQL = CONCAT(@__PARTITION_SQL, "(PARTITION ", PARTITION_NAME, " VALUES LESS THAN (", UNIX_TIMESTAMP(FUTURE_TIMESTAMP), "));");
        PREPARE STMT FROM @__PARTITION_SQL;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
    END IF;
END$$
DELIMITER ;
```

创建存储过程partition_maintenance_all  
这里的格式是  
>CALL partition_maintenance('[zabbix_db_name]', '[table_name]', [days_to_keep_data], [hourly_interval], [num_future_intervals_to_create])

后面3个参数分别为：数据保持的日数，时间间隔（一小时一个分区就是1，1天一个分区就是24，依此类推），预创建的分区数。  
里面的数字请根据需要调整。

```
DELIMITER $$
CREATE PROCEDURE `partition_maintenance_all`(SCHEMA_NAME VARCHAR(32))
BEGIN
    CALL partition_maintenance(SCHEMA_NAME, 'history', 35, 24, 14);
    CALL partition_maintenance(SCHEMA_NAME, 'history_log', 35, 24, 14);
    CALL partition_maintenance(SCHEMA_NAME, 'history_str', 35, 24, 14);
    CALL partition_maintenance(SCHEMA_NAME, 'history_text', 35, 24, 14);
    CALL partition_maintenance(SCHEMA_NAME, 'history_uint', 35, 24, 14);
    CALL partition_maintenance(SCHEMA_NAME, 'trends', 365, 24, 14);
    CALL partition_maintenance(SCHEMA_NAME, 'trends_uint', 365, 24, 14);
END$$
DELIMITER ;
```

然后执行  
>mysql> CALL partition_maintenance_all('zabbix');  

然后关闭zabbix页面上history和trends的housekeeping  
![](../assets/739083-20160412102118504-1297185656.jpg)

最后，由于我们在本列中只创建了14个分区，到第15天数据就没地方写了。  
所以把以下脚本放到crontab里面，每天或者每周执行一下就可以了。

>\#!/bin/sh  
mysql -u[username] -p[password] zabbix -e "CALL partition_maintenance_all('zabbix');"
 

另外，刚才我们创建的存储过程也可以单独使用

>partition_create - 创建一个表的分区  
partition_drop - 删除给出时间戳以前的分区  
partition_verify - 验证表是否开启了分区，如果没有，则创建一个单个的分区
用法分别为  

>partition_create(SCHEMANAME varchar(64), TABLENAME varchar(64), PARTITIONNAME varchar(64), CLOCK int)
 
>partition_drop(SCHEMANAME VARCHAR(64), TABLENAME VARCHAR(64), DELETE_BELOW_PARTITION_DATE VARCHAR(64))
 
>partition_verify(SCHEMANAME VARCHAR(64), TABLENAME VARCHAR(64), HOURLYINTERVAL INT(11))
 

参考文档：

Docs/howto/mysql partition  
https://www.zabbix.org/wiki/Docs/howto/mysql_partition