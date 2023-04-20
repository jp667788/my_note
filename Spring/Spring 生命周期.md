### Spring Bean 元信息配置阶段
- BeanDefinition 配置
	- 面向资源
		- XML 配置
			- XmlBeanDifinitionReader
		- Properties 资源配置
			- PropertiesBedifinitionReader 
	-  面向注解
		-  @Configuration 
		-  @Bean
		-  @Component
	-  面向 API


### Spring Bean 元信息解析阶段
- 面向资源 BeanDefinition 解析
	- BeanDefinitionReader
	- XML 解析器 - BeanDefinitionParser
- 面向注解 BeanDefinition 解析
	- AnnotatedBeanDefinitionReader
		``` java
			DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();  
			// 基于 Java 注解的 AnnotatedBeanDefinitionReader 的实现  
			AnnotatedBeanDefinitionReader beanDefinitionReader = new AnnotatedBeanDefinitionReader(beanFactory);  
			int beanDefinitionCountBefore = beanFactory.getBeanDefinitionCount();  
			// 注册当前类（非 @Component class）  
			beanDefinitionReader.register(AnnotatedBeanDefinitionParsingDemo.class);  
			int beanDefinitionCountAfter = beanFactory.getBeanDefinitionCount();  
			int beanDefinitionCount = beanDefinitionCountAfter - beanDefinitionCountBefore;  
			System.out.println("已加载 BeanDefinition 数量：" + beanDefinitionCount);  
			// 普通的 Class 作为 Component 注册到 Spring IoC 容器后，通常 Bean 名称为 annotatedBeanDefinitionParsingDemo  
			// Bean 名称生成来自于 BeanNameGenerator，注解实现 AnnotationBeanNameGenerator  
			AnnotatedBeanDefinitionParsingDemo demo = beanFactory.getBean("annotatedBeanDefinitionParsingDemo",  
			 AnnotatedBeanDefinitionParsingDemo.class);
		```


### Spring Bean 注册阶段
- BeanDefinition 注册接口
	- BeanDefinitionRegistry
		- DefaultListableBeanFactory
		``` java
			@Override  
			public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)  
				  throws BeanDefinitionStoreException {
				···
				BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
				// 是否已存在相同名称的 Bean
				if (existingDefinition != null) {
					if (!isAllowBeanDefinitionOverriding()) {  
					   throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);  
					}
				...
				// 更新成新的 BeanDifinition
				this.beanDefinitionMap.put(beanName, beanDefinition);
				} else {
					// 是否已开始创建
					if (hasBeanCreationStarted()) {
						// 加锁
						synchronized (this.beanDefinitionMap) {  
							// 存储 BeanDefinition
						   this.beanDefinitionMap.put(beanName, beanDefinition); 
							// 存储 BeanDefinitionName
						 List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);  
						 updatedDefinitions.addAll(this.beanDefinitionNames); 
						 updatedDefinitions.add(beanName);  
						 this.beanDefinitionNames = updatedDefinitions;  
						 removeManualSingletonName(beanName);  
						}
					} else {
						// Still in startup registration phase  
						this.beanDefinitionMap.put(beanName, beanDefinition);  
						this.beanDefinitionNames.add(beanName);  
						removeManualSingletonName(beanName);
					}
				}
				
			}
		```


### Spring BeanDefinition 合并阶段
- BeanDefinition 合并
	- 父子 BeanDefinition 合并
		- 当前 BeanFactory 查找
		- 层次性 BeanFactory 查找


- RootBeanDefinition 不需要合并
- GenericBeanDefinition 需要合并
- 合并结果由 GenericBeanDifinition -> RootBeanDefinition，将子类属性覆盖或新增。

AnstractBeanFactory#getMergedBeanDefinition

