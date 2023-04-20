### 依赖注入的模式和类型
- 手动模式 - 配置或者编程的方式，提前安排注入规则
	- XML 资源配置元信息
	- Java 注解配置元信息
	- API 配置元信息
- 自动模式 - 实现方提供依赖自动关联的方式，按照內建的注入规则
	- Autowiring（自动绑定）


### 依赖注入的模式和类型
| 依赖注入类型 | 配置元数据举例 |
|-|-|
| Setter 方法 | \<proeprty name=”user” ref=”userBean” /\> |
| 构造器 | \<constructor-arg name="user" ref="userBean" /\> |
| 字段 | @Autowired User user; |
| 方法 | @Autowired public void user(User user) { ... } |
| 接口回调 | class MyBean implements BeanFactoryAware { ... } |

### 自动绑定

#### 官方定义
The Spring container can autowire relationships between collaborating beans. You can let Spring resolve collaborators (other beans) automatically for your bean by inspecting the contents of the ApplicationContext.

#### 优点
- Autowiring can significantly reduce the need to specify properties or constructor arguments. 可以减少配置语法
- Autowiring can update a configuration as your objects evolve. 需要自动装配的对象增加字段时，有可以自动装配的bean时，也会随之自动装配

#### 缺点
- 精确依赖：显示的 property 或者 construct-arg 会覆盖 autowriting
- 不能绑定简单类型 ：String Integer 等简单类型
- 缺少精确性：Autowiring 具有猜测性
- 多个Bean定义存在时，会产生歧义，抛出异常


- 使用显示注入
- autowire-candidate ：设置当前 Bean 是否为 其他 Bean autowiring 候选者
- 指定 Primary
- 

### 自动绑定（Autowiring）模式

| 模式 | 说明 |
|-|-|
| no | 默认值，未激活 Autowiring，需要手动指定依赖注入对象 |
| byName | 根据被注入属性的名称作为 Bean 名称进行依赖查找，并将对象设置到该属性 |
| byType | 根据被注入属性的类型作为依赖类型进行查找，并将对象设置到该属性 |
| constructor | 特殊 byType 类型，用于构造器参数 |

> 参考枚举：org.springframework.beans.factory.annotation.Autowire

### Setter 方法注入

- 手动模式
	- XML 资源配置元信息
	
		``` XML	
		<bean class="org.geekbang.thinking.in.spring.ioc.dependency.injection.UserHolder">  
		 ``<property name="user" ref="superUser" />  
		</bean>
		```

	- Java 注解配置元信息

		```JAVA
			@Bean  
			public UserHolder userHolder(User user) {  
				UserHolder userHolder = new UserHolder();  
			 userHolder.setUser(user);  // Setter 方法注入
			 return userHolder;  
			}

			public class User {
				private String name;
				
				public User(){
				}
				
			}
		```

	- API 配置元信息

		``` JAVA
			private static BeanDefinition createUserHolderBeanDefinition() {  
				BeanDefinitionBuilder definitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(UserHolder.class);  
				// API setter 方法注入 
			 definitionBuilder.addPropertyReference("user", "superUser");  
			 return definitionBuilder.getBeanDefinition();  
			}


			// 创建 BeanFactory 容器  
			AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();  

			// 生成 UserHolder 的 BeanDefinition  
			BeanDefinition userHolderBeanDefinition = createUserHolderBeanDefinition();  
			// 注册 UserHolder 的 BeanDefinition  
			applicationContext.registerBeanDefinition("userHolder", userHolderBeanDefinition);
		```

- 自动模式
	- byName
	
		``` xml
		<bean class="org.geekbang.thinking.in.spring.ioc.dependency.injection.UserHolder"  
		 autowire="byName"  
		 >  
		<!--        <property name="user" ref="superUser" /> 替换成 autowiring 模式 -->  
		 </bean>
		```
	
	- byType
		``` xml
		<bean class="org.geekbang.thinking.in.spring.ioc.dependency.injection.UserHolder"  
		 autowire="byType"  
		 >  
		<!--        <property name="user" ref="superUser" /> 替换成 autowiring 模式 -->  
		 </bean>
		```


