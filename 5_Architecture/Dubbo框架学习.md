# Dubbo


## update @ 20160705

servlet啥容器？
通过http协议来理解, 结合服务启动bug日志



## update @ 20160702

实践熟悉下面2个概念。庖丁解牛。

* Spring: dubbo就是委托给spring来管理bean的生命周期的。afterPropertiesSet是Spring中的一个回调函数
* SPI(Service Provider Interface 扩展点机制): 专门的开源项目Cooma；com.alibaba.dubbo.common.extension包


## update @ 20160611
通过一个简单的RPC例子，对Exporter和Invoker的概念有定理解。

梳理概念

* Invoker: 
* Exporter: 暴露服务即绑定ServerSocket到配置端口(网络通信层面)，这样就可以接受外面的请求了。所以这个叫做暴露服务。
* Protocol

* 在RPC中，Protocol是核心层，也就是只要有Protocol + Invoker + Exporter就可以完成非透明的RPC调用，然后在Invoker的主过程上Filter拦截点。
* Remoting模块: 通讯层。实现是Dubbo协议的实现。Remoting内部再划为Transport传输层和Exchange信息交换层，Transport层只负责单向消息传输，是对Mina,Netty,Grizzly的抽象，它也可以扩展UDP传输，而Exchange层是在传输层之上封装了Request-Response语义。


梳理过程

* 服务启动过程
* 消费者消费过程


服务启动过程

```
    private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        // 导出服务
        String contextPath = protocolConfig.getContextpath();
        URL url = new URL(name, host, port, (contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);
		 // registryURLs 是注册url列表
		for (URL registryURL : registryURLs) {
       		url = url.addParameterIfAbsent("dynamic", registryURL.getParameter("dynamic"));
            URL monitorUrl = loadMonitor(registryURL);
            //获取invoker  
            Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
            //根据协议将invoker暴露成exporter，具体过程是创建一个ExchangeServer，它会绑定一个ServerSocket到配置端口  
            Exporter<?> exporter = protocol.export(invoker);
            //将创建的exporter放进链表便于管理  
            exporters.add(exporter);
    	}
	}
```



## update @ 20160605
之前看dubbo源码不是很理解export的细节过程，直到看到 `RPC框架几行代码就够了`这篇博客的介绍，才有所理解。

博客的评论也基本和个人认识一致，主要就是几个核心概念

```
1. Socket编程，传输Object
2. 动态代理
```

### 通信协议
客户端和服务端通信需要约定`通信协议`(这个协议很关键), 这个例子中的约定即是: 方法名，参数类型，方法所需参数

### 动态代理
`动态代理`的概念也需要理解。

### 网络通信
Netty框架 NIO



### Ref

```
RPC框架几行代码就够了
http://javatar.iteye.com/blog/1123915

动态代理
http://www.codekk.com/blogs/detail/54cfab086c4761e5001b253d
http://182.254.149.236/?p=71
http://zmx.iteye.com/blog/678416


Dubbo源码分析
http://blog.csdn.net/flashflight/article/details/43939275
http://gaofeihang.cn/archives/155
```


## before 20160505
Main启动: 容器模块com.alibaba.dubbo.container.Main 

问题：

* 业务服务DemoServiceImpl到AbstractProxyInvoker实例的封装: Javassist技术
	
	```
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
	```

* Invoker到Exporter的封装
	
	```
	Exporter<?> exporter = protocol.export(invoker);
   ```



当网络通讯层收到一个请求后，会找到对应的Exporter实例，并调用它所对应的AbstractProxyInvoker实例，从而真正调用了服务提供者的代码。


////////////通过阅读源码了解:

* 服务配置的读取
* 服务启动
* 一次请求的整个流程

* Spring的动态代理技术，彻底摆脱‘new’
* 设计模式: 工厂模式 和 代理模式


