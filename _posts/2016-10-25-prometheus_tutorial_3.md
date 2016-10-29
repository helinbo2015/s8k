---
layout: post 
title: 聊聊prometheus监控3(监控指标类型使用参考)
category: prometheus 
---

在[聊聊prometheus监控2](https://helinbo2015.github.io/s8k/prometheus/2016/10/21/prometheus_tutorial_2.html)中说过要专门写篇文章说说监控指标类型，说出去的话都来不及后悔，为了说清楚这事，把[prometheus go_client](https://github.com/prometheus/client_golang)的源码翻了一遍。太苦了....  
<!--description-->  

### 1. 指标类型(metric type)介绍  
prometheus 提供四种指标类型(metric type)供监控对象使用，所以在监控对象中定制自己的监控指标前，很有必要清楚理解各个指标类型的应用场景。  

1. Counter: 用于统计的一直增大的指标,例如发生错误数，http请求数等  
2. Gauge： 用于统计动态变化的指标，例如CPU使用量，某个特性的处理时间等  
3. Histogram: 用于需要采样观测的指标，例如http请求响应时间的分布区间(响应时间为1s以内的请求比率数)。 Prometheus sdk为prometheus server提供下列数据。  
  - 指标数值的累加值  
  - 指标的监控次数  
  - 各个监控Bucket里监控指标的监控次数(比如http请求响应时间小于1s的次数(设定了1s的Bucket))  
4. Summary: 用于需要采样观测的指标，例如某个比率的http请求响应时间区间(比如90%的http请求响应时间在什么区间)。 Prometheus sdk为prometheus server提供下列数据  
  - 时间窗口内监控指标数值的累加值  
  - 时间窗口内监控指标的监控次数  
  - 时间窗口内位于Quantile(比率)位置的监控指标数值 (比如10min内有100个http请求响应时间(按数值从小到大排列)，Quanatile=0.9时，返回的就是第90个监控数据(比如是1.5s)，也表示90%的请求响应小于1.5s)  

*看到这里，读者肯定会困惑Histogram和Summary之间没有什么区别啊，貌似用的场景差不多，别着急慢慢往下看，实在忍受不了各种代码分析，直接跳最后看结论吧*  

### 2. 指标类型(metric type)详解  
下面就从源码来说说各种指标类型的用途  

#### 1. Counter:  
   Counter接口的定义如下:  

![Counter_interface.PNG](/s8k/img/Counter_interface.PNG)  

Counter就提供Set(), Inc(), Add()方法，可以看出Counter指标类型就是为一直增大的数值定制的。  
另外Counter监控指标定义的初始化方法: NewCounter()、NewCounterVec()  

#### 2. Gauge:  
   Gauge接口的定义如下:  

![Gauge_interface.PNG](/s8k/img/Gauge_interface.PNG)  

相比Counter，Gauge就除了提供Set(), Inc(), Add()方法，另外还有Dec(), Sub()方法，毫无疑问Gauge指标类型是为动态变化的指标定制的  
另外Gauge监控指标定义的初始化方法: NewGauge()、NewGaugeVec()  

#### 3. Histogram:  
Histogram稍微复杂点，我们先看看Histogram的接口定义:  

![Histogram_interface.PNG](/s8k/img/Histogram_interface.PNG)  

看到有一个Observe()方法，主要用于采集指标数据。  

为了理解Historgram数据的场景，再看看Histogram接口的实现histogram struct给prometheus server提供的数据。（实现Metric接口的Write()方法，用于给Prometheus Server返回数据）  

![histogram_write.PNG](/s8k/img/histogram_write.PNG)  

从上面代码可以看到主要给prometheus server返回了下面的数据。  

* SampleSum: 由h.sumBits转换而来  
* SampleCount: 由h.count转换而来  
* []Buckets{CumulativeCount, UpperBound}: 结合h.upperBounds和h.counts来设定的  

接下来看看h.upperBounds的初始化和Observe（）方法的具体实现，看看上面的数据分别是什么意思  

![histogram_observe.gif](/s8k/img/histogram_observe.gif)  

根据上面的分析，可以看出Histogram给prometheus server返回的数据:  

* SampleSum: 监控指标数值的累加值  
* SampleCount: 监控指标的监控次数  
* Buckets{CumulativeCount, UpperBound}: 各个Bucket的值和落入各Bucket的监控次数  

根据上面的数据，在prometheus server端可以针对观测指标作各种计算和分析。(比如说300ms内http请求响应的占比)。并且可以看出，后续数据的分析，主要集中在prometheus server端(监控对象端仅提供上面的数据)  
另外Histogram监控指标定义的初始化方法: NewHistogram()、NewHistogramVec()  

#### 4. Summary  
Summary的接口定义和Histogram一样，我们直接看他的Write()和Observe()的实现  

![summary_observe.PNG](/s8k/img/summary_observe.PNG)  

每次调用中，把监控指标的数值写入s.hotBuf  

![summary_write.gif](/s8k/img/summary_write.gif)  

根据s.hotBufExpTime切换输出的数据流。所以s.hotBufExpTime+s.headStream就组成了一个滑动的时间窗口。s.hotBufExpTime默认值为2m, 或者用户通过SummaryOpts.MaxAge/SummaryOpts.AgeBuckets来指定  

从上面代码可以看到Summary主要给prometheus server返回了下面的数据  

* sum.SampleCount: 滑动时间窗口内的监控指标的监控次数  
* sum.SampleSum: 滑动时间窗口内的监控指标数值的累加值  
* sum.Quantile:  

  - Quantile(比率):  默认的Quantile值(0.5, 0.9, 0.99)或者用户指定值(通过SummaryOpts.Objectives指定)  
  - Value: sum.SampleCount * Quantile位置的监控指标数值。比如: sum.SampleCount=20, Quantile(比率)=0.5时，Value就是滑动时间窗口内的第10个数(窗口内数据和时间先后无关，一律按数值从小到大排列)  

根据上面的数据，在prometheus server端将会收到总的监控次数，累加的指标数值和指定比率的观测指标数值(比如说90%位置的http请求响应时间是多少)。并且可以看出，主要数据分析集中在prometheus sdk端(监控对象端)，prometheus server主要接收数据聚合结果。  

### 3. 指标类型(metric type)使用总结  

| metric type | 使用参考 |
|--------|--------|
| Counter |仅仅关注指标的单个数值，并且该指标在数值上一直上升。例如发生错误数，http请求数等|
| Gauge |     仅仅关注指标的单个数值，并且该指标在数值上动态变化(有增有减)。例如CPU使用量，某个特性的处理时间等   |
| Histogram/Summary |     1. 如果需要对观测结果做聚合处理，就只能选择Histogram。比如你的服务同时跑在几台机器上，想统计所有服务的http请求响应时间聚合后，90%的响应在在什么时间区间<br/> 2. 使用Histogram时，需要很清楚期待观测指标的数值区间和分布范围，这样才能配置好Buckets。 比如期待看到http请求响应时间为1s以内的比率，那么就可以配置1s的Bucket<br/> 3. 对监控对象的性能影响来说，相对Summary来说，Histogram能小一些。Summary中需要对监控指标结果排序和Quantile计算，而Histogram这些计算工作在prometheus server进行。当比较关心监控对象的性能，建议选择Histogram<br/> 4. 当不需要关心观测指标的数值区间和分布范围，而只想知道观测指标的精确比率的数值，就可以使用Summary  |  

参考链接:  
1. [prometheus go_client](https://github.com/prometheus/client_golang)  
2. [HISTOGRAMS AND SUMMARIES](https://prometheus.io/docs/practices/histograms/)  

