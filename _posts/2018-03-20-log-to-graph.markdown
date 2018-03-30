---
layout: post
category: "github"
title:  "apache access日志统计做图"
---

`Version:` `1.0.0`  
`OS:` `Linux`  
`Author:` `justbio`

[apache-log-to-graph](https://github.com/justbio/apache-log-to-graph/)  
用途  
-------  
把apache的access日志做成统计图  
> 暂时只支持默认格式的日志  
> LogFormat:"%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined

<!-- more -->

需求
------------
#### Base
* Python3.x

#### 需要安装  
matplotlib
> pip3 install matplotlib  

numpy  
>pip3 install numpy  

Basemap  
> 参考 [basemaptutorial](http://basemaptutorial.readthedocs.io)

#### 需求文件 
geoip2.database
> in [source\GeoLite2-db](https://github.com/justbio/apache-log-to-graph/tree/master/source/GeoLite2-db)

shapefile
> in [source\shapefile](https://github.com/justbio/apache-log-to-graph/tree/master/source/shapefile)

用法
------------
解压到任意路径
编辑conf文件
> vim conf/config.ini
>> logpath=apache_log_path_need_to_analyze  
>> graphpath=path_to_save_graph  

> 如果留空 ,默认为：
>> logpath=/var/log/httpd/   
>> graphpath=./pngs/

执行
> python3 mk_graph.py

Example
------------
![](https://github.com/justbio/apache-log-to-graph/blob/master/pngs/example/day_bar.png?raw=true)
![](https://github.com/justbio/apache-log-to-graph/blob/master/pngs/example/hour_bar.png?raw=true)
![](https://github.com/justbio/apache-log-to-graph/blob/master/pngs/example/URL_bar.png?raw=true)
![](https://github.com/justbio/apache-log-to-graph/blob/master/pngs/example/china_scatter.png?raw=true)
![](https://github.com/justbio/apache-log-to-graph/blob/master/pngs/example/world_scatter.png?raw=true)