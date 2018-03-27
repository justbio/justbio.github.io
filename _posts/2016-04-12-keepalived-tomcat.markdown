---
layout: post
category: "linux"
title:  "ZABBIX_SERVER.CONF 的性能调优"
---

节点  
>NODE1—————-IP：192.168.0.2  
NODE2—————-IP：192.168.0.3  
VIP—————-IP：192.168.0.10

以下操作在2台机器上都要运行 

先编辑hosts
>vi /etc/hosts
node1    192.168.0.2  
node2    192.168.0.3  

安装keepalived
>yum install keepalived

在node1上
>[root@node1 keepalived-1.1.20]# vim /etc/keepalived/keepalived.conf
```
vrrp_script chk_http_port {
    script "/opt/tomcat.pid" #调用脚本的位置
    interval 30 #检查时间，30秒
    weight 2 #权重值，每一次切换后priority的值即是当前priority-weight得到的数值
}
vrrp_instance VI_1 {
    state BACKUP #备机状态
    nopreempt #不自动failback
    interface eth0
    virtual_router_id 51
    priority 100 #用权重值决定优先权
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_http_port
    }
    virtual_ipaddress {
        192.168.0.10 #漂移的VIP
    }
}
```
 
在node2上  
>[root@node2 ~]# vim /etc/keepalived/keepalived.conf
 
```
vrrp_script chk_http_port {
    script "/opt/tomcat.pid"
    interval 30
    weight 2
}
vrrp_instance VI_1 {
    state BACKUP #备机状态
    interface eth0
    virtual_router_id 51
    priority 99 #这里不同，低于node1
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_http_port
    }
    virtual_ipaddress {
    192.168.10.196
    }
}
```
 
2个节点都要编辑tomcat的监控脚本

>[root@node2 ~]# vim /opt/tomcat.pid  
```
\#!/bin/bash
JAVA_PROCESS=`ps -C java --no-heading| wc -l`
if [ $JAVA_PROCESS -eq 0 ];then
    /data/tomcat5.5/bin/startup.sh start
    sleep 10
    if [ `ps -C java --no-heading| wc -l` -eq 0 ];then
        /etc/init.d/keepalived stop
    fi
fi
```

>chmod 777 /opt/tomcat.pid
 
以上，keepalived的部署就完成了。