```java
	@Override  
	public BeanDefinition getMergedBeanDefinition(String name) throws BeansException {  
		   String beanName = transformedBeanName(name);  
			// Efficiently check whether bean definition exists in this factory.
			// 如果没有注册，优先由父 BeanFactory 进行注册
		 if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory) {  
			  return ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(beanName);  
		 }  
		   // Resolve merged bean definition locally.
			// 当前 BeanFacotry 来注册
		 return getMergedLocalBeanDefinition(beanName);  
	}

	protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName) throws BeansException {  
	   // Quick check on the concurrent map first, with minimal locking.  
	 RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);  
	 if (mbd != null && !mbd.stale) {  
		  return mbd;  
	 }  
	   return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));  
	}

	protected RootBeanDefinition getMergedBeanDefinition(  
      String beanName, BeanDefinition bd, @Nullable BeanDefinition containingBd)  
      throws BeanDefinitionStoreException {
		synchronized (this.mergedBeanDefinitions) {
			RootBeanDefinition mbd = null;  
			RootBeanDefinition previous = null;  
			// Check with full lock now in order to enforce the same merged instance.  
			if (containingBd == null) {
			   mbd = this.mergedBeanDefinitions.get(beanName);  
			}
		}
		···
		if (bd.getParentName() == null) {  
		   // Use copy of given root bean definition. 
			// 如果本身是 root 克隆
			if (bd instanceof RootBeanDefinition) {  
			  mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();  
		 	}  
		   	else {  
			  mbd = new RootBeanDefinition(bd);  
		 	}  
		}  
		else {
			BeanDefinition pbd;  
			try {  
			   String parentBeanName = transformedBeanName(bd.getParentName()); 

			 if (!beanName.equals(parentBeanName)) {
				 // 如果还有父类 递归调用
				  pbd = getMergedBeanDefinition(parentBeanName);  
			 }  
			   else {  
				   // 用 parebtBeanFactory 
				  BeanFactory parent = getParentBeanFactory();  
			 if (parent instanceof ConfigurableBeanFactory) {  
					 pbd = ((ConfigurableBeanFactory) parent).getMergedBeanDefinition(parentBeanName);  
			 }  
				  else {  
					 throw new NoSuchBeanDefinitionException(parentBeanName,  
			 "Parent name '" + parentBeanName + "' is equal to bean name '" + beanName +  
						   "': cannot be resolved without an AbstractBeanFactory parent");  
			 }  
			   }  
			}
		}
		// Deep copy with overridden values.  
		mbd = new RootBeanDefinition(pbd);  
		mbd.overrideFrom(bd);
		···
		return mbd;
	}
```


### Spring Bean Class 加载阶段
- ClassLoader 类加载
- Java Security 安全控制
- ConfigurableBeanFactory 临时 ClassLoader

AbstractBeanFactory.java:320 doGetBean()：

``` java
	if (mbd.isSingleton()) {  
	   sharedInstance = getSingleton(beanName, () -> {  
		  try {  
			  // 创建 Bean 实例
			 return createBean(beanName, mbd, args);  
	 }  
		  catch (BeansException ex) {  
			 // Explicitly remove instance from singleton cache: It might have been put there  
	 // eagerly by the creation process, to allow for circular reference resolution. // Also remove any beans that received a temporary reference to the bean. destroySingleton(beanName);  
	 throw ex;  
	 }  
	   });  
	 bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);  
	}
```

AbstractAutowireCapableBeanFactory#createBean()：

``` java
	···
	RootBeanDefinition mbdToUse = mbd;  
  
	// Make sure bean class is actually resolved at this point, and  
	// clone the bean definition in case of a dynamically resolved Class  
	// which cannot be stored in the shared merged bean definition.  
	// Class 加载
	Class<?> resolvedClass = resolveBeanClass(mbd, beanName);  
	if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {  
	   mbdToUse = new RootBeanDefinition(mbd);  
	 mbdToUse.setBeanClass(resolvedClass);  
	}
	···
```

AbstractAutowireCapableBeanFactory#resolveBeanClass()：

```java
	protected Class<?> resolveBeanClass(final RootBeanDefinition mbd, String beanName, final Class<?>... typesToMatch)  
		  throws CannotLoadBeanClassException {  

	   try {  
		  if (mbd.hasBeanClass()) {  
			 return mbd.getBeanClass();  
	 }  
		  if (System.getSecurityManager() != null) {  
				// 安全权限校验
			  return AccessController.doPrivileged((PrivilegedExceptionAction<Class<?>>) () ->  	
				// 类加载							  
				doResolveBeanClass(mbd, typesToMatch), getAccessControlContext());  
	 }  
		  else {  
			 return doResolveBeanClass(mbd, typesToMatch);  
	 }  
	   }  
	   ···
	}
```


AbstractAutowireCapableBeanFactory#doResolveBeanClass()：