proxy代理层按照DUBBO官方文档的解释，是用来生成RPC调用的Stub和Skeleton，这样做的目的是让您在DUBBO服务端定义的具体业务实现不需要关心“它将被怎样调用”，也是您定义的服务接口“与RPC框架脱耦”。下图是DUBBO框架proxy层的主要类图结构：


关键几个概念:

* Invoker – 执行具体的远程调用: 这里的Invoker是Provider的一个可调用Service的抽象，Invoker封装了Provider地址及Service接口信息。

* Protocol – 服务地址的发布和订阅

```
远程调用层（Protocol）：封将RPC调用，以Invocation和Result为中心，扩展接口为 
Protocol、Invoker和Exporter。Protocol是服务域，它是Invoker暴露和引用的主功能
口，它负责Invoker的生命周期管理。Invoker是实体域，它是Dubbo的核心模型，其它模型都
它靠扰，或转换成它，它代表一个可执行体，可向它发起invoke调用，它有可能是一个本地的
现，也可能是一个远程的实现，也可能一个集群实现。

```

* Exporter – 暴露服务的引用，或取消暴露


Proxy层封装了所有接口的透明化代理，而在其它层都以Invoker为中心，只有到了暴露给用户使用时，才用Proxy将Invoker转成接口，或将接口实现转成Invoker，也就是去掉Proxy层RPC是可以Run的，只是不那么透明，不那么看起来像调本地服务一样调远程服务。



## Dubbo流程
### 1.服务启动：ServiceBean

```
	public void init() {
	    registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new DubboBeanDefinitionParser(AnnotationBean.class, true));
    }
```
重点关注 

```
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));

```

ProviderConfig、ProtocolConfig
服务的一些信息: host，port，threadpool。。。


ServiceBean extends ServiceConfig:

```
    // 接口类型
    private String              interfaceName;
    private Class<?>            interfaceClass;
    // 接口实现类引用
    private T                   ref;
    // 服务名称
    private String              path;
    private ProviderConfig provider;
```

调用关系: afterPropertiesSet(读取了各种配置之后就export)
export -> doExport -> doExportUrls -> doExportUrlsFor1Protocol(各种协议)

```
    private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    。。。
        // 导出服务
        String contextPath = protocolConfig.getContextpath();
        URL url = new URL(name, host, port, (contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);

        //如果配置不是local则暴露为远程服务.(配置为local，则表示只暴露远程服务)
		 // registryURLs 是注册url列表
		            for (URL registryURL : registryURLs) {
                        url = url.addParameterIfAbsent("dynamic", registryURL.getParameter("dynamic"));
                        URL monitorUrl = loadMonitor(registryURL);
                        if (monitorUrl != null) {
                            url = url.addParameterAndEncoded(Constants.MONITOR_KEY, monitorUrl.toFullString());
                        }
                        if (logger.isInfoEnabled()) {
                            logger.info("Register dubbo service " + interfaceClass.getName() + " url " + url + " to registry " + registryURL);
                        }
                        Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));

                        Exporter<?> exporter = protocol.export(invoker);
                        exporters.add(exporter);
                    }
			
	
	}
```

下面重点分析： 获取invoker，倒出exporter

```
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
Exporter<?> exporter = protocol.export(invoker);
exporters.add(exporter);
```


问题: 追踪 proxyFactory.getInvoker 的实际调用以及调用效果

