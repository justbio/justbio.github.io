---
layout: post
category: "linux"
title:  "Grafana的SQL文"
---

放几个Grafana用的sql文

<!-- more -->

hosts表：  
status=0 为monitored ， status=1 为not monitored ， status=2为templates  
flags = 2为一些内建的vm发现器  
hosts的status分类成moniterd/not moniterd
```
SELECT
  now() as time_sec,
  count(1) as value,
  (CASE status WHEN '1' THEN 'not moniterd' WHEN '0' THEN 'moniterd' ELSE 'unknow' END) as metric
FROM hosts
WHERE status in (0,1) and flags<>2
GROUP BY status
```

items表：
status=0为enabled,status=1为disabled   
flags=0为一般itemflags=1为discovery模板item名称，flags=2为discovery模板item内容flags=4为discovery发现的item  
state=0为正常item，state=1为notsupport item  
items的state分类成monited/not support，不包括disabled掉的item和host
```
SELECT
  now() as time_sec,
  count(1) as value,
  (CASE i.state WHEN '1' THEN 'problem' WHEN '0' THEN 'monited' ELSE 'unknow' END) as metric
FROM items as i, hosts as h
WHERE i.hostid=h.hostid and h.status =0 and h.flags<>2 and i.flags in (0,4) and i.status=0
GROUP BY i.state
```

triggers表： 
status=0为enabled,status=1为disabled 
flags=0为一般item，flags=1为discovery模板trigger名称，flags=2为discovery模板trigger内容，flags=4为discovery发现的trigger
value=0为OK，value=1为Problem
triggers的state分类成Alert/Nomarl，不包括disabled掉的item，host和trigger
```
SELECT
  now() as time_sec,
  count(1) as value,
  (CASE t.value WHEN '1' THEN 'Alert' WHEN '0' THEN 'Nomarl' ELSE 'unknow' END) as metric
FROM hosts h,items i,functions f,triggers t
WHERE  h.hostid=i.hostid and i.itemid=f.itemid and f.triggerid=t.triggerid and h.status =0 and h.flags<>2 and i.flags in (0,4) and t.flags in (0,4) and t.status=0 and i.status=0
```

dashboard模版  
[dashboard](https://github.com/justbio/test/blob/master/New%20dashboard%20Copy-1525247409684.json)