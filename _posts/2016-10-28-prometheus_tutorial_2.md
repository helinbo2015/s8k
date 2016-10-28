---
layout: post 
title: 聊聊prometheus监控2
category: prometheus 
---
看完[聊聊prometheus监控1](https://helinbo2015.github.io/s8k/prometheus/2016/10/27/prometheus_tutorial_1.html)后，对prometheus监控方案应该有个了解了,现在肯定跃跃欲试想把他用起来了。  
在这节里我打算用贴代码(`Go`语言)的方式说明一下怎么在监控对象中导入prometheus。  
<!--description-->  

引入prometheus监控主要包括4点  

1. 监控指标定义  
2. 监控指标注册  
3. 监控指标收集  
4. 监控服务发布  

上面4点在代码中具体反映如下:  

```Go
package main

import (
    "flag"
    "math/rand"
    "net/http"
    "time"

    "github.com/prometheus/client_golang/prometheus"
)

var (
    addr              = flag.String("listen-address", ":8080", "The address to listen on for HTTP requests.")
    uniformDomain     = flag.Float64("uniform.domain", 200, "The domain for the uniform distribution.")
)

var (
    // Create a summary to track fictional interservice RPC latencies for three
    // distinct services with different latency distributions. These services are
    // differentiated via a "service" label.
    rpcDurations = prometheus.NewSummaryVec(    **1. 监控指标定义**
        prometheus.SummaryOpts{
            Name: "rpc_durations_microseconds",
            Help: "RPC latency distributions.",
        },
        []string{"service"},
    )
)

func init() {
    // Register the summary and the histogram with Prometheus's default registry.
    prometheus.MustRegister(rpcDurations)        **2. 监控指标注册**
}

func main() {
    flag.Parse()

    // Periodically record some sample latencies for service uniform
    go func() {
        for {
            v := rand.Float64() * *uniformDomain
            rpcDurations.WithLabelValues("uniform").Observe(v)        **3. 监控指标收集**
            time.Sleep(1 * time.Second)
        }
    }()

    ...

    // Expose the registered metrics via HTTP.
    http.Handle("/metrics", prometheus.Handler())        **4. 监控服务发布**
    http.ListenAndServe(\*addr, nil)
}
```

关于监控指标定义部分，prometheus支持的指标类型，每种类型的意义，如何使用稍微有点复杂，打算专门写一节来说明一下。

参考链接:
1. [源码](https://github.com/prometheus/client_golang/blob/master/examples/random/main.go)  