### 构造器注入
**Constructor 能够确保线程安全。**

- 手动模式
	-  XML 资源配置元信息
		``` xml
		<bean class="org.geekbang.thinking.in.spring.ioc.dependency.injection.UserHolder">  
		 <constructor-arg name="user" ref="superUser" />  
		</bean>
		```

	-  Java 注解配置元信息

		``` Java
			@Bean  
			public UserHolder userHolder(User user) {  
			 return  new UserHolder(user); // 构造器注入
			}

			public class UserHolder {
				private User user;
				
				public UserHolder(){
				}
				public User(User user){
					this.user = user;
				}
				
			}
		```

	-  API 配置元信息
		``` JAVA
			private static BeanDefinition createUserHolderBeanDefinition() {  
				BeanDefinitionBuilder definitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(UserHolder.class); 
				// API 构造器注入
			 definitionBuilder.addConstructorArgReference("superUser");  
			 return definitionBuilder.getBeanDefinition();  
			}

			// 创建 BeanFactory 容器  
			AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();  

			// 生成 UserHolder 的 BeanDefinition  
			BeanDefinition userHolderBeanDefinition = createUserHolderBeanDefinition();  
			// 注册 UserHolder 的 BeanDefinition  
			applicationContext.registerBeanDefinition("userHolder", userHolderBeanDefinition);

		```

- 自动模式
	- constructor
		``` xml
		<bean class="org.geekbang.thinking.in.spring.ioc.dependency.injection.UserHolder" autowire="constructor"  
		 >  
		<!--        <property name="user" ref="superUser" /> 替换成 autowiring 模式 -->  
		 </bean>
		```

### 字段注入
- 手动模式
	- Java 注解配置元信息
		- @Autowired
			-  @Autowired 会忽略静态字段
v
		- @Resource

		-  @Inject（可选）



### 方法注入
 **参数名即 Bean 名称，如果查不到会使用类型查找**
 
- 手动模式	
	- Java 注解配置元信息
		- @Autowired
		``` java
		public class AnnotationDependencyMethodInjectionDemo {  
  
			private UserHolder userHolder;  

			private UserHolder userHolder2;  

			@Autowired  
			public void init1(UserHolder userHolder) {  
				this.userHolder = userHolder;  
			}
		}
		```

		- @Resource

		``` java
		public class AnnotationDependencyMethodInjectionDemo {  
  
			private UserHolder userHolder;  

			private UserHolder userHolder2;  

			@Resource  
			public void init2(UserHolder userHolder2) {  
				this.userHolder2 = userHolder2;  
			}
		}
		```

		- @Inject（可选）
		
 
		- @Bean
		
		``` java
		public class AnnotationDependencyMethodInjectionDemo {  
  
			private UserHolder userHolder;  

			private UserHolder userHolder2;  

			@Bean  
			public UserHolder userHolder(User user) {  
				return new UserHolder(user);  
			}
		}
		```


### 接口回调注入
- Aware 系列接口回调
	- 自动模式

| 內建接口 | 说明 | 
|-|-|
| BeanFactoryAware | 获取 IoC 容器 - BeanFactory |
| ApplicationContextAware | 获取 Spring 应用上下文 - ApplicationContext 对象 |
| EnvironmentAware | 获取 Environment 对象 |
| ResourceLoaderAware | 获取资源加载器 对象 - ResourceLoader |
| BeanClassLoaderAware | 获取加载当前 Bean Class 的 ClassLoader |
| BeanNameAware | 获取当前 Bean 的名称 |
| MessageSourceAware | 获取 MessageSource 对象，用于 Spring 国际化 |
| ApplicationEventPublisherAware | 获取 ApplicationEventPublishAware 对象，用于 Spring 事件 |
| EmbeddedValueResolverAware | 获取 StringValueResolver 对象，用于占位符处理 |

