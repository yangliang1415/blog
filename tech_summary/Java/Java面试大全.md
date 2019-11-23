## 基础

问：高并发下如何做到安全的修改同一行数据，乐观锁和悲观锁是什么？

答：

先说乐观锁和悲观锁的机制：

1. 乐观锁是一种思想，具体实现是，表中有一个版本字段，第一次读的时候，获取到这个字段。处理完业务逻辑开始更新的时候，需要再次查看该字段的值是否和第一次的一样。如果一样更新，反之拒绝。之所以叫乐观，因为这个模式没有从数据库加锁。
2. 悲观锁是读取的时候为后面的更新加锁，之后再来的读操作都会等待。这种是数据库锁

参考：

https://blog.csdn.net/truelove12358/article/details/54963791

http://www.hollischuang.com/archives/934

https://www.zhihu.com/question/29420056



问：join、sleep、await 、yield

答：

sleep：

await：

yield的意思是放手，放弃：暂停当前线程，以便其他线程有机会执行，不过不能指定暂停的时间，并且也不能保证当前线程马上停止。yield方法只是将Running状态转变为Runnable状态

join：等待调用join方法的线程结束，再继续执行。父线程等待子线程执行完成后再执行，换句话说就是将异步执行的线程合并为同步的线程

参考：https://www.cnblogs.com/paddix/p/5381958.html



问：单例模式

答：http://wudashan.cn/2017/03/20/Singleton-Pattern/



问：JDK 常用数据结构

答：

arraylist和linkedlist底层数据结构区别：arraylist是数组；linkedlist是链表

参考：http://fangjian0423.github.io/categories/jdk/



问：HashMap的并发问题；LinkedHashMap应用；TreeMap的实现原理

答：HashMap非线程安全，允许key为null。ConcurrentHashMap线程安全。TreeMap有序

参考：

http://www.jasongj.com/java/concurrenthashmap/

https://tech.meituan.com/java-hashmap.html

https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/index.html



问：Arrays.sort和Collections.sort实现原理

答：Collections.sort方法底层会调用Arrays.sort方法，底层实现都是TimeSort实现的。TimSort算法就是找到已经排好序数据的子序列，然后对剩余部分排序，然后合并起来



问：Object有哪些基本方法。**protected** native Object clone()方法作用

答：共11个方法。

hashCode and equals：有可能2个不相等的对象有相同的hashcode，即碰撞。对象的哈希码是通过将该对象的内部地址转换成一个整数来实现的。如果2个对象的equals方法相等，那么他们的hashCode值也必须相等，反之，如果2个对象hashCode值相等，但是equals不相等，这样会影响性能，所以还是建议2个方法都一起重写。

参考：

https://fangjian0423.github.io/2016/03/12/java-Object-method/



问：Java 中实现同步的几种方式

答：

synchronized、ReentrantLock、wait/notifyAll、Condition

并发包下的工具类：Semaphore，ThreadLocal、AbstractQueuedSynchronizer等

参考：http://fangjian0423.github.io/2016/04/18/java-synchronize-way/



问：Java 中常用关键词含义。synchronized，transient，volatile

答：

synchronized：作用在类方法、静态方法等不同的效果分析

transient：序列化将某些字段跳过

volatile：保证线程可见变量的修改



问：Serializable接口的理解(Serializable与Externalizable 的区别)

答：Serializable是默认的，不安全等。Externalizable可以自定义。

参考：

http://www.blogjava.net/jiangshachina/archive/2012/02/13/369898.html

https://www.ibm.com/developerworks/cn/java/j-lo-serial/index.html



问：Cloneable接口实现原理，浅拷贝or深拷贝

答：对象存在堆栈哪里。堆栈区别。



问：动态代理的几种方式

答：invoke神器。运行时生成代理类的动态代理方式



问：反射的原理，反射创建类实例的三种方式；Class.forName和ClassLoader区别

答：https://www.zhihu.com/question/24304289

