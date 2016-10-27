---
layout: post 
title: 聊聊prometheus监控1
category: prometheus 
---
### 1. 背景介绍  
前段时间折腾了一阵K8S性能调优，既然要调优首先是找到性能的瓶颈点，方法不外乎下面两点。  
1. 使劲看代码，画出数据各种流程图，架构图，看看什么地方有性能提升空间(比如说某个地方能不能加点Cache)  
2. 利用监控工具把各个部分的处理时间(比如说API请求时延)提取出来，看看时间都消耗在哪了，然后再考虑优化方法。  
关于都看了什么代码，做了哪些优化，不是这篇文章的重点。主要是想说说第二点中的监控工具prometheus，不管是程序调优，还是系统监控，都是一套不错的解决方案。    
<!--description-->  

### 2. prometheus架构  
首先说说prometheus的优势。  
1. prometheus是一套完整的监控解决方案，集监控，显示，告警等各种功能。且外围成熟的组件很多。  
2. 项目开源、并且使用广泛。目前热门的开源项目(比如: K8S、Docker等)中都集成了prometheus。  
3. prometheus支持多维度时间序列数据模型(时间序列数据由指标名(metric name)+指标标签(tags: key/value)组成)，格式同NOSQL数据库opentsdb的时间序列数据。  

然后看看prometheus的架构(*我只把我使用过的架构画出来*)  

![promtheus_arch.png](/s8k/img/promtheus_arch.png)

- 针对长时间运行程序(比如说K8S)，尽量采用简单方案: 直接在监控对象中集成prometheus client library来暴露监控数据，promehteus server采用PULL的方式从定时从监控对象中提取数据。  
- 针对短期任务(比如说一个shell job)，通过prometheus server来PULL数据的话，可能短期任务已经结束了，而PULL请求还没来。这时就需要采用Pushgateway的方案。  
- 在prometheus server中存储的数据，需要用比较漂亮的形式展示出来，就需要使用Grafana或者Promdash(个人建议用Grafana)  

Grafana的显示效果:

![grafana.PNG](https://camo.githubusercontent.com/912714b2aaa96d5ddfa54f1f0c7b7c66f078495a/687474703a2f2f67726166616e612e6f72672f6173736574732f696d672f73746172745f706167655f62672e706e67)  

### 3. 选择prometheus的一些考虑  
这个话题有点大，只能说一些个人的理解。不一定到位，欢迎大家指正。  
1. 首先不要认为要使用prometheus，就需要在监控对象中引入prometheus client library。而应该清楚自己的需求，找一下是否有现成的第三方exporter可以用。比如说监控节点的CPU/Mem，就可以直接用node exporter就可以。如果有现成的exporter可以用，选择用prometheus当然是不二选择。(*[支持prometheus的exporter](https://prometheus.io/docs/instrumenting/exporters/)*)  
2. 实在没有现成的exporter，这时候要考虑在监控对象中导入prometheus client library。目前prometheus client library支持比较好的语言有`Go`、`java`、`python`、`Ruby`。如果你的监控对象是用这些语言写的，那么用prometheus的话能省不少事。(*[prometheus client library支持的语言](https://prometheus.io/docs/instrumenting/clientlibs/)*)  
3. 如果你的监控信息是纯数值时间序列(比如说某个时间的用户请求数等)，prometheus擅长干这事。但是如果需要特别细节的数据(比如说注册请求的详细用户数据等),这样的数据就不太符合prometheus的数据模型(时间序列)，这时候就要慎用prometheus.  

补充一句: 目前prometheus client library支持最好的语言是`Go`，所以现在热门的`Go`语言项目`K8S`、`Docker`、`ETCD`中支持prometheus就不足为怪了。  
