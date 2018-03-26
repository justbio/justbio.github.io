---
layout: post
category: "zabbix"
title:  "ZABBIX_AGENT的安装配置"
---

## 在Linux上安装zabbix agent

安装

>[root@agtest ~]# yum install http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm  
>[root@agtest ~]# yum install zabbix-agent

修改配置文件
>[root@agtest etc]# vi /etc/zabbix/zabbix_agentd.conf
"<!-- more -->"
修改这几个参数就能连上zabbix服务器了
>Server=[zabbix服务器IP]  
ServerActive= [zabbix服务器IP]  
Hostname= [本机的hostname]
 
如果要用远程命令,打开
>EnableRemoteCommands=1
 
远程命令要用root权限执行的
>AllowRoot=1
 
如果你的远程命令运行要比较久,修改Timeout参数
>Timeout=30
 
要自定义监视项目的
>UserParameter=[key],[脚本]
 
自定义监视无法取到数据时,试试看打开
>UnsafeUserParameters=1  

编辑完后重启agent
>[root@agtest etc]# systemctl start zabbix-agent  
[root@agtest etc]# systemctl enable zabbix-agent

## 在Windows上安装zabbix agent

从官网下载zabbix agent的安装包
>http://www.zabbix.com/downloads/3.0.0/zabbix_agents_3.0.0.win.zip

解压到C:\zabbix下

Bin文件夹里有32位和64位的程序,请安装自己的系统来选择

修改conf文件夹下的zabbix_agentd.win.conf,
修改方法和linux 的一样.
 
打开CMD,输入
>zabbix_agentd.exe --config <your_configuration_file> --install

开启zabbix agent
>zabbix_agentd.exe –start  

设置开机启动
在服务中找到zabbix agent  
![](../assets/739083-20160401111923098-1318511565.png)

启动类型设置为自动  
![](../assets/739083-20160401111924144-237767191.png)