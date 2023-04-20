### Spring IoC 依赖查找
- 根据 Bean 名称查找 
	- 实时查找 
	- 延迟查找
- 根据 Bean 类型查找
	- 单个 Bean 对象
	-  集合 Bean 对象
-  根据 Bean 名称 + 类型查找
	-  根据 Java 注解查找
	-  单个 Bean 对象
	-   集合 Bean 对象


#### 根据 Bean 名称查找 
##### 实时查找
BeanFactory.getBean("beanName")

##### 延时查找
通过 ObjectFactory\<Bean\> 获取：
1. ObjectFactory\<Bean\> objectFactory = (ObjectFactory\<Bean\>) BeanFactory.getBean("objectFactoryName");
2. Bean bean = objectFactory.getObject();

BeanFacotry.getBean 的时候，实际返回了 ObjectFoctory，并不是真实的 Bean。当调用 ObjectFactory.getObject() 时，Bean 才真正初始化，触发了 Bean 的生命周期。

#### 根据 Bean 类型查找
##### 单个 Bean
BeanFactory.getBean(Bean.class);
单个 Bean 查找出多个的话会抛出异常，使用 **Primary** 解决

##### 多个 Bean （集合类型）
Map<String,Bean> map = ListableBeanFactory.getBeanOfType(Bean.class);
key 为 beanName

#### 根据 Bean 名称 + 类型复合查找
- Bean bean = BeanFactory.getBean("beanName",Bean.class);

#### 根据 Java 注解查找
##### 单个 Bean 

##### 多个 Bean
ListableBeanFactory.getBeanWithAnnotation(AnnotationType.class)


### Spring IoC 依赖注入
 - 根据 Bean 名称注入
 - 根据 Bean 类型注入
	 - 单个 Bean 对象
	 - 集合 Bean 对象
 - 注入容器內建 Bean 对象
 - 注入非 Bean 对象
 - 注入类型
	 - 实时注入
	 - 延迟注入


#### 根据 Bean 名称注入
autowire="byName"
#### 根据 Bean 类型注入
##### 单个 Bean 对象
##### 集合 Bean 对象
autowire="byType"


### Spring IoC 依赖来源
#### 自定义 Bean
业务定义的对象
#### 内建 Bean 对象
DefaultSingletonBeanRegistry#registerSingleton手动注册的beanbean
Enviroment
#### 容器内建依赖  
DefaultListableBeanFactory 属性 resolvableDependencies 这个map里面保存的
BeanFactory

### Spring IoC 配置元信息
 - Bean 定义配置  
	- 基于 XML 文件
	- 基于 Properties 文件
	- 基于 Java 注解
	- 基于 Java API(专题讨论)
- IoC 容器配置
	- 基于 XML 文件
    - 基于 Java 注解
    - 基于 Java API (专题讨论) 
-  外部化属性配置
	-   基于 Java 注解 @Vlaue

### Spring IoC 容器
- BeanFoctory 是 ApplicationContext 的底层容器
- ApplicationContext 是具有应用特性的 BeanFactory 的超集

#### AppilcatoinContext
ApplicationContext 除了 IoC 容器角色，还有提供:
- 面向切面(AOP)
- 配置元信息(Configuration Metadata)
- 资源管理(Resources)
- 事件(Events)
- 国际化(i18n)
- 注解(Annotations)
- Environment 抽象(Environment Abstraction)


### Spring Ioc 容器的生命周期
- 启动
	- ApplicationContext.refresh()
		- 创建 BeanFactory，进行初步的初始化，加入内建依赖与内建非 Bean 的依赖
		- 执行 BeanFactoryPostProcessor
		- 执行 BeanPostProcessor
		- 启动事件广播
		- 国际化
		- 资源加载
- 运行
- 停止
	- ApplicationContext.close()
		- 销毁容器里面所有的 Bean
		- 销毁 BeanFactory


### 什么是 IoC 容器
IoC 又被称作依赖注入，可以通过构造器参数、setter 方法、工厂方法去注入对象需求的依赖对象。

### BeanFactory 和 FactoryBean 的区别
- BeanFactory 是 IoC 容器的底层实现
- FactoryBean 是创建 Bean 的一种方式，帮助实现复杂的初始化逻辑

### Spring IoC 启动做了哪些功能
 - IoC 配置元信息读取和解析
 - IoC 容器生命周期
 - Spring 事件发布
 - 国际化等






