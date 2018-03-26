---
layout: post
category: "zabbix"
title:  "ZABBIX3.0 自动微信报障"
---

本来研究了一段时间短信报障的，但是短信报障需要手机运营商的设备支持，就没有继续下去。
正好发现微信公众号可以报障，完全可以代替短信报警的功能。

首先你需要一个微信公众号，实名认不认证没关系。  
https://qy.weixin.qq.com/ 注册微信企业号  
<!-- more -->

注册企业号和添加用户的流程可以参考以下的文章。
http://www.cnyunwei.com/thread-29593-1-1.html  
*感谢cnyunwei和作者ywjeck，侵删。

作者是用shell写的脚本，我正好在学习python，就用python写了
>vi weixin.py  
[weixin.py](https://github.com/justbio/test/blob/master/weixin.py)

zabbix页面上的设置基本和邮件报警差不多
Administrator -> media type -> create media type
![](../assets/739083-20160406171527390-1572055550.jpg)

*注意这里parameter只有两个，微信是没有标题（subject）的。
*如果用zabbix2.x的版本，这里默认是3个参数，上面的py脚本的sys.argv[2]要改成sys.argv[3]

Administrator -> user -> create user  
名字我这里取wxuser，在media标签中点add，如下填写
![](../assets/739083-20161024153557703-1871443670.jpg)

Send to 填的是微信公众号中通讯录组的编号，可以在组的下拉菜单中找到
Pemmision标签当然是选择zabbixadministrators

Configuration -> Action ->create action  
名字我这里取wxaction，在action和conditions标签按实际需求填写，
Operations中点New，如下填写
![](../assets/739083-20160406171528172-2044156896.jpg)

*user我这里涂掉了，应该填的是刚才新建的wxuser

这样，自动邮件就配置完成了，让我们测试一下脚本
>./weixin.py 2 testmessage

第一个参数是微信公众号中通讯录组的编号，第二个参数是正文
地址簿test里的所有人都会收到正文为testmessage的微信。