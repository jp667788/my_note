### 依赖查找的来源
- 查找来源

| 来源 | 配置元数据 |
| - | - |
| Spring BeanDefinition | \<bean id="user" class="org.geekbang...User" \> |
|  | @Bean public User user(){...} |
|  | BeanDefinitionBuilder |
| 单例对象 | API 实现 |

- Spring 內建 BeanDefintion
	- AnnotationConfigUtils#registerAnnotationConfigProcessors() 注册内建 Bean
		- 解析 XML bean （XML 驱动）
		- ComponentScan 注解驱动

| Bean 名称 | Bean 实例 | 使用场景 |
|-|-|-|
| org.springframework.context. annotation.internalConfigurationAnnotationProcessor | ConfigurationClassPostProcesso r 对象 | 处理 Spring 配置类 |
|org.springframework.context. annotation.internalAutowire dAnnotationProcessor|AutowiredAnnotationBeanPostPro cessor 对象|处理 @Autowired 以及 @Value 注解|
|org.springframework.context. annotation.internalCommonAn notationProcessor|CommonAnnotationBeanPostProces sor 对象|（条件激活）处理 JSR-250 注解， 如 @PostConstruct 等|
|org.springframework.context. event.internalEventListener Processor|EventListenerMethodProcessor 对象|处理标注 @EventListener 的 Spring 事件监听方法|


- Spring 內建单例对象
	- AbstractApplicationContext#prepareBeanFactory() 中会调用 beanFactory.registerSingleton() 注册内建单例对象

|Bean 名称|Bean 实例|使用场景|
|-|-|-|
|environment|Environment 对象|外部化配置以及 Profiles|
|systemProperties|java.util.Properties 对象|Java 系统属性|
|systemEnvironment|java.util.Map 对象|操作系统环境变量|
|messageSource|MessageSource 对象|国际化文案|
|lifecycleProcessor|LifecycleProcessor 对象|Lifecycle Bean 处理器|
|applicationEventMulticaster|ApplicationEventMulticaster 对 象|Spring 事件广播器|

### 依赖注入的来源

- 注入来源

|来源|配置元数据|
|-|-|
| Spring BeanDefinition | \<bean id="user" class="org.geekbang...User" \> |
|  | @Bean public User user(){...} |
|  | BeanDefinitionBuilder |
| 单例对象 | API 实现 |
| 非 Spring 容器管理对象 |  |

- 依赖对象

| 来源 | Spring Bean 对象 | 生命周期管理 | 配置元信息 | 使用场景 |
|-|-|-|-|-|
|Spring BeanDefinition|是|是|有|依赖查找、依赖注入|
|单体对象|是|否|无|依赖查找、依赖注入|
|Resolvable Dependencyn|否|否|无|依赖查找、依赖注入|

#### ResolvableDependency 
- ResolvableDependency 主要是 Spring 内部对象，而不需要重复创建。
- AbstractApplicationContext#prepareRefresh()  中注册了 4 个resolveDependency：
	- BeanFactory
	- ResourceLoader
	- ApplicationEventPublisher
	- ApplicationContext
- 存放在 **resolvableDependencies** ConcurrentHashMap 中 ，而并不是容器中
- ResolvableDependency 可以依赖注入如 @Autowired ，但是不能进行依赖查找 getBean()
	- 因为在依赖注入的时候 DefaultListableBeanFactory#findAutowireCandidates() 中，自动注入了 resolvableDependencies 中的对象：
	``` JAVA
		protected Map<String, Object> findAutowireCandidates(  
		  @Nullable String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {  

			String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(  
				 this, requiredType, true, descriptor.isEager());  
			Map<String, Object> result = new LinkedHashMap<>(candidateNames.length);  
			// 自动注入了 resolvableDependencies 中的对象：
			for (Map.Entry<Class<?>, Object> classObjectEntry : this.resolvableDependencies.entrySet()) {  
			  Class<?> autowiringType = classObjectEntry.getKey();  
			if (autowiringType.isAssignableFrom(requiredType)) {  
				 Object autowiringValue = classObjectEntry.getValue();  
		 autowiringValue = AutowireUtils.resolveAutowiringValue(autowiringValue, requiredType);  
			if (requiredType.isInstance(autowiringValue)) {  
					result.put(ObjectUtils.identityToString(autowiringValue), autowiringValue);  
		 break; }  
			  }  
		   }
			....
		}
	```

AbstractApplicationContext.java:663：

``` java
	// BeanFactory interface not registered as resolvable type in a plain factory.  
	// MessageSource registered (and found for autowiring) as a bean.  
	beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);  
	beanFactory.registerResolvableDependency(ResourceLoader.class, this);  
	beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);  
	beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```

DefaultListableBeanFactory#registerResolvableDependency：

``` java
	@Override  
	public void registerResolvableDependency(Class<?> dependencyType, @Nullable Object autowiredValue) {  
		   Assert.notNull(dependencyType, "Dependency type must not be null");  
		 if (autowiredValue != null) {  
			  if (!(autowiredValue instanceof ObjectFactory || dependencyType.isInstance(autowiredValue))) {  
				 throw new IllegalArgumentException("Value [" + autowiredValue +  
					   "] does not implement specified dependency type [" + dependencyType.getName() + "]");  
		 }  
			  this.resolvableDependencies.put(dependencyType, autowiredValue);  
		 }  
	}
```


### Spring BeanDefinition 作为依赖来源

- 元数据：BeanDefinition

- 注册：BeanDefinitionRegistry#registerBeanDefinition

- 类型：延迟和非延迟

- 顺序：Bean 生命周期顺序按照注册顺序
	- DefaultListableBeanFactory#beanDefinitionMap (ConcurrentHashMap) 存放 beanName - BeanDefinition
	- DefaultListableBeanFactory#beanDefinitionNames (ArrayList) 存放 BeanNames
	- ArrayList 可以记录注册顺序

### 单例对象作为依赖来源
- 来源：外部普通 Java 对象（不一定是 POJO）

- 注册：SingletonBeanRegistry#registerSingleton

- 限制
	- 无生命周期管理
		- 因为不是 BeanDefinition
	- 无法实现延迟初始化 Bean
		- 存储的是Object，不存在延迟初始化，实例都有了。延迟初始化是 BeanDefinition


### 非 Spring 容器管理对象作为依赖来源
- 注册：**ConfigurableListableBeanFactory#registerResolvableDependency**
- 限制
	- 无生命周期管理
	- 无法实现延迟初始化 Bean
	- 无法通过依赖查找

### 外部化配置作为依赖来源

- 类型：非常规 Spring 对象依赖来源
- 限制
	- 无生命周期管理
	- 无法实现延迟初始化 Bean
	- 无法通过依赖查找
- 实现: AutowiredAnnotationBeanPostProcessor


### 注入和查找的依赖来源是否相同？
否，依赖查找的来源仅限于 Spring BeanDefinition 以及单例 对象，而依赖注入的来源还包括 Resolvable Dependency 以及 @Value 所标注的外部化配置

### 单例对象能在 IoC 容器启动后注册吗？
可以的，单例对象的注册与 BeanDefinition 不同，BeanDefinition 会被 ConfigurableListableBeanFactory#**freezeConfiguration**() 方法影 响，从而冻结注册，单例对象则没有这个限制。

### Spring 依赖注入的来源有哪些？
- Spring BeanDefinition 
- 单例对象 
- Resolvable Dependency
- @Value 外部化配置