```
// 重要语句
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
// 参数含义
ref: 接口实现类引用
interfaceClass: 接口类
registryURL: 注册中心导出URL参数信息；url中包含host，port信息



// 代理接口ProxyFactory定义
public interface ProxyFactory {
    <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException;
}

// 抽象代理类 AbstractProxyFactory
public abstract class AbstractProxyFactory implements ProxyFactory {
}

// 抽象代理类的一个实现: JavassistProxyFactory
// javassist这个组件的功能：在运行时动态加载class，并进行实例化
public class JavassistProxyFactory extends AbstractProxyFactory {
	 ///////////////////// getInvoker 真面目
    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) {
        final Wrapper wrapper = Wrapper.getWrapper(proxy.getClass().getName().indexOf('$') < 0 ? proxy.getClass() : type);        
        // 返回 Invoker抽象代理, 复写doInvoke方法
        return new AbstractProxyInvoker<T>(proxy, type, url) {
            @Override
            protected Object doInvoke(T proxy, String methodName, 
                                      Class<?>[] parameterTypes, 
                                      Object[] arguments) throws Throwable {
                return wrapper.invokeMethod(proxy, methodName, parameterTypes, arguments);
            }
        };
    }
}

public abstract class Wrapper {
	// 问题: 代码生成代码
	private static Wrapper makeWrapper(Class<?> c) {
	}
}




// Invoker抽象代理: AbstractProxyInvoker
public abstract class AbstractProxyInvoker<T> implements Invoker<T> {
	 // invoke方法: 实际调用doInvoke
    public Result invoke(Invocation invocation) throws RpcException {
    	return new RpcResult(doInvoke(proxy, invocation.getMethodName(), invocation.getParameterTypes(), invocation.getArguments()));
   	 }
   	 // 实际调用
   	 protected abstract Object doInvoke(T proxy, String methodName, Class<?>[] parameterTypes, Object[] arguments) throws Throwable;

}

// Invoke
public interface Invoker<T> extends Node {
	// service interface
    Class<T> getInterface();
	// invoke抽象
    Result invoke(Invocation invocation) throws RpcException;
}

// Invoke抽象类: AbstractInvoker
public abstract class AbstractInvoker<T> implements Invoker<T> {
    private final Class<T>   type;
    private final URL        url;

    public Result invoke(Invocation inv) throws RpcException {
        return doInvoke(invocation);
    }
    // 实际调用doInvoke
    protected abstract Result doInvoke(Invocation invocation) throws Throwable;
}

// Dubbo Invoker
public class DubboInvoker<T> extends AbstractInvoker<T> {
	 // 问题: 合适初始化的clients
    private final ExchangeClient[]      clients;
    private final Set<Invoker<?>> invokers;
    
    protected Result doInvoke(final Invocation invocation) throws Throwable {
        RpcInvocation inv = (RpcInvocation) invocation;
        final String methodName = RpcUtils.getMethodName(invocation);
        inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
        inv.setAttachment(Constants.VERSION_KEY, version);
        
        ExchangeClient currentClient;     
        currentClient = clients[index.getAndIncrement() % clients.length];
        
        RpcContext.getContext().setFuture(null);
        // request
        return (Result) currentClient.request(inv, timeout).get();
	}
}
```



protocol协议

```
Exporter<?> exporter = protocol.export(invoker);
exporters.add(exporter);
```