在Java中的反射机制是指在运行状态中，对于任意一个类都能够知道这个类所有的属性和方法；并且对于任意一个对象，都能够调用它的任意一个方法；这种动态获取信息以及动态调用对象方法的功能成为Java语言的反射机制。

编译时类型和运行时类型

ref：http://blog.csdn.net/xu__cg/article/details/52877573



问：Java NIO使用和理解；简述NIO的最佳实践，比如netty，mina

答：

- IO是面向流的，NIO是面向缓冲区的
- 阻塞与非阻塞IO
- Java NIO的选择器允许一个单独的线程来监视多个输入通道




问：String，Stringbuffer，StringBuilder的区别

答：运行速度：StringBuilder > StringBuffer > String。String为字符串常量，而StringBuilder和StringBuffer均为字符串变量

线程安全：StringBuilder是线程不安全的，而StringBuffer是线程安全的

```
String：适用于少量的字符串操作的情况
StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况
StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况
```

参考：https://www.cnblogs.com/su-feng/p/6659064.html



问：线程安全的概念

答：　　

当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么这个类就是线程安全的。

参考：https://www.cnblogs.com/tcming/p/6711506.html



问：线程池。种类、区别、使用场景、实现原理、调度过程、如何调优、线程池的最大线程数目如何确定

答：

参考：http://fangjian0423.github.io/2016/03/22/java-threadpool-analysis/



问：线程6状态(NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED)

答：

BLOCKED：同步块synchronized

WAITING、TIMED_WAITING：wait、notify







##并发相关

问：ThreadLocal原理及使用注意点

答：可以总结为一句话：**ThreadLocal的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。**

参考：

https://www.zhihu.com/question/23089780



问：Synchronized和Lock的区别。Synchronized的原理。自旋锁，偏向锁，轻量级锁，可重入锁，公平锁和非公平锁等概念。

答：

* Lock不是Java语言内置的，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问；
* Lock和synchronized有一点非常大的不同，采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用；而Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。
* Lock可以提高多个线程进行读操作的效率。

参考：https://www.cnblogs.com/baizhanshi/p/6419268.html





问：如果让你实现一个并发安全的链表，你会怎么做

答：



问：ConcurrentLinkedQueue和LinkedBlockingQueue的用处和不同之处

答：



问：Condition接口及其实现原理

答：



问：concurrent包中使用过哪些类？分别说说使用在什么场景？为什么要使用？

答：



countdowlatch和cyclicbarrier的用法，以及相互之间的差别?

用过哪些原子类，他们的参数以及原理是什么

简述AQS的实现原理
LockSupport工具
Fork/Join框架的理解
jdk8的parallelStream的理解
分段锁的原理,锁力度减小的思考；CAS是什么，有何问题（ABA问题的解决，如加入修改次数、版本号）



## JVM相关

问：Java 内存泄漏（不再会被使用的对象的内存不能被回收，就是内存泄露）

答：

在Java程序中，我们通常使用new为对象分配内存，而这些内存空间都在堆（Heap）上。

内存的释放，也即清理那些不可达的对象，是由GC决定和执行的，所以GC会监控每一个对象的状态，包括申请、引用、被引用和赋值等。**释放对象的根本原则就是对象不会再被使用**。

场景1：对象都是有生命周期的，有的长，有的短，**如果长生命周期的对象持有短生命周期的引用，就很可能会出现内存泄露**。

```java
public class Simple {
    Object object;
    public void method1(){
        object = new Object();
        // ... 其他代码
      	// object = null; 避免内存泄漏
    }
}
// new的Object对象在method1结束后无法释放。整个类 over 后是会被释放的。


public E pop() {
  if(size == 0)
  	return null;
  else{
  	E e = (E) elementData[--size];
    elementData[size] = null;  // 及时将无用对象标记为可被清理的对象
    return e;
  }
}
```

ref：

