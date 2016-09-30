# RPC调研
yangliang @ 20160313

## 简易RPC
```
几行代码搞定RPC框架(dubbo作者梁飞)
http://javatar.iteye.com/blog/1123915

RPC的概念模型与实现解析，作者实现了一个简易版本的RPC
http://blog.csdn.net/mindfloating/article/details/51477557

RPC开山之作paper: Implementing Remote Procedure Calls
```

### craft.atom.nio源码分析

#### update@20160822
```
IoConnector connector;
connector.connect(ip, port);
```

```
NioFactory.newTcpConnectorBuilder(new NioEchoClientHandler())
.connectTimeoutInMillis(3000)
.build()


// build
NioTcpConnector(handler, config, dispatcher, predictorFactory);

// NioEchoClientHandler
public class NioEchoClientHandler extends AbstractIoHandler
abstract public class AbstractIoHandler implements IoHandler
public interface IoHandler
```


#### update@20160818
```
IoAcceptor acceptor;
acceptor.bind(PORT);
```

```
IoAcceptor acceptor = NioFactory
.newTcpAcceptorBuilder(new NioEchoServerHandler())
.build();

// build
public IoAcceptor build() {
	NioAcceptorConfig config = new NioAcceptorConfig();
	// Good!
	return new NioTcpAcceptor(handler, config, dispatcher, predictorFactory);
}


// NioEchoServerHandler的解释  一个IoHandler
IoHandler handler = new NioEchoServerHandler();
NioEchoServerHandler extends AbstractIoHandler {
	@Override
	public void channelRead(Channel<byte[]> channel, byte[] bytes) {
		channel.write(bytes);
	}
}

abstract public class AbstractIoHandler implements IoHandler {}
/**
 * Handles I/O events fired by <tt>craft-atom-io</tt> series component.
 */
public interface IoHandler {
	void channelOpened(Channel<byte[]> channel);
	void channelRead(Channel<byte[]> channel, byte[] bytes);
}
```

```
acceptor.bind(PORT);

public class NioTcpAcceptor extends NioAcceptor {}
abstract public class NioAcceptor extends NioReactor implements IoAcceptor {}
public interface IoAcceptor extends IoReactor, IoAcceptorMBean {}


// NioAcceptor
synchronized public void bind(int port) throws IOException {
	// 具体分析见代码
	bind(new InetSocketAddress(port));
}
```









### craft.atom.rpc源码分析

#### update@20160810
```
ds = client.refer(DemoService.class);

refer(Class<T> rpcInterface) {
		return proxyFactory.getProxy(rpcInterface);
}


proxyFactory.setInvoker(invoker);
DefaultRpcProxyFactory {
	public <T> T getProxy(Class<T> rpcInterface) {
		return (T) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class<?>[] 			{ rpcInterface }, new RpcInvocationHandler(invoker));
	}
}

invoker = new DefaultRpcClientInvoker();
RpcMessage invoke(RpcMessage req) throws RpcException {
	return connector.send(req, async);
}

RpcMessage send(RpcMessage req, boolean async) throws RpcException {
	DefaultRpcChannel channel = select(mid);
	channel.write(req);
	future.await(req.getRpcTimeoutInMillis(), TimeUnit.MILLISECONDS);
	return future.getResponse();
}
```



```
client = RpcFactory.newRpcClient(host, port);
client.open();

----------------client
DefaultRpcClient() {
	connector = new DefaultRpcConnector();
}

----------------open
open() {
	for (int i = 0; i < connections; i++) {
		connector.connect();
	}
}

----------------connect
connect() {
	Future<Channel<byte[]>> future = ioConnector.connect(address);
	Channel<byte[]> channel = future.get(connectTimeoutInMillis, TimeUnit.MILLISECONDS);
	DefaultRpcChannel rpcChannel = new DefaultRpcChannel(channel, protocol.getRpcEncoder(), protocol.getRpcDecoder());
	rpcChannel.setFutures(new ConcurrentHashMap<Long, RpcFuture<?>>());
	channel.setAttribute(RpcIoHandler.RPC_CHANNEL, rpcChannel);
	long id = channel.getId();
	channels.put(id, rpcChannel);
	return id;
}
```





#### update@20160808
```
server = RpcFactory.newRpcServer(port);
export()
open()

----------------server
server = RpcFactory.newRpcServer(port) {
	newRpcServerBuilder(port).build();
}

build() {
	DefaultRpcServer rs = new DefaultRpcServer();
	// 相关组件的设置: Rpc协议；注册中心；执行器等
	rs.init();
}

DefaultRpcServer() {
	// 相关组件的初始化: Rpc协议；注册中心；执行器等
}

init() {
	// 组件更加细致的初始化
}

----------------export
// 定义一个api，将api注册到registry中
public void export(String rpcId, Class<?> rpcInterface, String rpcMethodName, Class<?>[] rpcMethodParameterTypes, Object rpcObject, RpcParameter rpcParameter) {
	// 定义一个api
	DefaultRpcApi api = new DefaultRpcApi(rpcId, rpcInterface, new RpcMethod(rpcMethodName, rpcMethodParameterTypes), rpcObject, rpcParameter);
	// registry存储了一个api的map<apiKey, api>
	registry.register(api);	
}

----------------open
open() {
	acceptor.bind();
}

----------------RpcAcceptor bind()
public void bind() {
	ioHandler  = new RpcServerIoHandler(protocol, processor);
	ioAcceptor = NioFactory.newTcpAcceptorBuilder(ioHandler)
			    	.channelSize(connections)
				   	.ioTimeoutInMillis(ioTimeoutInMillis)
				   	.dispatcher(new NioOrderedDirectChannelEventDispatcher())
				   	.build();
	ioAcceptor.bind(address);  // 监听事件
}

继续深入: 传入protocol，processor
protocol: encoder函数
processor: 
public void process(RpcMessage req, RpcChannel channel) {
	RpcApi api = api(req);
	// ???
	executor = executorFactory.getExecutor(api);
	// 
	executor.execute(new ProcessTask(req, channel));
}

ProcessTask {
	run() {
		invoker.invoke(req)
	}
}
```




#### update 

```
		host = "localhost";
		port = AvailablePortFinder.getNextAvailable();
		server = RpcFactory.newRpcServer(port);
		server.export(DemoService.class, new DemoServiceImpl1(), new RpcParameter(10, 100));
		server.open();
		
		client = RpcFactory.newRpcClient(host, port);
		client.open();
		ds = client.refer(DemoService.class);
```

通过追踪上述代码的执行流，来分析源码架构。

```
client = RpcFactory.newRpcClient(host, port);
新建一个client: 包含connector，invoker

client.open(): 很关键的一句 connector.connect()
connector.connect(): 和服务端建立连接，数据传输通道rpcChannel


ds = client.refer(DemoService.class): 关键一句proxyFactory.getProxy(rpcInterface);
动态代理的概念

DefaultRpcProxyFactory(proxyFactory)有一个关键成员invoker
DefaultRpcClientInvoker(invoker)有一个关键成员connector

DefaultRpcConnector(connector)数据通道

数据通道更底层是DefaultRpcChannel
```


TODO

* 动态代理: client.refer(DemoService.class)
* 类图


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