``` java
  
	private Class<?> doResolveBeanClass(RootBeanDefinition mbd, Class<?>... typesToMatch)  
		  throws ClassNotFoundException {  
		// 默认 AppClassLoader
	   ClassLoader beanClassLoader = getBeanClassLoader();  
	 ClassLoader dynamicLoader = beanClassLoader;  
	 boolean freshResolve = false;
		···
			
		···
	// Resolve regularly, caching the result in the BeanDefinition...  
	return mbd.resolveBeanClass(beanClassLoader);
	}

	public Class<?> resolveBeanClass(@Nullable ClassLoader classLoader) throws ClassNotFoundException {  
	   String className = getBeanClassName();  
	 if (className == null) {  
		  return null;  
	 }  
		// 使用 ClassLoader 加载
	   Class<?> resolvedClass = ClassUtils.forName(className, classLoader);  
	 	// 将解析出来的 Class 存在 BeanDefinitino beanClass 中
		this.beanClass = resolvedClass;  
	 	return resolvedClass;  
	}
```



### Spring Bean 实例化前阶段
- 非主流生命周期 - Bean 实例化前阶段
	- **InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation**
		- AbstractAutowireCapableBeanFactory# createBean->resolveBeforeInstantiation -> applyBeanPostProcessorsBeforeInstantiation -> **InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation**
		- 返回空时，createBean 继续往下执行，创建默认配置的类
		- 不是空时，实例化返回的新对象，createBean 返回 (类似于一个代理对象)


### Spring Bean 实例化阶段
- 实例化方式
	- 传统实例化方式
		- 实例化策略 - InstantiationStrategy 
		
		AbstractAutowireCapableBeanFactory#doCreateBean -> AbstractAutowireCapableBeanFactory#createBeanInstance().java:1162

	``` java
		···
		// 如果已解析 （非 Singleton）
		if (resolved) {  
			if (autowireNecessary) {  
				return autowireConstructor(beanName, mbd, null, null);  
			}  
			else {  
				return instantiateBean(beanName, mbd);  
			} 
		}  
  
		// Candidate constructors for autowiring? 
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);  
		// 如果存在 SmartInstantiationAwareBeanPostProcessor 、构造器注入 、指定了构造器参数 construct-arg，则执行构造器注入
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||  
			  mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {  
		   return autowireConstructor(beanName, mbd, ctors, args);  
		}  

		// Preferred constructors for default construction?  
		ctors = mbd.getPreferredConstructors();  
		if (ctors != null) {  
		   return autowireConstructor(beanName, mbd, ctors, null);  
		}  

		// No special handling: simply use no-arg constructor.  
		// 普通注入
		return instantiateBean(beanName, mbd);
	```
	
	```java
		protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {  
		   	try {  
				  Object beanInstance;  
			 final BeanFactory parent = this;  
			 // 获取 instantionStratgey  来初始化 Bean 实例
			 if (System.getSecurityManager() != null) {  
					 beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->  
						   getInstantiationStrategy().instantiate(mbd, beanName, parent),  
			 				getAccessControlContext());  
			 } else {  
					 beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);  
			 }  
				// 返回的不是 Bean Object实例 而是 BeanWrapper
				  BeanWrapper bw = new BeanWrapperImpl(beanInstance);  
			 initBeanWrapper(bw);  
			 return bw;  
			 }  
			   catch (Throwable ex) {  
				  throw new BeanCreationException(  
						mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);  
			 }  
		}
	```

	SimpleInstantiationStrategy#instantiate():

	``` JAVA
		public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {  
   			// Don't override the class with CGLIB if no overrides.  
			if (!bd.hasMethodOverrides()) {
				···
				if (System.getSecurityManager() != null) {  
					constructorToUse = AccessController.doPrivileged(  
         				(PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);  
				}  
				else {
					// 获取默认构造器
			      	constructorToUse = clazz.getDeclaredConstructor();  
				}  
					bd.resolvedConstructorOrFactoryMethod = constructorToUse;
				···
			} else {
				// Must generate CGLIB subclass.  
				return instantiateWithMethodInjection(bd, beanName, owner);
			}
		}
	```

	- 构造器依赖注入
		AbstractAutowireCapableBeanFactory#autowireConstructor()
	
	``` java
	protected BeanWrapper autowireConstructor(  
		  String beanName, RootBeanDefinition mbd, @Nullable Constructor<?>[] ctors, @Nullable Object[] explicitArgs) {  

	   return new ConstructorResolver(this).autowireConstructor(beanName, mbd, ctors, explicitArgs);  
	}
	```

	``` java
		···
		Constructor<?>[] candidates = chosenCtors;
		// 获取所有的构造器
		candidates = (mbd.isNonPublicAccessAllowed() ?  
      beanClass.getDeclaredConstructors() : beanClass.getConstructors());
		···
		// 构造器排序，根据 public 和 参数数量 排序
		AutowireUtils.sortConstructors(candidates);
		···
		for (Constructor<?> candidate : candidates) {
			···
			// 尝试获取构造器参数名称 通过 ConstructorProperties 注解
			String[] paramNames = ConstructorPropertiesChecker.evaluate(candidate, parameterCount);  
			if (paramNames == null) {
				// 通过 ParameterNameDiscoverer 获取构造器参数名称
				// Java8+ LocalVariableTableParameterNameDiscoverer 可获取接口参数
				// Java8- DefaultParameterNameDiscoverer
   				ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();  
 				if (pnd != null) {  
      				paramNames = pnd.getParameterNames(candidate);  
				}  	
			}
		}
	```
	