```java
	public class AwareInterfaceDependencyInjectionDemo implements BeanFactoryAware, ApplicationContextAware {  

		private static BeanFactory beanFactory;  

		 private static ApplicationContext applicationContext;  


		 public static void main(String[] args) {  

				// 创建 BeanFactory 容器  
		 AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();  
		 // 注册 Configuration Class（配置类） -> Spring Bean  
		 context.register(AwareInterfaceDependencyInjectionDemo.class);  

		 // 启动 Spring 应用上下文  
		 context.refresh();  

		 System.out.println(beanFactory == context.getBeanFactory());  
		 System.out.println(applicationContext == context);  

		 // 显示地关闭 Spring 应用上下文  
		 context.close();  
		 }  

			@Override  
		 public void setBeanFactory(BeanFactory beanFactory) throws BeansException {  
				AwareInterfaceDependencyInjectionDemo.beanFactory = beanFactory;  
		 }  

			@Override  
		 public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {  
				AwareInterfaceDependencyInjectionDemo.applicationContext = applicationContext;  
		 }  
	}
```


### 依赖注入类型选择
- 低依赖：构造器注入
- 多依赖：Setter 方法注入
- 便利性：字段注入
- 声明类：方法注入

### 基础类型注入
- 原生类型（Primitive）：boolean、byte、char、short、int、float、long、double
- 标量类型（Scalar）：Number、Character、Boolean、Enum、Locale、Charset、Currency、 Properties、UUID
- 常规类型（General）：Object、String、TimeZone、Calendar、Optional 等
- Spring 类型：Resource、InputSource、Formatter 等

``` XML
	<bean id="user" class="org.geekbang.thinking.in.spring.ioc.overview.domain.User">  
	 <property name="id" value="1"/>  
	 <property name="name" value="小马哥"/>   
	 <property name="city" value="HANGZHOU"/> 
	</bean>
```

``` JAVA
	public class User {
		private Long id;  
  
		private String name;  

		private City city;
	}
```


### 集合类型注入
- 数组类型（Array）：原生类型、标量类型、常规类型、Spring 类型

``` XML
	<bean id="user" class="org.geekbang.thinking.in.spring.ioc.overview.domain.User">  
	 <property name="id" value="1"/>  
	 <property name="name" value="小马哥"/>   
	 <property name="workCities" value="BEIJING,HANGZHOU"/>  
	</bean>
```

``` JAVA
	public class User {
		private Long id;  
  
		private String name;  

		private City[] workCities;
	}
```

- 集合类型（Collection）
	- Collection：List、Set（SortedSet、NavigableSet、EnumSet）
	
	``` XML
		<bean id="user" class="org.geekbang.thinking.in.spring.ioc.overview.domain.User">  
		 <property name="id" value="1"/>  
		 <property name="name" value="小马哥"/>   
		 <property name="lifeCities">  
			 <list> 
				 <value>BEIJING</value>  
			 	 <value>SHANGHAI</value>  
			 </list>
		 </property>
		</bean>
	```

	``` JAVA
		public class User {
			private Long id;  

			private String name;  

			private List<City> lifeCities;
		}
	```

	-  Map：Properties