```
public interface Protocol {
	// 暴露远程服务
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;
	//引用远程服务
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

}

public abstract class AbstractProtocol implements Protocol {
	protected final Map<String, Exporter<?>> exporterMap = new ConcurrentHashMap<String, Exporter<?>>();
    protected final Set<Invoker<?>> invokers = new ConcurrentHashSet<Invoker<?>>();
}

public class DubboProtocol extends AbstractProtocol {
    private final Map<String, ExchangeServer> serverMap = new ConcurrentHashMap<String, ExchangeServer>(); // <host:port,Exchanger>
    private final Map<String, ReferenceCountExchangeClient> referenceClientMap = new ConcurrentHashMap<String, ReferenceCountExchangeClient>(); // <host:port,Exchanger>
    private final ConcurrentMap<String, LazyConnectExchangeClient> ghostClientMap = new ConcurrentHashMap<String, LazyConnectExchangeClient>();

   public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        URL url = invoker.getUrl();
        
        // export service.
        String key = serviceKey(url);
        DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
        exporterMap.put(key, exporter);
        
        //export an stub service for dispaching event
        Boolean isStubSupportEvent = url.getParameter(Constants.STUB_EVENT_KEY,Constants.DEFAULT_STUB_EVENT);
        Boolean isCallbackservice = url.getParameter(Constants.IS_CALLBACK_SERVICE, false);
        if (isStubSupportEvent && !isCallbackservice){
            String stubServiceMethods = url.getParameter(Constants.STUB_EVENT_METHODS_KEY);
            if (stubServiceMethods == null || stubServiceMethods.length() == 0 ){
                if (logger.isWarnEnabled()){
                    logger.warn(new IllegalStateException("consumer [" +url.getParameter(Constants.INTERFACE_KEY) +
                            "], has set stubproxy support event ,but no stub methods founded."));
                }
            } else {
                stubServiceMethodsMap.put(url.getServiceKey(), stubServiceMethods);
            }
        }

        openServer(url);
        
        return exporter;
    }
}


    private void openServer(URL url) {
        // find server.
        String key = url.getAddress();
        //client 也可以暴露一个只有server可以调用的服务。
        boolean isServer = url.getParameter(Constants.IS_SERVER_KEY,true);
        if (isServer) {
        	ExchangeServer server = serverMap.get(key);
        	if (server == null) {
        		serverMap.put(key, createServer(url));
        	} else {
        		//server支持reset,配合override功能使用
        		server.reset(url);
        	}
        }
    }

    private ExchangeServer createServer(URL url) {
        ExchangeServer server;
        server = Exchangers.bind(url, requestHandler);
        return server;
    }
    
    
    public static ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
        return getExchanger(url).bind(url, handler);
    }


public class HeaderExchanger implements Exchanger {
    public static final String NAME = "header";
    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }
    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
    }

}

public class Transporters {
    public static Server bind(URL url, ChannelHandler... handlers) throws RemotingException {
        ChannelHandler handler;
        handler = new ChannelHandlerDispatcher(handlers);
        return getTransporter().bind(url, handler);
    }
    
    public static Transporter getTransporter() {
       return ExtensionLoader.getExtensionLoader(Transporter.class).getAdaptiveExtension();
    }
}
 
 
 public class NettyTransporter implements Transporter {    
    public Server bind(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyServer(url, listener);
    }
    public Client connect(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyClient(url, listener);
    }
}
 
 
 public class NettyServer extends AbstractServer implements Server {
    protected void doOpen() throws Throwable {
        NettyHelper.setNettyLoggerFactory();
        ExecutorService boss = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerBoss", true));
        ExecutorService worker = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerWorker", true));
        ChannelFactory channelFactory = new NioServerSocketChannelFactory(boss, worker, getUrl().getPositiveParameter(Constants.IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS));
        bootstrap = new ServerBootstrap(channelFactory);
        
        final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
        channels = nettyHandler.getChannels();
        // https://issues.jboss.org/browse/NETTY-365
        // https://issues.jboss.org/browse/NETTY-379
        // final Timer timer = new HashedWheelTimer(new NamedThreadFactory("NettyIdleTimer", true));
        bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
            public ChannelPipeline getPipeline() {
                NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec() ,getUrl(), NettyServer.this);
                ChannelPipeline pipeline = Channels.pipeline();
                /*int idleTimeout = getIdleTimeout();
                if (idleTimeout > 10000) {
                    pipeline.addLast("timer", new IdleStateHandler(timer, idleTimeout / 1000, 0, 0));
                }*/
                pipeline.addLast("decoder", adapter.getDecoder());
                pipeline.addLast("encoder", adapter.getEncoder());
                pipeline.addLast("handler", nettyHandler);
                return pipeline;
            }
        });
        // bind
        channel = bootstrap.bind(getBindAddress());
    }
}
```



```
Exporter

public class DubboExporter<T> extends AbstractExporter<T> {
    private final Map<String, Exporter<?>> exporterMap;

}

public abstract class AbstractExporter<T> implements Exporter<T> {
    private final Invoker<T> invoker;
}

public interface Exporter<T> {
	Invoker<T> getInvoker();
}
```


### 2.服务消费：ReferenceBean
get() -> init() -> invoker = refprotocol.refer(interfaceClass, urls.get(0)) -> proxyFactory.getProxy(invoker)


