首要目标：将 Java Http 客户端调用过程变得简单

### FeignClient
- 用于创建 声明式 API 接口
	- value、name： 被调用的服务的 ServiceId
	- url：服务 url 地址
	- configuration：指明 FeginClinet 的配置类


### FeignClient的配置
- FeignClientsConfiguration 是 Feign Client 默认的配置类
- 如果需要覆盖，则重写一个 @Bean 就行覆盖

### Feign 的工作原理
- Feign 是一个伪 Java 客户端
- Feign 不做任何的请求处理
- Feign 通过处理注解生成 Request 模板
- 在发送 Http Request 请求之前，通过注解的方式替换 Request 模板中的参数，生成真正的 Request，交给 Java Http 客户端去处理
- 使得开发者只需关注 Feign 注解模板的开发

#### FeignClientsRegistrar
- 程序启动时，会检查是否有 @EnableFeignClients
- 如果有，则开启包扫描 basePackages 下 带有 @FeignClinet 注解的类
- 如果不是接口，报错
- 解析注解信息，通过 BeanDefinitionBuilder 构建 BeanDefinition
- BeanDefinition 的 beanClass 是 FeignClinetFactoryBean
	- FactoryBean 在 getbean 的时候，会判断是否是 FacotryBean，如果是的话，就调用 FactoryBean 的 getObject() 方法 获取 bean 实例
- 通过 BeanDefinitionHolder 注册 BeanDefinition
- FeignClinetFactoryBean # getBean 时，会进行 JDK 动态代理，通过 SynchronousMethodHandler 的拦截，根据参数生成 RequestTemplate 对象
- SynchronousMethodHandler#executeAndDecode() 会使用 Http Client 获取 Response 

``` java
@Override
public Object getObject() throws Exception {  
   return getTarget();  
}

<T> T getTarget() {  
	FeignContext context = this.applicationContext.getBean(FeignContext.class);  
	Feign.Builder builder = feign(context);
	...
	// 返回代理对象
	return (T) targeter.target(this, builder, context,  
 		new HardCodedTarget<>(this.type, this.name, url));
}

public <T> T target(Target<T> target) {  
	return build().newInstance(target);  
}

private final SynchronousMethodHandler.Factory factory;
@Override  
public <T> T newInstance(Target<T> target) {
	···
	InvocationHandler handler = factory.create(target, methodToHandler);  
	T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),  
 new Class<?>[] {target.type()}, handler)
	···
}
final class SynchronousMethodHandler implements MethodHandler{
	@Override  
	public Object invoke(Object[] argv) throws Throwable {
		// 创建 Request 模板
		RequestTemplate template = buildTemplateFromArgs.create(argv);  
		Options options = findOptions(argv);
		···
		return executeAndDecode(template, options);
		···
	}
}

Object executeAndDecode(RequestTemplate template, Options options) throws Throwable {
	// 根据 Request 模板创建 Request
	Request request = targetRequest(template);	
	Response response;
	···
	// 通过 HttpClient 发送请求获取响应
	response = client.execute(request, options);
	···
	// 返回响应
	return response;
}

```

### Feign 中使用HttpClient 和 OKHttp
- Client 在 Feign 中是一个接口
- 默认实现是 Client.Default
- Client.Default 由 HttpURLConnection 实现网络请求

### Feign 如何实现负载均衡
- FeignRibbonClientAutoConfig  最终往容器注入的是 LoadBalancerFeignClient

```java
public class FeignRibbonClientAutoConfiguration {
	@Bean  
	@ConditionalOnMissingBean  
	public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory, 
 SpringClientFactory clientFactory) {  
		return new LoadBalancerFeignClient(new Client.Default(null, null),  cachingFactory, clientFactory);  
	}
}

public class LoadBalancerFeignClient implements Client {
	@Override  
	public Response execute(Request request, Request.Options options) throws IOException {
		URI asUri = URI.create(request.url());  
		String clientName = asUri.getHost();
		···
		// 执行负载均衡
		return lbClient(clientName).executeWithLoadBalancer(ribbonRequest,  
 requestConfig).toResponse();
		···
	}
}
public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
	···
	// 利用 Ribbon 的 LoadBalancerCommand 实现负载均衡
	LoadBalancerCommand<T> command = LoadBalancerCommand.<T>builder()  
	.withLoadBalancerContext(this)  
	.withRetryHandler(handler)  
	.withLoadBalancerURI(request.getUri())  
	.build();
	···
		return command.submit(
			new ServerOperation<T>() {
			})
	···
}
```

### 使用 Ribbon
- Ioc 容器中注入一个 RestTemplate
- 
- 在 restTemplate 上加上 @LoadBalanced


### LoadBalancerClient 简介
- 负责均衡器的核心类为 LoadBalancerClient，LoadBalancerClinet 可以获取负载均衡服务提供者的实例信息
- LoadBalancerClient 通过注册中心 Client 获取服务注册列表信息，并缓存起来
- LoadBalancerClient 调用 choose() 方法，根据负载均衡策略选在一个服务实例的信息
- LoadBalancerClient 也可以不从 Eureka Client 获取注册列表信息

#### 源码解析
- 继承结构：
	- ServiceInstanceChooser -> LoadBalancerClient -> RibbonLoadBalanceClient

- LoadBalancerClient
	- 三个方法
		- T execute(String serviceId, LoadBalancerRequest\<T\> request)
		- T execute(String serviceId, ServiceInstance serviceInstance,  LoadBalancerRequest\<T\> request)
		- URI reconstructURI(ServiceInstance instance, URI original)
	
- RibbonLoadBalancerClient
	- choose() 用于选择具体的实例

	```java
	public class RibbonLoadBalancerClient implements LoadBalancerClient {
		public ServiceInstance choose(String serviceId, Object hint) {  
			Server server = getServer(getLoadBalancer(serviceId), hint);  
			if (server == null) {  
				return null;  
			}  
			return new RibbonServer(serviceId, server, isSecure(server, serviceId),  
			serverIntrospector(serviceId).getMetadata(server));  
		}
	}
	```

	- getServer() 方法去获取实例，最终交给 ILoadBalancer 选择实例

	``` java
	protected Server getServer(ILoadBalancer loadBalancer, Object hint) {  
		if (loadBalancer == null) {  
			return null;  
		}  
		// Use 'default' on a null hint, or just pass it on?  
		return loadBalancer.chooseServer(hint != null ? hint : "default");  
	}
	```
	
- ILoadBalancer 定义了一系列实现负载均衡的方法
	- addServers() 添加一个 Servers 集合
	- chooseServer() 根据 key 去获取 Server
	- markServerDown() 标记某个服务下线
	- getReachableServers() 获取可用的 Server 集合
	- getAllServers() 获取所有的 Server 集合
- ILoadBalancer 子类为 BaseLoadBalancer，BaseLoadBalancer实现类为 DynamicServerListLoadBalancer