### 限定注入
- 使用注解 @Qualifier 限定
	- 通过 Bean 名称限定
		``` JAVA
		@Autowired  
		private User user; // superUser -> primary =true  

		@Autowired  
		@Qualifier("user") // 指定 Bean 名称或 ID  
		private User namedUser;

		// 输出 superUser Bean  
		System.out.println("demo.user = " + demo.user);  
		// 输出 user Bean  
		System.out.println("demo.namedUser = " + demo.namedUser);
		```

	- 通过分组限定
	``` JAVA
		@Autowired  
		private User user; // superUser -> primary =true  


		@Autowired  
		@Qualifier("user") // 指定 Bean 名称或 ID  
		private User namedUser;

		@Bean  
		@Qualifier // 进行逻辑分组  
		public User user1() {  
			return createUser(7L);  
		}  

		@Bean  
		@Qualifier // 进行逻辑分组  
		public static User user2() {  
			return createUser(8L);  

		}

		private static User createUser(Long id) {  
			User user = new User();  
		 user.setId(id);  
		 return user;  
		}

		@Autowired  
		private Collection<User> allUsers; // 2 Beans = user + superUser  

		@Autowired  
		@Qualifier  
		private Collection<User> qualifiedUsers; // 2 Beans = user1 + user2 

		// 输出 superUser Bean  
		System.out.println("demo.user = " + demo.user);  
		// 输出 user Bean  
		System.out.println("demo.namedUser = " + demo.namedUser);
		// 输出 superUser user user1 user2  
		System.out.println("demo.allUsers = " + demo.allUsers);  
		// 输出 user1 user2  
		System.out.println("demo.qualifiedUsers = " + demo.qualifiedUsers);
	```


- 基于注解 @Qualifier 扩展限定
	- 自定义注解 
	
		``` java
			/**  
			 * 用户组注解，扩展 {@link Qualifier @Qualifier}  
			 * * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>  
			 * @since  
			 */  
			@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})  
			@Retention(RetentionPolicy.RUNTIME)  
			@Inherited 
			@Documented  
			@Qualifier  
			public @interface UserGroup {  
			}
		```

		``` JAVA
		@Autowired  
		private User user; // superUser -> primary =true  


		@Autowired  
		@Qualifier("user") // 指定 Bean 名称或 ID  
		private User namedUser;

		@Bean  
		@Qualifier // 进行逻辑分组  
		public User user1() {  
			return createUser(7L);  
		}  

		@Bean  
		@Qualifier // 进行逻辑分组  
		public static User user2() {  
			return createUser(8L);  

		}
	
		@Bean  
		@UserGroup  
		public static User user3() {  
			return createUser(9L);  
		}  

		@Bean  
		@UserGroup  
		public static User user4() {  
			return createUser(10L);  
		}

		private static User createUser(Long id) {  
			User user = new User();  
		 user.setId(id);  
		 return user;  
		}

		@Autowired  
		private Collection<User> allUsers; // 2 Beans = user + superUser  

		@Autowired  
		@Qualifier  
		private Collection<User> qualifiedUsers; // 2 Beans = user1 + user2  -> 4 Beans = user1 + user2 + user3 + user4

		@UserGroup  
		private Collection<User> groupedUsers; // 2 Beans = user3 + user4

		// 输出 superUser Bean  
		System.out.println("demo.user = " + demo.user);  
		// 输出 user Bean  
		System.out.println("demo.namedUser = " + demo.namedUser);
		// 输出 superUser user user1 user2  
		System.out.println("demo.allUsers = " + demo.allUsers);  
		// 输出 user1 user2  
		System.out.println("demo.qualifiedUsers = " + demo.qualifiedUsers);
		// 输出 user3 user4  
		System.out.println("demo.groupedUsers = " + demo.groupedUsers);
		```

		- 如 Spring Cloud @LoadBalanced
			``` java
			@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})   
			@Retention(RetentionPolicy.RUNTIME)  
			@Documented  
			@Inherited  
			@Qualifier  
			public @interface LoadBalanced {  
			}
			```
			
### 延迟依赖注入
- 使用 API ObjectFactory 延迟注入
	- 单一类型
	- 集合类型
		``` java

			@Autowired  
			private ObjectProvider<User> userObjectProvider; // 延迟注入  

			@Autowired  
			private ObjectFactory<Set<User>> usersObjectFactory;

			// 期待输出 superUser Bean  
			System.out.println("demo.user = " + demo.user);
			// 期待输出 superUser user Beans  
			System.out.println("demo.usersObjectFactory = " + demo.usersObjectFactory.getObject());
		```


