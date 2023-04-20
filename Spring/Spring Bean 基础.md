 ### Spring Bean 定义
#### 什么是 BeanDefinition
- BeanDefinition 是 Spring 中定义 Bean 的配置元信息接口
	- Bean 的类名
	- Bean 的行为配置元素，作用域、自动绑定模式、生命周期回调
	- 其他 Bean 引用（合作者 / 依赖）
	- 配置设置


### BeanDefinition 元信息

|属性(Property) | 说明 |
| - | - |
|  Class  | Bean 全类名，必须是具体类，不能用抽象类或接口 |
|  Name  |  Bean 的名称或者 ID |
|  Scope  | Bean 的作用域(如:singleton、prototype 等)  |
|  Constructor arguments  |  Bean 构造器参数(用于依赖注入) |
|  Properties  |   Bean 属性设置(用于依赖注入) |
|  Autowiring mode  |  Bean 自动绑定模式(如:通过名称 byName) |
|  Lazy initialization mode  |  Bean 延迟初始化模式(延迟和非延迟) |
|  Initialization method  |  Bean 初始化回调方法名称 |
|  Destruction method  |  Bean 销毁回调方法名称 |


### 构建 BeanDefinition
- BeanDefinitionBuilder
``` java
// 获取 BeanDefinitionBuilder
BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(Bean.class);
// 属性设置
builder.addPropertyValue("id", 1);
builder.addPropertyValue("name", "小马哥");
// 获取 BeanDefinition 实例
BeanDefinition beanDefinition = builder.getBeanDefinition();

```

- AbstractBeanDefinition 及其派生类

``` java
GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
// 设置 Bean 类型
beanDefinition.setBeanClass(Bean.class);
// 通过 MutablePropertyValues 批量更新
MutablePropertyValues propertyValues = new MutablePropertyValues();
propertyValues.add("id",1).add("name","小马哥");
// 设置属性
beanDefinition.setPropertyValues(propertyValues);
```

### 命名 Bean
#### Bean 的名称
 - 每个 Bean 拥有一个或多个标识符(identifiers)，这些标识符在 Bean 所在的容器必须是 唯一的。通常，一个 Bean 仅有一个标识符，如果需要额外的，可考虑使用别名(Alias)来 扩充。
 - 在基于 XML 的配置元信息中，开发人员可用 id 或者 name 属性来规定 Bean 的 标识符。通 常 Bean 的 标识符由字母组成，允许出现特殊字符。如果要想引入 Bean 的别名的话，可在 name 属性使用半角逗号(“,”)或分号(“;”) 来间隔。
 - Bean 的 id 或 name 属性并非必须制定，如果留空的话，容器会为 Bean 自动生成一个唯一 的名称。Bean 的命名尽管没有限制，不过官方建议采用驼峰的方式，更符合 Java 的命名约 定。

#### Bean 名称生成器(BeanNameGenerator)
- Spring Framework 2.0.3 引入，框架內建两种实现
	- **DefaultBeanNameGenerator**：默认通用 BeanNameGenerator 实现
		- generateBeanName()
			- 如果全路径类名为空
				- 父Bean名字存在,假设为Parent,generatedBeanName = "Parent$child"  
				- 工厂类Bean存在,假设为FactoryBean,generatedBeanName = "FactoryBean$created"
			- 为内嵌Bean,id = generatedBeanName#随机数
			- 非内嵌Bean,id = generatedBeanName#递增数
	- **AnnotationBeanNameGenerator**：基于注解扫描的 BeanNameGenerator 实现，起始于Spring Framework 2.5
		- 判断是为注解-@Component等,且指定value值,直接返回value值
		- 否则根据spring规则生成
			- 如果发现类的前两个字符都是大写，则直接返回类名; MYService
			- 将类名的第一个字母转成小写，然后返回 MyService -> myService


### Spring Bean 的别名
 #### Bean 别名(Alias)的价值
 - 复用现有的 BeanDefinition
 - 更具有场景化的命名方法
``` xml
	<alias name="myApp-dataSource" alias="subsystemA-dataSource"/> 
	<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```


### 注册 Spring Bean
- BeanDefinition 注册
	- XML 配置元信息
		- \<bean name="" .../\>
	- Java 注解配置元信息
		- @Bean
		- @Component
		- @Import
	- Java API 配置元信息
		- 命名方式：BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)
		- 非命名方式： BeanDefinitionReaderUtils#registerWithGeneratedName(AbstractBeanDefinition,BeanDefinitionRegistry)
		- 配置类方式:AnnotatedBeanDefinitionReader#register(Class...)