### Spring Bean 实例化后阶段
- Bean 属性赋值（Populate）判断
	- InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
		- 返回 true/false
			- ture：属性应该被设置到 Bean 中 （配置元信息设置为 Bean 的属性）
			- false：当前属性不会被设置，且跳过后续操作


### Spring Bean 属性赋值前阶段
- Bean 属性值元信息
	- PropertyValues
- Bean 属性赋值前回调 
	- Spring 1.2 - 5.0： #postProcessPropertyValues
	- Spring 5.1：InstantiationAwareBeanPostProcessor#postProcessProperties

### InstantiationAwareBeanPostProcessor
- **postProcessBeforeInstantiation**()在bean实例化前回调。
	- 返回实例则不对bean实例化
	- 返回null则进行spring bean实例化(doCreateBean)；
- **postProcessAfterInstantiation**()在bean实例化后在填充bean属性之前回调
	- 返回true：则进行下一步的属性填充
	- 返回false：则不进行属性填充
- **postProcessProperties**在属性赋值前的回调
	- 在applyPropertyValues之前操作可以对属性添加或修改，最后在通过applyPropertyValues应用bean对应的wapper对象

### Spring Bean Aware接口回调阶段
- Spring Aware 接口 (顺序)
	- BeanNameAware
	- BeanClassLoaderAware
	- BeanFactoryAware
	-
	- EnvironmentAware
	- EmbeddedValueResolverAware
	- ResourceLoaderAware
	- ApplicationEventPublisherAware
	- MessageSourceAware
	- ApplicationContextAware

AbstractAutowireCapableBeanFactory#invokeAwareMethods:

```java
	private void invokeAwareMethods(final String beanName, final Object bean) {  
	   if (bean instanceof Aware) {  
		  if (bean instanceof BeanNameAware) {  
			 ((BeanNameAware) bean).setBeanName(beanName);  
	 }  
		  if (bean instanceof BeanClassLoaderAware) {  
			 ClassLoader bcl = getBeanClassLoader();  
	 if (bcl != null) {  
				((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);  
	 }  
		  }  
		  if (bean instanceof BeanFactoryAware) {  
			 ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);  
	 }  
	   }  
	}
```

AbstractApplicationContext#prepareBeanFactory：

```java
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		···
		// ApplicationContextAwareProcessor 中会添加 aware 的实现
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		···
	}
```

ApplicationContextAwareProcessor#invokeAwareInterfaces：

```JAVA
	private void invokeAwareInterfaces(Object bean) {  
	   if (bean instanceof EnvironmentAware) {  
		  ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());  
	 }  
	   if (bean instanceof EmbeddedValueResolverAware) {  
		  ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);  
	 }  
	   if (bean instanceof ResourceLoaderAware) {  
		  ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);  
	 }  
	   if (bean instanceof ApplicationEventPublisherAware) {  
		  ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);  
	 }  
	   if (bean instanceof MessageSourceAware) {  
		  ((MessageSourceAware) bean).setMessageSource(this.applicationContext);  
	 }  
	   if (bean instanceof ApplicationContextAware) {  
		  ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);  
	 }  
	}
```

