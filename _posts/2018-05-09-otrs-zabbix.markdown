---
layout: post
category: "zabbix"
title:  "Zabbix3.x自动生成工单至OTRS"
---

Zabbix server  
OS：CentOS6.8  
软件：Zabbix3.4、Mysql5.6、Python3.6

OTRS server  
OS：CentOS7.4  
软件：otrs-china-1.0.12

<!-- more -->
OTRS的安装请参考官方手册  
[OTRS-installation-guide-2018-01-15.pdf](http://www.dian-tong.com/downloads/Installation%20Manual/OTRS-installation-guide-2018-01-15.pdf.pdf)

## OTRS的配置
用root@localhost的账号登录OTRS进行配置  
### 服务人员管理  
1，服务人员  
新建3个服务人员，employeeS，employeeN，manager。  
2，角色  
新建3个角色，ServerEngineer，NetworkEngineer，GeneralManager。  
3，组  
新建3个组，ServerG，NetworkG，ManagerG。  
4，将各个模块关联起来  
employeeS->ServerEngineer->ServerG  
employeeN->NetworkEngineer->NetworkG  
manager->GeneralManager->ManagerG  
5，权限  
各个组、角色、服务人员的权限一一对应  
另外manager拥有其他的所有权限  

### 队列设置
1，队列  
新建两个队列：服务器队列和网络队列  
服务器队列分配给ServerG，网络队列分配给NetworkG，  
ManagerG由于拥有所有权限，两个队列都能看到

### 用户所属单位管理
1，用户所属单位  
新建单位zabbixC  
2，用户  
新建用户zabbix，并归属至单位zabbixC

### 工单设置
1，自定义字段  
在下拉菜单工单里面选择文本，进入新建自定义字段画面  
名称：EventID  
标记：EventID  
字段顺序：默认  
自定义字段页面配置：选择所有

### 系统管理员
1，web服务  
新建web服务->导入web服务  
从官方下载一个叫GenericTicketConnectorSOAP.yml的文件导入
[GenericTicketConnectorSOAP.yml](https://github.com/justbio/test/blob/GenericTicketConnectorSOAP.yml)

## Zabbix的配置
用admin账号进行zabbix的配置
### Media Types
新建Media Type  
Name：otrs  
Type：script  
Scriptname：ticket.py  
Scriptparameters：{ALERT.SENDTO}，{ALERT.SUBJECT}，{ALERT.MESSAGE}  

### Users
新建两个用户otrs_server和otrs_network
分别添加上面新建的Media：otrs
其中otrs_server的Media的sendto填server  
otrs_network的Media的sendto填network

### Actions
新建Action  
Name：otrs  
Conditions看实际需求填  

Operations选项卡中，  
Defaultsubject填  
```
Problem: {TRIGGER.NAME} 
```
Default message填
```
Problem started at {EVENT.TIME} on {EVENT.DATE}
Problem name: {TRIGGER.NAME}
Host: {HOST.NAME}
Severity: {TRIGGER.SEVERITY}
Eventid: {EVENT.ID}
```
Operations添加  
Send message to users: otrs_server (otrs_server) via otrs  

Recoveryoperations选项卡中，  
Defaultsubject填  
```
Resolved: {TRIGGER.NAME}
```
Default message填
```
Problem has been resolved at {EVENT.RECOVERY.TIME} on {EVENT.RECOVERY.DATE}
Problem name: {TRIGGER.NAME}
Host: {HOST.NAME}
Severity: {TRIGGER.SEVERITY}
Eventid: {EVENT.ID}
```
Operations添加  
Send message to users: otrs_server (otrs_server) via otrs  

标题和内容如果需要修改的话，请保证  
标题以Ploblem/Resolved开头，  
内容的第4行第5行为Severity: {TRIGGER.SEVERITY}和Eventid: {EVENT.ID}  

### 上传脚本
将以下脚本上传至alertscripts文件夹并赋予可执行权限
修改40行和41行的为你自己的manager的帐号密码  
[ticket.py](https://github.com/justbio/test/blob/ticket.py)

### python3设置
安装python3，并且安装python-otrs扩展包  
pip3 install python-otrs

OK这样当故障符合你设置的condition时，  
zabbix就会自动去otrs开一个工单，  
当故障恢复时，zabbix会自动关闭此工单并附上恢复的消息