- 使用 API ObjectProvider 延迟注入（推荐）
	- 单一类型
	- 集合类型
		``` java
			@Autowired  
			@Qualifier("user")  
			private User user; // 实时注入  

			@Autowired  
			private ObjectProvider<User> userObjectProvider; // 延迟注入

			// 输出 superUser Bean  
			System.out.println("demo.user = " + demo.user);  
			// 输出 superUser Bean  
			System.out.println("demo.userObjectProvider = " + demo.userObjectProvider.getObject()); // 继承 ObjectFactory
		```
	

### 依赖处理过程
- 入口 - DefaultListableBeanFactory#resolveDependency
##### DefaultListableBeanFactory#resolveDependency
``` java
	@Override  
	@Nullable  
	public Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName,  
	 @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {  

	   descriptor.initParameterNameDiscovery(getParameterNameDiscoverer()); 
		// descriptor.getDependencyType() 获取当前注入的类的 class
		// Optional ObjectFatctory JSR330 会特殊处理
	 if (Optional.class == descriptor.getDependencyType()) {  
		  return createOptionalDependency(descriptor, requestingBeanName);  
	 }  
	   else if (ObjectFactory.class == descriptor.getDependencyType() ||  
			 ObjectProvider.class == descriptor.getDependencyType()) {  
		  return new DependencyObjectProvider(descriptor, requestingBeanName);  
	 }  
	   else if (javaxInjectProviderClass == descriptor.getDependencyType()) {  
		  return new Jsr330Factory().createDependencyProvider(descriptor, requestingBeanName);  
	 }  
	   else {  
		   // 如果延迟注入 @Lazy 会返回一个 cglib 代理对象
		  Object result = getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(  
				descriptor, requestingBeanName);  
	 if (result == null) {  
		 	// 非延迟注入查找到需要注入的类的实例 doResolveDependency
			 result = doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);  
	 }  
		  return result;  
	 }  
	}
```

##### DefaultListableBeanFactory#doResolveDependency
```java
	@Nullable  
	public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,  
	 @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {
		try {
			...
			// 判断当前注入的类是不是多个类（集合 Map 数组 Stream） 
			Object multipleBeans = resolveMultipleBeans(descriptor, beanName, autowiredBeanNames, typeConverter);  
			if (multipleBeans != null) {  
			   return multipleBeans;  
			}  
			
			// 查找当前类需要自动注入的类
			Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);  
			if (matchingBeans.isEmpty()) {  
			   if (isRequired(descriptor)) {  
				  raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);  
			 }  
			   return null;  
			}
			
			// 如果查找到多个类，需要确定注入的是哪一个 （Primary 优先级）
			// 集合类型注入上面已经返回 multipleBeans
			if (matchingBeans.size() > 1) {  
   autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor); 
				 if (autowiredBeanName == null) {  
					  if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {  
						 return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);  
				 }  
					  else {  
						 // In case of an optional Collection/Map, silently ignore a non-unique case:  
				 // possibly it was meant to be an empty collection of multiple regular beans // (before 4.3 in particular when we didn't even look for collection beans). return null;  
				 }  
				   }
					// 取出找到的对象
				   instanceCandidate = matchingBeans.get(autowiredBeanName);  
				}  
				else {
					...
				}
			if (autowiredBeanNames != null) {  
			   autowiredBeanNames.add(autowiredBeanName);  
			}  
			if (instanceCandidate instanceof Class) {  
				// 底层调用 当前 BeanFacotry.getBean(beanName) 获取 Bean 的实例
			   instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);  
			}
			Object result = instanceCandidate;
			...
			// 返回 Bean 实例
			return result;
		}
		finally {  
   ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);  
		}
	}
```

