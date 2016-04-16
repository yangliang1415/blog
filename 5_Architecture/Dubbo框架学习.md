# Dubbo

通过阅读源码了解:

* 服务配置的读取
* 服务启动
* 一次请求的整个流程

关键几个概念:

* Invoker – 执行具体的远程调用
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

Main启动: 容器模块com.alibaba.dubbo.container.Main 

### 1. DubboNamespaceHandler extends NamespaceHandlerSupport(spring 框架): 解析加载配置

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

下面重点分析： 获取invoker，到处exporter

```
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
Exporter<?> exporter = protocol.export(invoker);
exporters.add(exporter);
```
proxyFactory.getInvoker实际调用对应协议例如DubboProtocol中的getInvoker方法

问题:

```
Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
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
