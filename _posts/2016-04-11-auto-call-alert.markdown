---
layout: post
category: "zabbix"
title:  "ZABBIX3.0 自动电话报障"
---

#### 第一种：Pagerduty

网站：[www.pagerduty.com](www.pagerduty.com)  
优点：老牌服务商，稳定  
缺点：贵，英文，网站要FQ  
价格参考(34美元每月才25个电话) *2016年

安装方式：
先去官网注册帐号什么的不用说了，有半个月的免费试用时间。
注册完后会有配置向导,一步一步照着来就能设置好用户，电话号码和报警规则。
途中会有一个Integration Key，记下来，在zabbix中会用到。

转到zabbix服务器，
编辑一个pagerduty用的repo文件
>vi /etc/yum.repos.d/pdagent.repo   
[pdagent]  
name=PDAgent  
baseurl=http://packages.pagerduty.com/pdagent/rpm  
enabled=1  
gpgcheck=1  
gpgkey=http://packages.pagerduty.com/GPG-KEY-RPM-pagerduty

安装pagerduty的agent
>yum install pdagent pdagent-integrations  
systemctl start pdagent  
systemctl enable pdagent  
ln -s /usr/share/pdagent-integrations/bin/pd-zabbix /usr/lib/zabbix/alertscripts/
 
zabbix页面上的设置  
先建media  
![](../assets/739083-20160411155916145-1473715293.jpg)

建user，media标签里添加pagerduty，sendto里面填上刚才记下来的 intergration key
![](../assets/739083-20160411155916395-1672422515.jpg)

建action，pagerduty的action有固定的格式

名字随便取，
标题填：trigger
内容填：
>name:{TRIGGER.NAME}  
id:{TRIGGER.ID}  
status:{TRIGGER.STATUS}  
hostname:{HOSTNAME}  
ip:{IPADDRESS}  
value:{TRIGGER.VALUE}  
event_id:{EVENT.ID}  
severity:{TRIGGER.SEVERITY}  

恢复的标题填：resolve

内容和上面一样
Operation标签里添加Sendmessage 给 pagerduty的用户

 
这样有故障时就会报警给pagerduty里设置的电话号码了。
电话接起来时，按4会把当前的incident变为acknowledge状态，按6变成reslove状态。
Pagerduty在applestore和googlestore都有app上线，app是没有被墙的，可以很方便的使用。

#### 第二种：onealert
#### 2018/2/20 更新
现在onealert部署不用那么麻烦了，直接下载agent，解压执行就可以。  
但是脚本是基于zabbix3.0+写的，zabbix2.x会报错，而且只有单用户  
有时间我去写一个zabbix2.x的单用户版和2.x/3.x多用户版agent。。。

#### 原文
网站：[http://www.onealert.com/](http://www.onealert.com/)  
优点：便宜，中文本土化
缺点：注册以及激活的时候发生了好多次网站出错无法继续的状况，让人不禁对服务质量有所疑问。

但是在实际测试中（一周）没有发生什么问题。客服和技术人员解决问题的态度和速度也不错。
这个网站其实提供了很多功能，电话报警只是其中的一小块，
其他功能我这边有的已经实现了，有的用不到，有兴趣的可以研究研究。

安装方法基本上和pagerduty一样。
注册完按照提示一步一步后创建zabbix的应用，会给出一个appkey，记下来。
下载他们的agent  
[agent](http://www.onealert.com/open/alert/download.jsp)

安装
>tar xvf alert-agent-4.0.1-RC2.tar.gz  
cp -R alert-agent /usr/lib/zabbix/alertscripts  
cd /usr/lib/zabbix/alertscripts  
chown -R zabbix:zabbix alert-agent  
cp alert-agent/plugin/zabbix-plugin/110monitor /usr/lib/zabbix/alertscripts/  
chmod +x /usr/lib/zabbix/alertscripts/110monitor 
 

然后就是zabbix页面的操作
创建media  
![](../assets/739083-20160411155916645-262415315.png)

创建user，media标签里添加刚才创建的media
Send to里面填刚才记下来的appkey  
![](../assets/739083-20160411155916941-1528961984.png)

创建action，也是固定格式
主题为trigger和resolve
内容固定为：
>alarmName:{TRIGGER.NAME}  
entityName:{HOSTNAME}  
entityId:{IPADDRESS}  
value:{TRIGGER.VALUE}    
eventId:{EVENT.ID}  
priority:{TRIGGER.SEVERITY}  
alarmContent:{IPADDRESS} {ITEM.NAME}:{ITEM.VALUE}
 
opration里这样设  
![](../assets/739083-20160411155917301-1961331459.png)

这样有故障时就会报警给onealert里设置的电话号码了。
这个没有按键改incident状态的功能，用户回访的时候我提过这个意见，现在不知道有没有实现了。