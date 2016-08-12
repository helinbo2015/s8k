#### 0. 写在最前面  


#### 1. k8s架构介绍  


#### 2. k8s各组件介绍  

```

ReplicationController {
	Spec {
		Replicas: 1
	}
	Status {
		Replicas: 0
	}
}
 
``` 
```
Pod {
	CreationTimestamp:
	Spec {
		NodeName: "9.100.22.81"
	} 
	Status {
		Phase: Pending
	}
}
```