```
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean { 
  	public void afterPropertiesSet() throws Exception {
		// 读取完各种配置文件后
       getObject();
       // 实际调用 get()
	}
}
```

```
public class ReferenceConfig<T> extends AbstractReferenceConfig {
	private static final Protocol refprotocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
    private static final ProxyFactory proxyFactory = ExtensionLoader.getExtensionLoader(ProxyFactory.class).getAdaptiveExtension();


    // 接口类型
    private String               interfaceName;
    private Class<?>             interfaceClass;
    // 客户端类型
    private String               client;
    // 点对点直连服务提供地址
    private String               url;
    // 方法配置
    private List<MethodConfig>   methods;
    // 缺省配置
    private ConsumerConfig       consumer;
    private String				 protocol;
    // 接口代理类引用
    private transient volatile T ref;
    private transient volatile Invoker<?> invoker;
    
    public synchronized T get() {
    	if (ref == null) {
    		init();
    	}
    	return ref;
    }
    
    
    private void init() {
    	// 一些配置信息
       Map<String, String> map = new HashMap<String, String>();
       ref = createProxy(map);
	 }  
	 
	 private T createProxy(Map<String, String> map) {
	 	// 远程服务(用户指定url／注册中心配置拼装url)
       invoker = refprotocol.refer(interfaceClass, urls.get(0));
       
	 	// 创建服务代理
    	return (T) proxyFactory.getProxy(invoker);
	 }
}
```

```
DubboProtocol

    public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
        // create rpc invoker.
        DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
        invokers.add(invoker);
        return invoker;
    }
```

```
public class JavassistProxyFactory extends AbstractProxyFactory {
    public <T> T getProxy(Invoker<T> invoker, Class<?>[] interfaces) {
        return (T) Proxy.getProxy(interfaces).newInstance(new InvokerInvocationHandler(invoker));
    }
}
```







## Dubbo各个模块  

* dubbo-remoting: 远程通讯模块，底层通讯

```
dubbo-remoting：dubbo是一个RPC框架，肯定需要进行远程通信，这里指的远程通信包含
dubbo涉及到的所有远程通信，比如zookeeper的通信，服务调用的通信等，所以这里远程通信不
只是服务调用的远程通信。dubbo通过该模块对它的远程通信进行了抽象，定义了通信层面的相关
接口，并且基于这些接口在该模块中提供了各种通信方式的实现。比如netty,mina,redis以及
zookeeper的通信实现。所以这个模块是要解决通信领域的问题。
```

* dubbo-rpc: 远程调用模块，抽象动态代理以及各种协议

```
该模块很容易和dubbo-remoting混淆，但是这两个模块是为了解决两个完全不同领域的问题。上 
面说了dubbo-remoting是解决通信传输层的问题，它面对的是整个通信底层，站在它的角度，他 
不需要关注通信是使用什么协议，它只是知道需要将请求消息包装成统一的Request发送给远程服 
务，并且只需要将远程服务响应消息封装成Response下发到上层模块。而dubbo-rpc属于协议层 
面的抽象，它是为了解决dubbo协议层面的解耦问题，该模块需要依赖dubbo-remoting来完成和
远程服务的通信，它是基于dubbo-remoting实现各种协议。比如dubbo协议可以基于remoting
的mina或者netty的实现，那么此时mina或者netty传输的消息格式是复合dubbo协议的格式，而
dubbo协议消息格式组装则是在dubbo-rpc层来完成。这里就体现了dubbo在整个传输层面，对协
议和通信进行了分离，从而可以实现一种协议可以通过多种通信方式和远程交互。这也增加了
dubbo框架的灵活性，也体现了它的低耦合。

```


* dubbo-config: 配置模块，Dubbo对外的API，用户通过Config使用Dubbo

```
dubbo-config：有关dubbo框架的加载各个途径配置都在该模块实现，比如通过properties文
件进行配置，通过spring的xml对服务进行配置。并且对这些配置信息的解析和组装各个模块的配
置也是在该模块实现，比如对配置中心，服务引用，服务发布等。
```