### 实例化 Spring Bean （Instantiation）
- 常规方式
	- 通过构造器(配置元信息:XML、Java 注解和 Java API )
	- 通过静态工厂方法(配置元信息:XML 和 Java API )
	- 通过 Bean 工厂方法(配置元信息:XML和 Java API )
	- 通过 FactoryBean(配置元信息:XML、Java 注解和 Java API )
- 特殊方式
	- 通过 ServiceLoaderFactoryBean(配置元信息:XML、Java 注解和 Java API )
	- 通过 AutowireCapableBeanFactory#createBean(java.lang.Class, int, boolean)
	- 通过 BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)

### 初始化 Spring Bean  
- **@PostConstruct**

```java
	// 1. 基于 @PostConstruct 注解  
	@PostConstruct  
	public void init() {  
		System.out.println("@PostConstruct : UserFactory 初始化中...");  
	}
```


- 实现 **InitializingBean** 接口的 afterPropertiesSet() 方法

``` java
	public class DefaultUserFactory implements InitializingBean {
		@Override  
		public void afterPropertiesSet() throws Exception {  
			System.out.println("InitializingBean#afterPropertiesSet() : UserFactory 初始化中...");  
		}
	}
```

- 自定义初始化方法 
	- XML 配置：\<bean **init-method** = "init"... \>
	- @Bean(**initMethod**="xxx")
	
	``` java
	public class DefaultUserFactory {
		public void initUserFactory() {  
			System.out.println("自定义初始化方法 initUserFactory() : UserFactory 初始化中...");  
		}
	}

	@Bean(initMethod = "initUserFactory")  
	public UserFactory userFactory() {  
		return new DefaultUserFactory();  
	}
	```
	- Java API：**AbstarctBeanDefinition#setInitMethodName(String)**

#### 初始化顺序
PostContruct->afterPropertiesSet->自定义init方法

### 延迟初始化 Spring Bean
- XML 配置：\<bean **lazy-init**="true" ... /\>
- Java 注解： **@Lazy(true)** 可以解决[[循环依赖]]

applicationContext.refresh() -> finishBeanFacotryInitialization(beanFactory) 实例化初始化所有非延迟加载的单例对象

### 销毁 Spring Bean
- **@PreDestroy** 标注方法

``` java	
	public class DefaultUserFactory {
		@PreDestroy  
		public void preDestroy() {  
			System.out.println("@PreDestroy : UserFactory 销毁中...");  
		}
	}
```

- 实现 **DisposableBean** 接口的 destroy() 方法

``` java
	public class DefaultUserFactory implements DisposableBean{
		@Override  
		public void destroy() throws Exception {  
			System.out.println("DisposableBean#destroy() : UserFactory 销毁中...");  
		}
	}
```

- 自定义销毁方法
	- XML 配置：\<bean destroy=”destroy” ... /\>
	- Java 注解：@Bean(destroy=”destroy”)
	- Java API：AbstractBeanDefinition#setDestroyMethodName(String)

#### 销毁顺序
@PreDestroy -> DisposableBean#destroy()  -> 自定义销毁方法

### 垃圾回收 Spring Bean
1. 关闭 Spring 容器 （应用上下文）
2. 执行 GC
3. Spring Bean 覆盖的 finalize() 方法被回调


### 如何注册一个 Spring Bean
- 通过 BeanDefinition [[#注册 Spring Bean]]  
- 外部单体对象注册
``` java
	// 创建 BeanFactory 容器  
	AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();  
	// 创建一个外部 UserFactory 对象  
	UserFactory userFactor y = new DefaultUserFactory();  
	SingletonBeanRegistry singletonBeanRegistry = applicationContext.getBeanFactory();  
	// 注册外部单例对象  
	singletonBeanRegistry.registerSingleton("userFactory", userFactory);  
	// 启动 Spring 应用上下文  
	applicationContext.refresh();  

	// 通过依赖查找的方式来获取 UserFactory  
	UserFactory userFactoryByLookup = applicationContext.getBean("userFactory", UserFactory.class);  
	System.out.println("userFactory  == userFactoryByLookup : " + (userFactory == userFactoryByLookup));  

	// 关闭 Spring 应用上下文  
	applicationContext.close();
```

### 什么是 Spring BeanDefinition
 - [[#Spring Bean 定义]] 
 - [[#BeanDefinition 元信息]]
  