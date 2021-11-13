# kubernetes学习之service、deployment、pod的关系

deployment根据Pod的标签关联到Pod,是为了管理pod的生命周期

service根据Pod的标签关联到pod,是为了让外部访问到pod,给pod做负载均衡

需要注意:

  deployment控制器关联的Pod,Pod的name和hostname(如果不手动指定)就是deployment控制器的Name

  StatefulSet控制器关联的Pod,Pod的Name和Hostname(如果不手动指定)就是StatefulSet控制器的Name + 序号





# K8S内微服务之间访问方式



对于部署于K8S中的微服务，一般大家都会思考如何将服务暴露到公网，让外部用户访问，这种场景一般有三种方式：NodePort，ClusterIP + ingress， Loadbalancer。今天我们不讨论这三种外部访问的方法，而是说一说k8s内各个微服务间的访问方式。

在一篇博客中曾经看到，可以在pod中通过环境变量获取到对应service的IP和Port。这个是k8s内部逻辑，在同一个namespace下，如果有service被创建，对于部署在该namespace下的所有Pod都会有对应service的host和port的全局变量生成，这样pod可以访问到该service。但是这样的做法有个弊端，就是如果你只是想在Pod启动的时候取到某个service的host和port，那么Pod的启动必须放在service配置成功之后，这样非常不方便。

经过尝试，其实k8s给每个内部微服务提供了一个内部使用的host，就是servicename.namespace，注意两个单词之间有一个点。也就是说，如果你想访问部署在A namespace下的B service 的暴露端口C Port，你就可以在代码中直接访问[http://B.A:C](https://links.jianshu.com/go?to=http%3A%2F%2FB.A%3AC)。如果是部署到默认namespace下的服务，A为“default”。

# k8s 之如何从集群外部访问内部服务的三种方法