http://blog.csdn.net/anxpp/article/details/51325838

http://www.cnblogs.com/handsomeye/p/5840869.html



问：类的实例化顺序

答：（静态变量、静态初始化块）>（变量、初始化块）>构造器。



JVM内存分代
Java 8的内存分代改进
JVM垃圾回收机制，何时触发MinorGC等操作
jvm中一次完整的GC流程（从ygc到fgc）是怎样的，重点讲讲对象如何晋升到老年代，几种主要的jvm参数等
你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms，g1
新生代和老生代的内存回收策略
Eden和Survivor的比例分配等
深入分析了Classloader，双亲委派机制
JVM的编译优化
对Java内存模型的理解，以及其在并发中的应用
指令重排序，内存栅栏等
OOM错误，stackoverflow错误，permgen space错误
JVM常用参数
tomcat结构，类加载器流程



问：volatile的语义，它修饰的变量一定线程安全吗  

答：不是



g1和cms区别,吞吐量优先和响应优先的垃圾收集器选择
说一说你对环境变量classpath的理解？如果一个类不在classpath下，为什么会抛出ClassNotFoundException异常，如果在不改变这个类路径的前期下，怎样才能正确加载这个类？
说一下强引用、软引用、弱引用、虚引用以及他们之间和gc的关系

## Spring

问：Spring的核心特性就是IOC和AOP，实现原理

答：

IOC（Inversion of Control），即“控制反转”。另一种说法叫DI（Dependency Injection），即依赖注入。工厂模式，通过sessionfactory去注入实例

AOP（Aspect-Oriented Programming），即“面向切面编程”。代理模式。

ref：https://www.cnblogs.com/gaopeng527/p/5290997.html



Spring的beanFactory和factoryBean的区别
为什么CGlib方式可以对接口实现代理？
RMI与代理模式
Spring的事务隔离级别，实现原理
对Spring的理解，非单例注入的原理？它的生命周期？循环注入的原理，aop的实现原理，说说aop中的几个术语，它们是怎么相互工作的？
Mybatis的底层实现原理
MVC框架原理，他们都是怎么做url路由的
spring boot特性，优势，适用场景等
quartz和timer对比
spring的controller是单例还是多例，怎么保证并发的安全

## 分布式相关

Dubbo的底层实现原理和机制



问：描述一个服务从发布到被消费的详细过程

答：Invoker，Exporter，Invocation，Protocol

暴露服务

- 服务到Invoker：首先ServiceConfig类拿到对外提供服务的实际类ref(如：HelloWorldImpl),然后通过ProxyFactory类的getInvoker方法使用ref生成一个AbstractProxyInvoker实例，
- Invoker到Exporter(Dubbo协议)：Dubbo协议的Invoker转为Exporter发生在DubboProtocol类的export方法，它主要是打开socket侦听服务，并接收客户端发来的各种请求，通讯细节由Dubbo自己实现。

消费服务

* 生成Invoker：首先ReferenceConfig类的init方法调用Protocol的refer方法生成Invoker实例(如上图中的红色部分)，这是服务消费的关键。
* Invoker到接口：接下来把Invoker转换为客户端需要的接口(如：HelloWorld)。



参考：

https://yq.aliyun.com/articles/38380

http://blog.csdn.net/qq418517226/article/details/51818769

http://blog.csdn.net/ZuoAnYinXiang/article/details/51345547



问：Dubbo的服务请求失败怎么处理

答：重试



接口的幂等性的概念

消息中间件如何解决消息丢失问题
重连机制会不会造成错误
对分布式事务的理解
如何实现负载均衡，有哪些算法可以实现？
Zookeeper的用途，选举的原理是什么？
数据的垂直拆分水平拆分。
zookeeper原理和适用场景
zookeeper watch机制
redis/zk节点宕机如何处理
分布式集群下如何做到唯一序列号
如何做一个分布式锁： Redis
用过哪些MQ，怎么用的，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗
MQ系统的数据如何保证不丢失
列举出你能想到的数据库分库分表策略；分库分表后，如何解决全表查询的问题。

