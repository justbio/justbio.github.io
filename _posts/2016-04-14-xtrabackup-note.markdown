---
layout: post
category: "read"
title:  "XTRABACKUP备份笔记"
---

安装
>rpm -Uhv http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm  
yum install xtrabackup

<!-- more -->

备份
>innobackupex /backup/xtrabackup/

预备
>innobackupex --apply-log /backup/xtrabackup/全备

还原
>service mysqld stop  
删除原来的mysql文件夹，创建一个新的mysql文件夹  
innobackupex --copy-back /backup/xtrabackup/全备  
chown –R mysql.mysql /var/lib/mysql

增量备份

先弄个全备
>innobackupex /home/db_backup/

做第一个增量
>innobackupex --incremental /backup/xtrabackup/ --incremental-basedir=/backup/xtrabackup/全备

做第二个增量
>innobackupex --incremental /backup/xtrabackup/ --incremental-basedir=/backup/xtrabackup/增量1

预备增量
>innobackupex --apply-log --redo-only /backup/xtrabackup/全备  
innobackupex --apply-log --redo-only /backup/xtrabackup/全备 --incremental-dir=/backup/xtrabackup/增量1  
innobackupex --apply-log /backup/xtrabackup/增量1 --incremental-dir=/backup/xtrabackup/增量2

还原

>service mysqld stop  
删除原来的mysql文件夹，创建一个新的mysql文件夹  
innobackupex --copy-back /backup/xtrabackup/全备  
chown –R mysql.mysql /var/lib/mysql

最后放个异地备份的脚本
忘了哪里找到的了，侵删
```
#!/bin/sh
# xtrabackup的相关配置
INNOBACKUPEX="innobackupex "
MY_CNF="/etc/my.cnf"
MY_USER="xtrabackup"
MY_PASSWORD="xtrabackup"
MY_SOCKET="/var/lib/mysql/mysql.sock"
# 远程备份机 文件名配置
REMOTE_HOST="testa"
REMOTE_DIR="/backup/xtrabackup"
LOCAL_LSN_FILE="/backup/xtrabackup/.to_lsn_important"
DATE_NAME=`date +%Y-%m-%d-%H-%M-%S`
REMOTE_FILE=$DATE_NAME.tar.gz
LOCK_FILE="/backup/xtrabackup/.mysql.backup.lock"
LOCAL_BACKUP_DIR="/backup/xtrabackup/$DATE_NAME"
# 输出帮助信息
function usage()
{
  echo "Usage:"
  echo "-f db will be backuped fully with this parameter. If not , incrementally. "
}
#防止同时执行两个备份命令，发生冲突
if [ -f $LOCK_FILE ] ;then
  echo 'Mysql backup lockfile is locked!'
  exit 0
fi
full=0
while getopts "fh" arg #选项后面的冒号表示该选项需要参数
do
  case $arg in
  f)
    full=1
    ;;
  h) # 输出帮助信息
    usage
    exit 0
    ;;
  esac
done
echo "backup dir is $REMOTE_DIR/$REMOTE_FILE"
# backup up db to remote host
echo "start to backup db!"`date +%Y-%m-%d-%H-%M-%S`
if [ "$full"x = "1"x ] ;then
# 全量备份
  echo '1' > $LOCK_FILE
  $INNOBACKUPEX --defaults-file=$MY_CNF --user=$MY_USER --password=$MY_PASSWORD --socket=$MY_SOCKET ./ --stream=tar | gzip | ssh $REMOTE_HOST "cat - > $REMOTE_DIR/FULL-$REMOTE_FILE"
  ssh $REMOTE_HOST "cd $REMOTE_DIR;rm -f xtrabackup_checkpoints;tar zxfi $REMOTE_DIR/FULL-$REMOTE_FILE xtrabackup_checkpoints "
  toLSN=$( ssh $REMOTE_HOST "cat $REMOTE_DIR/xtrabackup_checkpoints|grep to_lsn|awk -F= '{gsub(/ /,\"\",\$2);print \$2}'" )
  if [ $toLSN ] ;then
    echo $toLSN > $LOCAL_LSN_FILE
  else
    echo 'no lsn from remote host!please check!'
  fi
else
# 增量备份
  if [ -f $LOCAL_LSN_FILE ] ;then
    toLSN=`cat $LOCAL_LSN_FILE`
  fi
  if [ ! $toLSN ] ;then
    echo 'last LSN is not set !please check!'
    exit 0
  fi
  echo '1' > $LOCK_FILE
  mkdir -p $LOCAL_BACKUP_DIR
  echo "last to lsn is "$toLSN
  $INNOBACKUPEX --parallel=6 --defaults-file=$MY_CNF --user=$MY_USER --password=$MY_PASSWORD --socket=$MY_SOCKET --incremental --incremental-lsn=$toLSN $LOCAL_BACKUP_DIR 2>/tmp/innobackexLog
  toLSN=$( cd $LOCAL_BACKUP_DIR/*; cat xtrabackup_checkpoints|grep to_lsn|awk -F= '{gsub(/ /,"",$2);print $2}' )
  echo "new to lsn is "$toLSN;
  if [ $toLSN ] ;then
    echo $toLSN > $LOCAL_LSN_FILE
    cd $LOCAL_BACKUP_DIR/*;tar zc .|ssh $REMOTE_HOST "cat - > $REMOTE_DIR/$REMOTE_FILE"
    echo "save file to $REMOTE_HOST @ $REMOTE_DIR/$REMOTE_FILE"
  else
    echo 'no lsn from local backup file!delete remote backup file!'$LOCAL_BACKUP_DIR/$REMOTE_FILE
  fi
  echo "remove $LOCAL_BACKUP_DIR"
  rm -rf $LOCAL_BACKUP_DIR
fi
rm -f $LOCK_FILE
echo "end to backup db!"`date +%Y-%m-%d-%H-%M-%S`
```