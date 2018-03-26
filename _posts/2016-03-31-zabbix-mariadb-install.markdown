---
layout: post
category: "zabbix"
title:  "ZABBIX3.0+CENTOS7.0+MARIADB5.5监视服务器安装"
---

本次安装采用：

`Centos7.0`
`Zabbix3.0`
`MariaDB5.5`

### 2016/12/2更新

最新的Centos7.1或者Redhat7.1版本在最后启动zabbix-server的时候会报错无法启动

> \#systemctl start zabbix-server  
Job for zabbix-server.service failed.  
See 'systemctl status zabbix-server.service' and 'journalctl -xn' for details.

是因为少一个依赖包，只要安装就好了  
>yum install trousers

<!-- more -->

另外，用Redhat的源会少两个PHP的包php-mbstring，php-bcmath，
请修改成Centos的源或者163源后继续安装

### 原文
系统安装完成后，关闭selinux，防火墙

>[root@testz ~]#vi /etc/selinux/config  
SELINUX=disabled
 
>[root@testz ~]#systemctl stop firewalld  
>[root@testz ~]#systemctl disable firewalld
 
如果安全需求上不能关闭防火墙的话，打开10051端口和http协议

>[root@testz ~]# firewall-cmd --add-port=10051/tcp --zone=public --permanent  
>success  
>[root@testz ~]# firewall-cmd --add-service=http --zone=public --permanent  
>success  
>[root@testz ~]# systemctl restart firewalld  
>重启

### Zabbix安装

下载安装源

>[root@testz ~]# wget http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm  
>[root@testz ~]# rpm -ivh zabbix-release-3.0-1.el7.noarch.rpm  

安装zabbix:

mysql和MariaDB安装zabbix-server-mysql zabbix-web-mysql，
PostgreSQL安装zabbix-server-pgsql zabbix-web- pgsql 。

>[root@testz ~]# yum install zabbix-server-mysql zabbix-web-mysql zabbix-get zabbix-agent
 
安装MariaDB：

YUM安装就可以了

>[root@testz ~]# yum install mariadb-server  
[root@testz ~]# vi /etc/my.cnf.d/server.cnf  
[mysqld]  
character-set-server = utf8  
collation-server = utf8_bin  
skip-character-set-client-handshake  
skip-external-locking  
symbolic-links=0  
innodb_buffer_pool_size = 2048M  
innodb_log_file_size = 512M  
sort_buffer_size = 2M  
innodb_additional_mem_pool_size = 30M  
innodb_log_buffer_size = 8M  
key_buffer_size = 16M  
log-bin=mysql-bin  
expire_logs_days = 7  
server-id=1001  
innodb_data_file_path = ibdata1:1G  
innodb_file_per_table  

启动mariaDB并设开机启动

>[root@testz ~]# systemctl start mariadb  
[root@testz ~]# systemctl enable mariadb

初始数据库导入以及zabbix配置：

数据库中创建zabbix的数据库和账号

>[root@testz ~]# mysql -uroot  
MariaDB [(none)]> create database zabbix;  
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix' ;  
MariaDB [(none)]> exit
 

导入初始表

>[root@testz etc]# zcat /usr/share/doc/zabbix-server-mysql-3.0.1/create.sql.gz | mysql -uroot zabbix
 
配置zabbix

>[root@testz etc]# vi /etc/zabbix/zabbix_server.conf  
DBPassword=zabbix

配置Http

>[root@testz etc]# vi /etc/httpd/conf.d/zabbix.conf  
php_value date.timezone Asia/Shanghai
 
设置开机启动

>[root@testz etc]# systemctl start zabbix-server  
[root@testz etc]# systemctl start zabbix-agent  
[root@testz etc]# systemctl start httpd  
[root@testz etc]# systemctl enable zabbix-server  
[root@testz etc]# systemctl enable zabbix-agent  
[root@testz etc]# systemctl enable httpd  

打开浏览器，键入http:/serverIP/zabbix

一路点next，最后出现登录画面，默认账号是Admin，密码是zabbix

至此，zabbix服务器安装完成。
