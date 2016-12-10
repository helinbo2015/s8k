---
layout: post 
title: 聊聊prometheus监控5
category: prometheus 
---

为什么要推荐使用Grafana，而不是promdash。因为Grafana颜值高。  

##### 1. Grafana启动  
同样也推荐使用docker来运行Grafana，谁用谁知道  
<!--description-->  

```golang
docker pull grafana/grafana		// 获取Grafana镜像
docker run -d -p 3000:3000 grafana/grafana	// 启动Grafana
```

##### 2. Grafana使用  
web登陆http://localhost:3000后，admin/admin就登陆到Grafana的主页面了。  
Grafana的使用主要包括两点。  
1. 添加数据源(`datasource`)(就是从哪个prometheus server获取数据用于显示)  
2. 创建数据展示图(`Graph`)(就是把你需要的数据展示出来)  


数据源添加方式如下(具体请参照[官网](http://docs.grafana.org/datasources/prometheus/)):  

![grafana_datasource.png](/s8k/img/grafana_datasource.png)


数据展示图创建如下(具体请参照[官网](http://docs.grafana.org/datasources/prometheus/)):  

![grafana_graph.png](/s8k/img/grafana_graph.png)

另外想提一下的是: Grafana各种显示配置可以导出，方便后续使用和分享。官网上就有各种开源软件的监控展示配置，比如[kubernetes](https://grafana.net/dashboards/315)  

参考链接:  
1. [Grafana使用](http://docs.grafana.org/datasources/prometheus/)  
2. [Dashboards](https://grafana.net/dashboards)  
