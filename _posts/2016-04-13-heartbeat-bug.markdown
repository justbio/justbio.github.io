---
layout: post
category: "linux"
title:  "HEARTBEAT的两个小BUG"
---

### 1，heartbeat启动不起来

如果你是用了linux-ha.japan里面的repo文件，Yum安装pacemaker+heartbeat时。 
可能会发现打了service heartbeat start后什么反应也没有。 
其实是这个网站里的软件默认配置写错了，做以下的修改就能解决。

>vi /usr/lib/ocf/resource.d/heartbeat/.ocf-directories  
: ${HA_BIN:=/usr/libexec/heartbeat}  
改成  
: ${HA_BIN:=/usr/lib64/heartbeat}  
cp /usr/libexec/heartbeat/* /usr/lib64/heartbeat/

这个BUG只有用pacemaker+heartbeat才有，pacemaker+corosync时没有。

### 2，IPaddr2 模块不监控网卡状态

>/usr/lib/ocf/resource.d/heartbeat/IPaddr2

这个模块去看一下代码就知道了，他不关心网卡的状态，网卡down掉也不会切换。 
有的网站上说配置一个pingd的资源来监视ping可以解决这个问题。 
但是我做了测试，有一种情况下pingd也无法解决问题，就是网线掉了。。。 

有人会问，网线掉了IP肯定ping不通了啊？pingd怎么会监视不到？ 
我们的linux服务器里面有个叫NetworkManager的服务，默认是开启的， 
当网线掉了的时候，他会去判断网卡的状态，发现是unplug后就会清掉IP信息。 
但是这个家伙经常会自作主张的去修改路由信息，所以做服务器的时候往往会把它关掉。

当NetworkManager不再运行的时候，拔掉网线，在什么也不做的情况下， 
自己ping自己的IP仍然是通的（虚拟机可能在vshpere等的控制下IP会被清掉）， 
而heartbeat的网络监视方法就是自己ping自己，因此，ping就会判断错误。 
不相信的同学可以去找台物理服务器试试看。 

解决方法很简单，让IPaddr2加一个网卡状态判断，如果不是up状态就切换。

>vi /usr/lib/ocf/resource.d/heartbeat/IPaddr2

找到ip_monitor()

在注解文字的直下加上一段代码
```
t=$(ip link show "$NIC" | grep -c "state UP")  
test $t -ne 1 && return $OCF_ERR_PERM
```
 

这样做后检测失败会在crm里留下错误记录，记录累计到一定的次数资源就会强制失效，  
所以还要在资源里加上  
```
primitive vip ocf:heartbeat:IPaddr2 \
    params ip="10.100.1.102" cidr_netmask="24" \
    op monitor interval="10" timeout="20" \
    meta failure-timeout="120"
```
 
定期清理vip的错误信息,这个时间不能设太短,否则备机还没有接管，  
信息就清理掉的话会导致failover失败.