> 注意：ApplicationContextAwareProcessor 只有 ApplicationContext 会往 BeanFatory 自动添加，如果是 BeanFactory 需要手动添加并刷新。
> AbstractApplicatoinContext中 refresh() -> registerBeanPostProcessors() 中会 通过 beanFacotry.getBeanNamesByType 获取 BeanPostProcessor 所有的 BeanName，然后注册。

### Spring Bean 初始化前阶段
- 已完成
	- Bean 实例化
	- Bean 属性赋值
	- Bean Aware 接口回调

- 方法回调
	- BeanPostProcessor#**postProcessBeforeInitialization**
	- AbstractAutowireCapableBeanFactory#initializeBean ->applyBeanPostProcessorsBeforeInitialization 中会调用 BeanPostProcessor#**postProcessBeforeInitialization**：
	``` java
		@Override  
		public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)  
			  throws BeansException {  

		   Object result = existingBean;  
		 for (BeanPostProcessor processor : getBeanPostProcessors()) {  
			 // 执行postProcessBeforeInitialization 方法（默认返回bean 本身）
			  Object current = processor.postProcessBeforeInitialization(result, beanName);  
		 if (current == null) {  
			 // 如果是空，返回Bean本身
				 return result;  
		 }  
			 // 替换原有的 Bean，可能是同一个，也可能是新的对象
			  result = current;  
		 }  
		   return result;  
	}
	```


### Spring Bean 初始化阶段
- Bean 初始化（Initialization）
	- @PostConstruct 标注方法
		- 依赖具有注解驱动能力的 ApplicationContext
		- 通过 CommAnnotationBeanPostProcessor#postProcessBeforeInitialization 执行。**所以 @PostConstruct 在初始化前阶段，就已经完成调用**
	- 实现 InitializingBean 接口的 afterPropertiesSet() 方法
	- 自定义初始化方法

AbstractAutowireCapableBeanFactory#initializeBean ->invokeInitMethods 中会调用 afterPropertiesSet() 和 自定义初始化方法：

``` JAVA
	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)  
		  throws Throwable {  
		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {   
	   		// 执行 InitializingBean afterPropertiesSet 方法
			(InitializingBean) bean).afterPropertiesSet()
		}
	 	if (mbd != null && bean.getClass() != NullBean.class) {  
	   		String initMethodName = mbd.getInitMethodName();  
	 		if (StringUtils.hasLength(initMethodName) &&  
			 !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&  
			 !mbd.isExternallyManagedInitMethod(initMethodName)) {  
			// 执行自定义初始化方法
		  	invokeCustomInitMethod(beanName, bean, mbd);  
	 		}  
	 	}
		
	}
```

### Spring Bean 初始化后阶段
- 方法回调
	- BeanPostProcessor#postProcessAfterInitialization
		-  AbstractAutowireCapableBeanFactory#initializeBean ->applyBeanPostProcessorAfterInitialization 来执行：

		```java
		@Override  
		public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)  
			  throws BeansException {  
		   Object result = existingBean;  
			 for (BeanPostProcessor processor : getBeanPostProcessors()) {  
				  Object current = processor.postProcessAfterInitialization(result, beanName);  
			 if (current == null) {  
					 return result;  
			 }  
				  result = current;  
			 }  
			   return result;  
		}
		```

### Spring Bean 初始化完成阶段
- 方法回调
	- Spring 4.1 +：SmartInitializingSingleton#afterSingletonsInstantiated
		- SmartInitializingSingleton 通常 在 ApplicatoinContext 中使用

AbstractApplicatoinContext#refresh -> finishBeanFactoryInitialization -> DeafultListableBeanFactory -> preInstantiateSingletons

``` JAVA
	···
	for (String beanName : beanNames) {
		RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
		···
		// 注册 bean
		getBean(beanName);
	}
	
	// 此时代表所有 Bean 已完成初始化
	
	// Trigger post-initialization callback for all applicable beans...  
	for (String beanName : beanNames) {  
	   Object singletonInstance = getSingleton(beanName);  
	 if (singletonInstance instanceof SmartInitializingSingleton) {  
		  final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;  
	 if (System.getSecurityManager() != null) {  
			 AccessController.doPrivileged((PrivilegedAction<Object>) () -> {  
				smartSingleton.afterSingletonsInstantiated();  
	 return null; }, getAccessControlContext());  
	 }  
		  else {  
			  // 调用  afterSingletonsInstantiated 进行初始化后回调 
			 smartSingleton.afterSingletonsInstantiated();  
	 }  
	   }  
	}
```