* dubbo-common

```
这个模块就是dubbo的内核，该模块将dubbo的其他模块组合成一个整体，所有的模块都是围绕该 模块进行扩展开，这里也是Dubbo微内核的体现。这个模块主要定义了dubbo整体框架的插件形  
式，以及模块存在共享的逻辑实现，它是各个模块之间的纽带，各个模块之间的通信，均是需要通
过该模块将它们关联起来，有它的存在dubbo各个模块才是一个整体。从而使得Dubbo框架达到高
内聚的目的。
```

ref:
http://bbs.dubboclub.net/read-31.html
http://gaofeihang.cn/archives/155
http://shiyanjun.cn/archives/325.html




反射  工厂模式

## 代理
```
public interface BusinessInterface {
    public void dosomething(String username);
}
```


```
public class RealBusinessImpl implements BusinessInterface {
    @Override
    public void dosomething(String username) { 
    }
}
```

```
public class ProxyBusinessImpl implements BusinessInterface {
    private RealBusinessImpl realBusiness;

    public ProxyBusinessImpl(RealBusinessImpl realBusiness) {
        this.realBusiness = realBusiness;
    }

    @Override
    public void dosomething(String username) {
        System.out.println("---------正式业务执行前；");
        this.realBusiness.dosomething(username);
        System.out.println("---------正式业务执行后；");
    }
}
```

## 动态代理

```
public class BusinessInvocationHandler implements InvocationHandler {
    private BusinessInterface realBusiness;

    public BusinessInvocationHandler(BusinessInterface realBusiness) {
        this.realBusiness = realBusiness;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object resultObject = method.invoke(this.realBusiness, args);
        return resultObject;
    }

}
```


外部模块/外部系统如何进行调用：

```
public class Main {
    public static void main(String[] args) throws Exception {
        BusinessInterface realBusiness = new RealBusinessImpl();
        BusinessInvocationHandler invocationHandler = new BusinessInvocationHandler(realBusiness);

        /*
         * 生成一个动态代理实例。里面的三个参数需要讲解一下：
         * 1-loader：这个newProxyInstance会有一个返回值，即代理对象。
         * 那么问题就是类实例的创建必须要有classloader的支持，第一个参数就是指等“代理对象”的创建所依据的classloader
         * 
         * 2-interfaces：第二个参数是一个数组。在设计原理中，有一个重要的原则是“依赖倒置”，它的实践经验是：“依赖接口，而不是以来实现”。
         * 所以，JAVA中动态代理的支持假定程序员是遵循这一原则的：所有业务都定义的接口。这个参数就是为动态代理指定“代理对象所实现的接口”，
         * 由于JAVA中一个类可以实现多个接口，所以这个参数是一个数组（我的实例代码中，只为真实的业务实现定义了一个接口BusinessInterface，
         * 所以参数中指定的也就只有这个接口）.另外，这个参数的类型是Class，所以如果您不定义接口，而是指定某个具体类，也是可行的。但是这不符合设计原则。
         * 
         * 3-InvocationHandler：这个就是我们的“调用处理器”，这个参数没有太多解释的
         * */
        BusinessInterface proxyBusiness = (BusinessInterface)Proxy.newProxyInstance(
                Thread.currentThread().getContextClassLoader(), 
                new Class[]{BusinessInterface.class}, 
                invocationHandler);

        // 正式调用
        proxyBusiness.dosomething("yinwenjie");
    }
}
```



# BIO Vs NIO
http://tutorials.jenkov.com/java-nio/overview.html


```
Netty is a NIO client server framework which enables quick and easy development of network 
applications such as protocol servers and clients. It greatly simplifies and streamlines network 
programming such as TCP and UDP socket server.
```

# Hibernate框架
主要是实现数据库与实体类间的映射，使的操作实体类相当与操作hibernate框架。

# Spring 框架