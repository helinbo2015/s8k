---
layout: post 
title: 聊聊prometheus监控4
category: prometheus 
---

在[聊聊prometheus监控2](https://helinbo2015.github.io/s8k/prometheus/2016/10/21/prometheus_tutorial_2.html)和[聊聊prometheus监控3](https://helinbo2015.github.io/s8k/prometheus/2016/10/25/prometheus_tutorial_3.html)中聊完侵入式引入prometheus client library之后，接下来要轻松一下，说说prometheus server的使用  
<!--description-->  

### 1. prometheus server启动  
强烈推荐使用docker来运行prometheus，真的很方便。  

```Go
docker pull prom/prometheus		// 获取prometheus server镜像
docker run -d -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml -p 9090:9090 prom/prometheus	// 启动prometheus server
```
/tmp/prometheus.yml是prometheus server的配置文件，主要用于配置监控对象信息，数据采集周期，超时时间等  

### 2. prometheus.yml  

```Go
# my global config
global:
  // 全局采集周期设定，对所有job有效。但是会被job中设定覆盖。
  scrape_interval:     15s # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s # By default, scrape targets every 15 seconds.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load and evaluate rules in this file every 'evaluation_interval' seconds.
rule_files:
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape: 
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'resource'  // job的名字

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s    // job的采集时间，覆盖全局的设定
    scrape_timeout: 10s    // job的超时时间

    target_groups:
      - targets: ['10.120.178.236:8081']  //监控对象信息,IP和Port错了就收集不到数据了。
        labels:
          group: 'node-1'
```

看完上面的说明估计再说什么也是多余。但是prometheus server版本(*我用的版本是0.16.1*)不同，配置稍微有一些差异。
具体请参照[CONFIGURATION](https://prometheus.io/docs/operating/configuration/)

### 3. prometheus server故障定位  
当prometheus server是否正常启动了，可以通过下面的2个方法确认。  

- 方法1: curl http://localhost:9090/metrics看看是否有监控数据返回  
- 方法2: web登陆http://localhost:9090查看，具体如下图:  

![proemtheus_status.png](/s8k/img/proemtheus_status.png)

*最后，prometheus.rules有点复杂，但是需要在prometheus server端对数据做处理(比如说数据聚合)时，它有用处了。有兴趣了解的读者可以看这个[DEFINING RECORDING RULES](https://prometheus.io/docs/querying/rules/)。另外打算在下一节聊聊Grafana*  

参考链接:  
1. [CONFIGURATION](https://prometheus.io/docs/operating/configuration/)  
2. [DEFINING RECORDING RULES](https://prometheus.io/docs/querying/rules/)  

