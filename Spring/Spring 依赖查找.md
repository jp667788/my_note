### 单一类型依赖查找 BeanFactory
- 根据 Bean 名称查找
	- getBean(String)
	- Spring 2.5 覆盖默认参数：getBean(String,Object...)
- 根据 Bean 类型查找
	- Bean 实时查找
		- Spring 3.0 getBean(Class)
		- Spring 4.1 覆盖默认参数：getBean(Class,Object...)
	- Spring 5.1 Bean 延迟查找
		- getBeanProvider(Class)

	``` java
		ObjectProvider<String> objectProvider = applicationContext.getBeanProvider(String.class);
	```
	
	- getBeanProvider(ResolvableType)
-  根据 Bean 名称 + 类型查找：getBean(String,Class)

### 集合类型依赖查找 ListableBeanFactory
- 根据 Bean 类型查找
	- 获取同类型 Bean 名称列表 **通过名称不会初始化Bean**
		- getBeanNamesForType(Class)
		- getBeanNamesForType(ResolvableType) （Spring 4.2 ） 当前类和子类都匹配 
	- 获取同类型 Bean 实例列表   **会提前对Bean进行初始化，可能会出现未知的一些错误**
		- getBeansOfType(Class) 以及重载方法
- 通过注解类型查找
	- 通过注解类型查找
		- getBeanNamesForAnnotation(Class\<? extends Annotation\>)
	-  获取标注类型 Bean 实例列表 (Spring 3.0 )
		- getBeansWithAnnotation(Class\<? extends Annotation\>)
	- 获取指定名称 + 标注类型 Bean 实例 (Spring 3.0 )
		- findAnnotationOnBean(String,Class\<? extends Annotation\>)

### 层次性依赖查找 HierarchicalBeanFactory
- 双亲 BeanFactory：getParentBeanFactory()
- 层次性查找
	- 根据 Bean 名称查找
		- 基于 containsLocalBean 方法实现 (双亲递归实现)
	- 根据 Bean 类型查找实例列表
		- 单一类型：BeanFactoryUtils#**beanOfTypeIncludingAncestors**
		- 集合类型：BeanFactoryUtils#**beansOfTypeIncludingAncestors**
	- 根据 Java 注解查找名称列表
		- BeanFactoryUtils#**beanNamesForTypeIncludingAncestors**


### 延迟依赖查找
- org.springframework.beans.factory.**ObjectFactory**
- org.springframework.beans.factory.**ObjectProvider**
	- Spring 5 对 Java 8 特性扩展
		- 函数式接口
			- getIfAvailable(Supplier)
				- 未查找到的情况下，返回 Supplier.get() 作为默认实现
			- ifAvailable(Consumer)
		- Stream 扩展 - stream()
``` java
	ObjectProvider<String> objectProvider = applicationContext.getBeanProvider(String.class);  
	Iterable<String> stringIterable = objectProvider;  
	for (String string : stringIterable) {  
	   System.out.println(string);  
   }
```
> 延迟查找，我个人认为主要是给架构开发者使用的。非常典型的一个使用场景，就是SpringBoot里的自动配置，可以看看LettuceConnectionConfiguration这个类，这个类用于创建RedisClient。显然，RedisClient的创建是要根据用户的需求来创建的。  
>  
有些属性是必须的，可以在配置文件里配置，比如ip，port这种。Lettuce在创建RedisClient的时候，会从配置文件里读取这些数据来创建RedisClient。  
>  
但有些属性是非必须的，而且不能在配置文件里配置，比如开启读写分离，RedisClient在Lettuce内部，是通过一个Builder来创建的，如果要开启读写分离，这需要你在这个Builder在执行build的过程中，额外加一行：clientConfigurationBuilder.readFrom(ReadFrom.REPLICA);  
  >
问题就在这里，怎么让业务开发人员把这行代码加入到其内部的build流程中?这个问题，一种比较常见的思路，是使用模板方法，写一个抽象方法，调用它。具体的实现交给开发人员。  
  >
所以Lettuce设计了一个LettuceClientConfigurationBuilderCustomizer的类，他有一个customize方法，并且把上面提到的Builder作为这个方法的参数传递进来。开发人员，如果能去配置LettuceClientConfigurationBuilderCustomizer这样一个类，就能达到上述的目的。  
  >
但问题是，如果是开发人员去配置这样一个类，说明LettuceClientConfigurationBuilderCustomizer这个类还没有被实例化。但根据模板模式，流程中必须调用LettuceClientConfigurationBuilderCustomizer这个类的抽象方法，才能达到目的。  
  >
这个时候延迟加载，ObjectProvider的作用就体现出来了。他可以规定，他产生的是一个LettuceClientConfigurationBuilderCustomizer的对象，并且指定这个对象产生以后，做什么事情。比如调用customize方法。  
  >
如果用户配置了LettuceClientConfigurationBuilderCustomizer对象。那么在创建RedisClient的流程中，ObjectProvider就能拿到该对象，然后按照预先指定的动作执行，比如执行customize方法。  
  >
如果用户没配置，那么拿不到Bean对象，就什么都不做。  
  >
所以这个场景，我认为是延迟查找的一个典型实现


### 安全依赖查找
- 依赖查找安全性对比

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-23-151816.png)

不安全：会抛出 BeansException 异常，如 NoSuchBeanDefinitionException
> 层次性依赖查找的安全性取决于其扩展的单一或集合类型的 BeanFactory 接口，如  HierarchicalBeanFactory 继承 集合类型 和 单一类型

### 内建可查找的依赖
- AbstractApplicationContext 内建可查找的依赖

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-23-152101.png)

- 注解驱动 Spring 应用上下文内建可查找的依赖（部分） 

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-23-152253.png)
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-23-152318.png)


### 依赖查找中的经典异常
BeansException 子类型

| 异常类型 | 触发条件（举例） | 场景举例 |
|-|-|-|
| NoSuchBeanDefinitionException | 当查找 Bean 不存在于 IoC 容器 时 | BeanFactory#getBean ObjectFactory#getObject |
| NoUniqueBeanDefinitionException | 类型依赖查找时，IoC 容器存在多 个 Bean 实例 | BeanFactory#getBean(Clas s) |
| BeanInstantiationException | 当 Bean 所对应的类型非具体类时 |BeanFactory#getBean |
| BeanCreationException | 当 Bean 初始化过程中 | Bean 初始化方法执行异常 时 |
| BeanDefinitionStoreException | 当 BeanDefinition 配置元信息非 法时 | XML 配置资源无法打开时 |


### ObjectFactory 与 BeanFactory 的区别
- ObjectFactory 与 BeanFactory 均提供依赖查找的能力。 
- 不过 ObjectFactory 仅关注一个或一种类型的 Bean 依赖查找，并且**自身不具备依赖查找的能力，能力则由 BeanFactory 输出。**
- BeanFactory 则提供了单一类型、集合类型以及层次性等多种依赖查 找方式。

### BeanFactory.getBean 操作是否线程安全
BeanFactory.getBean 方法的执行是线程安全的，操作过程中会增加互斥锁 synchronized

### Spring 依赖查找与注入在来源上的区别 