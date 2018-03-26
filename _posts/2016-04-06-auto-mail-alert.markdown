---
layout: post
category: "zabbix"
title:  "ZABBIX3.0 自动邮件报障"
---

Zabbix3.0以后，自带的邮件报警支持SSL验证了，
但是仍然没有发送复数个邮箱以及CC，BCC的功能。
<!-- more -->
![](../assets/739083-20160406163536593-1640601560.jpg)

因此，我们还是得用别的方法来实现邮件报障。
实现方法有很多种，我用的是PHPmailer。

下载后解压到文件夹。
>cd /usr/lib/zabbix/alertscripts/  
unzip PHPMailer-master.zip

编写一个shell文件，是用来给zabbix调用的。
>cd /usr/lib/zabbix/alertscripts/  
vi sendmail.sh  
\#!/bin/sh
nohup /usr/lib/zabbix/alertscripts/sendmessage_php.sh "$1" "$2" "$3" >/dev/null 2>&1 &

编写一个php文件，就是上面的sendmessage_php.sh
>vi sendmessage_php.sh  

[sendmessage_php.sh](https://github.com/justbio/test/blob/master/sendmessage_php.sh)

编辑地址簿文件，不同的地址用逗号隔开，to和cc用##隔开
>mkdir /usr/lib/zabbix/alertscripts/addresslist  
vi test  
xxxx@xxx.com##yyyy@yyy.com,zzzz@zzz.com

回到zabbix网页
Administrator -> media type -> create media type
如下设置
![](../assets/739083-20160406163537015-1816305491.jpg)

*在zabbix3.0以前，PARAMETER是固定的，不用设置也不能更改。3.0默认值变成空值，需要自己设置。
Administrator -> user -> create user
名字我这里取testuser，在media标签中点add，如下填写
![](../assets/739083-20160406163537468-2104989576.png)

Send to 填的是刚才建的地址簿的文件名
Pemmision标签当然是选择zabbixadministrators
Configuration -> Action ->create action
名字我这里取testaction，在action和conditions标签按实际需求填写，
Operations中点New，如下填写
![](../assets/739083-20160406163537984-960197297.jpg)

*user我这里涂掉了，应该填的是刚才新建的testuser
这样，自动邮件就配置完成了，让我们测试一下脚本
>./sendmail.sh test testsubject testmessage

第一个参数是地址簿的文件名，第二个参数是标题，第三个参数是正文
地址簿test里的所有人都会收到标题为testsubject，正文为testmessage的邮件。