## 算法&数据结构&设计模式

海量url去重类问题（布隆过滤器）
数组和链表数据结构描述，各自的时间复杂度
二叉树遍历
快速排序
BTree相关的操作
在工作中遇到过哪些设计模式，是如何应用的
hash算法的有哪几种，优缺点，使用场景
什么是一致性hash
paxos算法
在装饰器模式和代理模式之间，你如何抉择，请结合自身实际情况聊聊
代码重构的步骤和原因，如何理解重构到模式？

## 数据库


问：INNODB的行级锁有哪2种，解释其含义

答：

问：mysql索引为什么使用B+树；索引树是如何维护的？

答：数据都在叶节点上，每次查询都需要访问到叶节点。内存硬盘数据访问。



问：MySQL的几种优化

答：

- 使用连接（JOIN）来代替子查询(Sub-Queries)

ref：https://www.cnblogs.com/zhyunfe/p/6209074.html



MySQL InnoDB存储的文件结构

数据库自增主键可能的问题

数据库锁表的相关处理
索引失效场景
数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁

## Redis&缓存相关

Redis的并发竞争问题如何解决了解Redis事务的CAS操作吗
缓存机器增删如何对系统影响最小，一致性哈希的实现
Redis持久化的几种方式，优缺点是什么，怎么实现的
Redis的缓存失效策略
缓存穿透的解决办法
redis集群，高可用，原理
mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据
用Redis和任意语言实现一段恶意登录保护的代码，限制1小时内每用户Id最多只能登录5次
redis的数据淘汰策略

## 网络相关

http1.0和http1.1有什么区别
TCP/IP协议
几种HTTP响应码
当你用浏览器打开一个链接的时候，计算机做了哪些工作步骤
TCP/IP如何保证可靠性，数据包有哪些数据组成
长连接与短连接
Http请求get和post的区别以及数据包格式



问：TIME_WAIT和CLOSE_WAIT的区别

答：

CLOSE_WAIT：连接一端**被动关闭**。在对方连接关闭之后，程序里没有检测到，或者程序压根就忘记了这个时候需要关闭连接，于是这个资源就一直被程序占着。

TIME_WAIT：通信双方建立TCP连接后，**主动关闭连接的一方**就会进入TIME_WAIT状态

存在原因：可靠地实现TCP全双工连接的终止(最终的ACK丢失)；允许老的重复segment在网络中消逝



问：TCP三次握手和四次挥手的流程

答：



## 其他

问：用三个线程按顺序循环打印abc三个字母，比如abcabcabc。

答：

ref: http://mouselearnjava.iteye.com/blog/1949228



问：Linux下IO模型有几种，各自的含义是什么

答：

1、阻塞IO

2、非阻塞IO

3、多路复用IO

4、信号驱动IO

5、异步IO

前四种都是同步，只有最后一种是异步IO



linux利用哪些命令，查找哪里出了问题（例如io密集任务，cpu过度）

maven解决依赖冲突,快照版和发行版的区别

实际场景问题，海量登录日志如何排序和处理SQL操作，主要是索引和聚合函数的应用
实际场景问题解决，典型的TOP K问题
如何从线上日志发现问题
场景问题，有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有10个线程同时调用它，如何做到。
常见的缓存策略有哪些，你们项目中用到了什么缓存系统，如何设计的
设计一个秒杀系统，30分钟没付款就自动关闭交易（并发会很高）
请列出你所了解的性能测试工具
后台系统怎么防止请求重复提交？
有多个相同的接口，我想客户端同时请求，然后只需要在第一个请求返回结果的时候返回给客户端

## 参考

https://github.com/wgd12389/java-server-interview-questions/blob/master/README.md#java%E5%9F%BA%E7%A1%80

https://zhuanlan.zhihu.com/p/32844813