##### DefaultListableBeanFactory#findAutowireCandidates
``` java
	protected Map<String, Object> findAutowireCandidates(  
		  @Nullable String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {
		
		...
		
			// 通过类型查找当前 BeanFactory 中注册好的 BeanName 列表
		String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors( 
		Map<String, Object> result = new LinkedHashMap<>(candidateNames.length);
      this, requiredType, true, descriptor.isEager());
		// 遍历BeanNames，通过 BeanName 底层通过掉还用 beanFactory.getBean() 查找到当前对象放入 result Map 中
		for (String candidate : candidateNames) {  
		   if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {  
			  addCandidateEntry(result, candidate, descriptor, requiredType); 
	 		}  
		}
		
		...
		
		
	}
```


- 依赖描述符 - DependencyDescriptor
	
	``` java
		// 被注入的类
		private final Class<?> declaringClass;  
		// 方法注入
		@Nullable  
		private String methodName;  
		// 参数注入
		@Nullable  
		private Class<?>[] parameterTypes;  
		// 参数位置
		private int parameterIndex;  
		// 字段注入
		@Nullable  
		private String fieldName;
		// @Autowired(required=true)
		private final boolean required;  
  		// @Lazy
		private final boolean eager;
	```

- 自定绑定候选对象处理器 - AutowireCandidateResolver

### @Autowired 注入
1. 依赖处理过程得到被注入的对象
2. 通过依赖注入的对象找到注入的字段


- @Autowired 注入规则
	- 非静态字段
	- 非静态方法
	- 构造器
- @Autowired 注入过程
	- 元信息解析
		
		``` java
			public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {  
					// 构建 Bean 的元数据
				   InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);  
				 try {  
					 // AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement#inject
					  metadata.inject(bean, beanName, pvs);  
				 }  
				   catch (BeanCreationException ex) {  
					  throw ex;  
				 }  
				   catch (Throwable ex) {  
					  throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);  
				 }  
				   return pvs;  
			}


			private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {  
			   // Fall back to class name as cache key, for backwards compatibility with custom callers. 
				// 优先从缓存当中获取
			 String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());  
				 // Quick check on the concurrent map first, with minimal locking.  
				 InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);  
				 if (InjectionMetadata.needsRefresh(metadata, clazz)) {  
					  synchronized (this.injectionMetadataCache) {  
						 metadata = this.injectionMetadataCache.get(cacheKey);  
				 if (InjectionMetadata.needsRefresh(metadata, clazz)) {  
							if (metadata != null) {  
							   metadata.clear(pvs);  
				 }  
				// 构建元数据 
				metadata = buildAutowiringMetadata(clazz);  
				 this.injectionMetadataCache.put(cacheKey, metadata);  
					}  
				  }  
			   }  
			   return metadata;  
			}

		```
		
		- 构建元数据
			> AutowiredAnnotationBeanPostProcessor#postProcessMergeBeanDefinition 会先调用，内部已经调用了 **findAutowiredAnnotation** ，将结果缓存。并将父类的 BeanDefinition 合并。这里可以直接从缓存中取
		
		``` java
			private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {  
			   if (!AnnotationUtils.isCandidateClass(clazz, this.autowiredAnnotationTypes)) {  
				  return InjectionMetadata.EMPTY;  
			 }  

			   List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();  
			 Class<?> targetClass = clazz;  

			 do {  
				  final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();  
			
			// 处理每一个属性
			 ReflectionUtils.doWithLocalFields(targetClass, field -> {  
				 	// 查找属性是否打上了 @Autowired 注解
					 MergedAnnotation<?> ann = findAutowiredAnnotation(field); 
			 if (ann != null) {  
				 	// static 属性不支持 @Autowired 注入
						if (Modifier.isStatic(field.getModifiers())) {  
						   if (logger.isInfoEnabled()) {  
							  logger.info("Autowired annotation is not supported on static fields: " + field);  
			 }  
						   return;  
			 }  
						boolean required = determineRequiredStatus(ann);  
			 currElements.add(new AutowiredFieldElement(field, required));  
			 }  
				  });  

			// 处理每一个方法
			 ReflectionUtils.doWithLocalMethods(targetClass, method -> {  
					 Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);  
			 if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {  
						return;  
			 }  
				 // 查找方法上是否打上了 @Autowired 注解
					 MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);  
			 if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {  
				 // static 方法不支持 @Autowired 注入
						if (Modifier.isStatic(method.getModifiers())) {  
						   if (logger.isInfoEnabled()) {  
							  logger.info("Autowired annotation is not supported on static methods: " + method);  
			 }  
						   return;  
			 }  
						if (method.getParameterCount() == 0) {  
						   if (logger.isInfoEnabled()) {  
							  logger.info("Autowired annotation should only be used on methods with parameters: " +  
									method);  
			 }  
						}  
						boolean required = determineRequiredStatus(ann);  
			 PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);  
			 currElements.add(new AutowiredMethodElement(method, required, pd));  
			 }  
				  });  

			 elements.addAll(0, currElements);  
			 targetClass = targetClass.getSuperclass();  
			 }  
				// 递归调用查找父类需要注入的属性、方法
			   while (targetClass != null && targetClass != Object.class);  
			
			// 返回需要自动注入的元数据
			 return InjectionMetadata.forElements(elements, clazz);  
			}
		```
  
	- 依赖查找
	- 依赖注入（字段、方法）
		- AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement#inject
		
		``` java
			@Override  
			protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {  
			   Field field = (Field) this.member;  
			 Object value;  
			 if (this.cached) {  
				  value = resolvedCachedArgument(beanName, this.cachedFieldValue);  
			 }  
			   else {  
				  DependencyDescriptor desc = new DependencyDescriptor(field, this.required);  
			 desc.setContainingClass(bean.getClass());  
			 Set<String> autowiredBeanNames = new LinkedHashSet<>(1);  
			 Assert.state(beanFactory != null, "No BeanFactory available");  
			 TypeConverter typeConverter = beanFactory.getTypeConverter();  
			 try {  
				 // 调用 DefaultListableBeanFactory.resolveDependency 方法获取 Bean 实例
					 value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);  
			 }  
				  catch (BeansException ex) {  
					 throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);  
			 }   
		   }  
			// 反射调用 set 方法注入
		   	if (value != null) {  
			  ReflectionUtils.makeAccessible(field);  
		 field.set(bean, value);  
		 	}  
		}
		```
		
		

