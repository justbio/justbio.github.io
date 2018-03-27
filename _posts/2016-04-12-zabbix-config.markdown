---
layout: post
category: "zabbix"
title:  "ZABBIX_SERVER.CONF 的性能调优"
---

Zabbix安装完成后，模板里面有一个Template App Zabbix Server，添加到zabbix服务器里。  
过个一两天，查看以下的图表（在Graphs里面）。

>Zabbix cache useage，%free：

指标是剩余的cache容量，如果这个值变的很小，就需要调整zabbix_server.conf里的缓存参数了。  
<!-- more -->
![](../assets/739083-20160412133829363-795227183.png)

>Zabbix Server Performance

主要看等待队列，队列过多的话也需要调整参数  
![](../assets/739083-20160412133830145-1279571376.png)

队列过长时可以根据队列中具体的项目适当的调大下面几个参数  
>StartPollers  
StartPollersUnreachable  
StartTrappers  
StartPingers  
StartDiscoverers  
StartDBSyncers  

Cache不够用时可以根据具体的项目适当的调大下面几个参数

>CacheSize  
HistoryCacheSize  
ValueCacheSize  
TrendCacheSize  
HistoryTextCacheSize  

这个参数用来调整一次housekeeping删除的数据条数，  
如果做了partitioning的话一般不用改

>MaxHousekeeperDelete
 
具体每个参数调成多少不好确定，要根据上面的图表慢慢的调， 
以调到cache够用，队列不长，而且服务器资源能支持的住为准。

我这边一台小型的zabbix服务器，4G内存，250个host，10000个item的参数是这样的，  
仅供参考

>StartPollers=30  
StartPollersUnreachable=3  
StartTrappers=5  
StartPingers=5  
StartDiscoverers=5  
MaxHousekeeperDelete=20000  
CacheSize=64M  
StartDBSyncers=4  
HistoryCacheSize=128M  
ValueCacheSize=128M  
TrendCacheSize=4M  
HistoryTextCacheSize=16M  