# 概述
```
create @ 20160718
update @ 20160719
```

## 1. Dubbo是什么

DUBBO是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，是阿里巴巴SOA服务化治理方案的核心框架，每天为2,000+个服务提供3,000,000,000+次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点。

## 2. 实践一个Example
github地址

## 3. Dubbo架构

Dubbo官网上的文档其实已经写的比较清楚了。这里我就简单搬运过来，其实随着看源代码的不断深入，每次看文档都会有不同的感受。


Dubbo架构的各层说明

* `config配置层`: 对外配置接口。以ServiceConfig，ReferenceConfig为中心，可以直接new配置类，也可以通过spring解析配置声称配置类
* `proxy服务代理层`: 服务接口透明代理，生成服务的客户端Stub和服务器端Skeleton，以ServiceProxy为中心，扩展接口为ProxyFactory
* `registry注册中心层`: 封装服务地址的注册与发现。以服务URL为中心，扩展接口为RegistryFactory，Registry，RegistryService
* `cluster路由层`: 封装多个提供者的路由和负载均衡，并桥接注册中心。以Invoker为中心，扩展接口为Cluster，Directory，Router，LoadBalance
* `monitor监控层`: RPC调用次数和调用时间监控。以Statistics为中心，扩展接口为MonitorFactory，Monitor，MonitorService
* `protocol远程调用层`: 封装RPC调用。以Invocation，Result为中心，扩展接口为Protocol，Invoker，Exporter
* `exchange信息交换层`: 封装请求响应模式，同步转异步。以Request，Response为中心，扩展接口为Exchanger，ExchangeChannel，ExchangeClient，ExchangeServer
* `transport网络传输层`: 抽象mina和netty为统一接口。以Message为中心，扩展接口为Channel，Transporter，Client，Server，Codec
* `serialize数据序列化层`: 可复用的一些工具，扩展接口为Serialization，ObjectInput，ObjectOutput，ThreadPool

各层的关系说明

* 在RPC中，Protocol是核心层，即只要有Protocol＋Invoker＋Exporter就可以完成非透明的RPC调用，然后在Invoker的主过程上Filter拦截点 `TODO:拦截点暂时还没懂`
* Cluster是个外围概念，其目的是将多个Invoker伪装成一个Invoker，这样其他人主要关注Protocol层的Invoker即可
* Proxy层封装了所有接口的透明化代理，而其他层都以Invoker为中心。只有到了暴露给用户使用时，才用Proxy将Invoker转成接口，或将接口实现转成Invoker


Dubbo源码的模块划分

* dubbo-common: 公共逻辑模块
* dubbo－remoting: 远程通讯模块，相当于Dubbo协议的实现
* dubbo－rpc: 远程调用模块，抽象各种协议以及动态代理。只包含一对一的调用
* dubbo－cluster: 集群模块。将多个服务提供方伪装为一个提供方，包括：附在均衡，容错，路由等
* dubbo－register: 注册中心模块
* dubbo－monitor: 监控模块
* dubbo－config: 配置模块，是Dubbo对外的API，用户通过Config使用Dubbo，隐藏Dubbo所有细节
* dubbo-container: 




## 4. 一些基础知识

## 参考资料
```

```