### @Inject 注入
- AutowiredAnnotationBeanPostProcessor 可以处理的注解 有顺序（优先级）
	- @Autowired
	- @Value
	- @Inject
• 如果 JSR-330 存在于 ClassPath 中，复用 AutowiredAnnotationBeanPostProcessor 实现

### Java通用注解注入原理
- CommonAnnotationBeanPostProcessor
	- 与 AutowiredAnnotationBeanPostProcessor 类似，只是多了生命周期的管理 
	- 执行顺序倒数第三 @Autowired 执行第三
	- 注入注解
		- javax.xml.ws.WebServiceRef
		- javax.ejb.EJB
		- javax.annotation.Resource
	- 生命周期注解
		- javax.annotation.PostConstruct
		- javax.annotation.PreDestroy

### 自定义依赖注入注解
- 基于 AutowiredAnnotationBeanPostProcessor 实现
	``` java
		@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.FIELD})  
		@Retention(RetentionPolicy.RUNTIME)  
		@Documented  
		@Autowired  
		public @interface MyAutowired {  

			/**  
		 * Declares whether the annotated dependency is required. * <p>Defaults to {@code true}.  
		 */ boolean required() default true;  
		}

		//    @MyAutowired  
//    private Optional<User> userOptional; // superUser
	```


- 自定义实现
	- 生命周期处理
		- InstantiationAwareBeanPostProcessor
		- MergedBeanDefinitionPostProcessor
	- 元数据
		- InjectedElement
		- InjectionMetadata