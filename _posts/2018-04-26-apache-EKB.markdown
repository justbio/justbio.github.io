---
layout: post
category: "linux"
title:  "EKB6.x+apache模板使用"
---

本文基于上一篇文章的配置修改，  
由于beats模块已经很强大，部分常用的日志已经不需要logstash的处理直接可以用了。    
同样，B安装在客户机上，EK安装在服务端上。  

<!-- more -->

### Elasticsearch配置
编辑配置文件
```
# vi /etc/elasticsearch/elasticsearch.yml
network.host: 192.168.0.10
```
安装插件
```
/usr/share/elasticsearch/bin/elasticsearch-plugin install ingest-geoip
/usr/share/elasticsearch/bin/elasticsearch-plugin install ingest-user-agent
```
重新启动Elasticsearch
```
# systemctl restart elasticsearch
```

### Kibana配置
编辑配置文件
```
# vi /etc/kibana/kibana.yml
server.host: "192.168.0.10"
```
重新启动kibana
```
# systemctl restart kibana
```

### Nginx配置
由于filebeat模板是访问默认端口的，删除原来的端口转发配置
```
# rm /etc/nginx/conf.d/kibana.conf
```
重新启动nginx
```
# systemctl restart nginx
```

### Logstash配置
不要logstash了，关闭
```
# systemctl stop logstash
```

### filebeat配置
编辑配置文件
```
# vi /etc/filebeat/filebeat.yml
#=========================== Filebeat prospectors =============================
#改回默认的false
- type: log
  enabled: false
  paths:
    - /var/log/httpd/access_log
#================================ Outputs =====================================
#还原elasticsearch的转发
output.elasticsearch:
   hosts: ["192.168.0.10:9200"]
   username: "elastic"
   password: "changeme"
#开启kibana的转发
setup.kibana:
   host: "192.168.0.10:5601"
#注掉logstash的转发
#output.logstash:
  #hosts: ["192.168.0.10:5044"]
```
启用模块
```
# filebeat modules enable apache2
# filebeat setup
```
编辑模块文件
```
# vi /etc/filebeat/modules.d/apache2.yml
- module: apache2
  # Access logs
  access:
    enabled: true

    # 这个地方改成accesslog的路径，一定要用方括号括起来
    # 因为是要拿来迭代的，多个日志可以用逗号分隔
    var.paths: ["/var/log/httpd/access_log*"]

  # Error logs
  error:
    enabled: true

    # 这个地方改成errorlog的路径
    var.paths: ["/var/log/httpd/error_log*"]
```
重新启动filebeat
```
# systemctl restart filebeat
```

### kibana页面配置
打开http://192.168.0.10：5601    
点击add log data，  
点击apache logs，    
点击最下面右边的apache2 logs dashboard 

就能看到用来展示的面板了。  