> SmartInitializingSingleton#afterSingletonsInstantiated 位于 容器getBean 之后，也代表着，此时 bean 已经完全初始化了。但是 BeanPostProcessor 中的方法执行时，bean 可能还没有完全初始化。
> 因此，BeanPostProcessor 建议只对 Bean 做修改，而不是替换
> 

### Spring Bean 销毁前阶段
- 方法回调
	- DestructionAwareBeanPostProcessor#postProcessBeforeDestruction

CommonAnnotationBeanPostProcessor 继承 InitDestroyAnnotationBeanPostProcessor，InitDestroyAnnotationBeanPostProcessor 实现了 DestructionAwareBeanPostProcessor

DefaultListableBeanFatory#destoryBean() -> DispsableBeanApdaer#destroy ：
```java
	public void destroy() {
		// 找到所有的  DestructionAwareBeanPostProcessor 执行 postProcessBeforeDestruction
		if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {  
   			for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {  
      			processor.postProcessBeforeDestruction(this.bean, this.beanName);  
 			}  
		}
		// 执行完后
		if (this.invokeDisposableBean) {
			···
			// 执行 DisposableBean 接口的 destory 方法
			((DisposableBean) this.bean).destroy();
			···
		}
		if (this.destroyMethod != null) {
			// 执行自定义销毁方法
   			invokeCustomDestroyMethod(this.destroyMethod);  
		}  
		else if (this.destroyMethodName != null) {  
			Method methodToInvoke = determineDestroyMethod(this.destroyMethodName);  
 			if (methodToInvoke != null) {  
      			invokeCustomDestroyMethod(ClassUtils.getInterfaceMethodIfPossible(methodToInvoke));  
	 		}  
		}
	}
```

### Spring Bean 销毁阶段
- Bean 销毁（Destroy）
	- @PreDestroy 标注方法
		- 在 CommonAnnotationBeanPostProcessor
	- 实现 DisposableBean 接口的 destroy() 方法
	- 自定义销毁方法

### Spring Bean 垃圾收集
- Bean 垃圾回收（GC）
	- 关闭 Spring 容器（应用上下文）
	- 执行 GC
	- Spring Bean 覆盖的 finalize() 方法被回调


### BeanPostProcessor 的使用场景有哪些
BeanPostProcessor 提供 Spring Bean 初始化前和初始化后的 生命周期回调，分别对应 postProcessBeforeInitialization 以及 postProcessAfterInitialization 方法，允许对关心的 Bean 进行 扩展，甚至是替换。

加分项：其中，**ApplicationContext 相关的 Aware 回调也是基于 BeanPostProcessor 实现，即 ApplicationContextAwareProcessor 。**

### BeanFactoryPostProcessor 与 BeanPostProcessor 的区别
BeanFactoryPostProcessor 是 Spring BeanFactory（实际为 ConfigurableListableBeanFactory） 的后置处理器，用于扩展 BeanFactory，或通过 BeanFactory 进行依赖查找和依赖注入。

加分项：**BeanFactoryPostProcessor 必须有 Spring ApplicationContext 执行，BeanFactory 无法与其直接交互。**

而 BeanPostProcessor 则直接与BeanFactory 关联，属于 N 对 1 的关系 。

### BeanFactory 是怎样处理 Bean 生命周期
BeanFactory 的默认实现为 DefaultListableBeanFactory，其中 Bean生命周期与方法映射如下：
- BeanDefinition 注册阶段 - registerBeanDefinition
- BeanDefinition 合并阶段 - getMergedBeanDefinition
- Bean 实例化前阶段 - resolveBeforeInstantiation
- Bean 实例化阶段 - createBeanInstance
- Bean 实例化后阶段 - populateBean
- Bean 属性赋值前阶段 - populateBean
- Bean 属性赋值阶段 - populateBean
- Bean Aware 接口回调阶段 - initializeBean
- Bean 初始化前阶段 - initializeBean
- Bean 初始化阶段 - initializeBean
- Bean 初始化后阶段 - initializeBean
- Bean 初始化完成阶段 - preInstantiateSingletons
- Bean 销毁前阶段 - destroyBean
- Bean 销毁阶段 - destroyBean