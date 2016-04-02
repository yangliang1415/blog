# RPC调研
yangliang @ 20160313


## RPC概念

https://en.wikipedia.org/wiki/Remote_procedure_call
https://www.zhihu.com/question/25536695


web框架是http，RPC是socket

RPC的协议有很多，比如最早的CORBA，Java RMI，Web Service的RPC风格，Hessian，Thrift，甚至Rest API。



1. Netty只是网络通信框架,目的是让你用最少的代码构建出足够支撑网络通信的功能。

2.完成RPC 需要两个协议： 对象序列化协议 和 调用控制协议

常见例子举例：

1.zeroC ICE，拥有自己的网络通信框架 + ICE 调用控制协议和对象序列化协议,同时也涵盖了服务组件的抽象部署等功能。

2.thrift，有自己的网络通信框架+thrift 对象序列化协议+thrift 调用控制协议

3.probuff，只是 对象序列化协议

4.XMLRPC ，jsonRPC，常见的语境是利用HTTP协议作为调用控制协议,XML 和 JSON 作为对象序列化之后的格式。

5.其他的大概也差不多了。

作者：巴多崽
链接：https://www.zhihu.com/question/25536695/answer/36197588
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。




## dubbo

Dubbo is a distributed service framework enpowers applications with service import/export capability with high performance RPC.

https://github.com/alibaba/dubbo


构建项目的一些疑问:

* maven打包
* pom.xml文件
* eclipse IDE 


问题1：打包区别
dubbo-demo-consumer-2.5.4-SNAPSHOT-assembly.tar.gz
dubbo-demo-consumer-2.5.4-SNAPSHOT.jar

任务启动




## sofa-pbrpc
A light-weight RPC implement of google protobuf RPC framework.

sofa-pbrpc(sofa protobuf-based rpc)是使用Boost::Asio实现的基于Google Protocol Buffers RPC框架的网络通信库

https://github.com/baidu/sofa-pbrpc/wiki


## protocal buffer

Protocol buffers are a flexible, efficient, automated mechanism for serializing structured data – think XML, but smaller, faster, and simpler. 


## Finagle
Finagle is an extensible RPC system for the JVM, used to construct high-concurrency servers

http://twitter.github.io/scala_school/zh_cn/index.html




lt6s nchnch



