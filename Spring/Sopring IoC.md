### IoC 容器的职责
- 通用职责
	- 依赖处理
		- 依赖查找
		- 依赖注入
	- 生命周期管理
		- 容器
		-  托管的资源（Java Beans 或其他资源）
	-  配置
		-  容器
		-  外部化配置
		-  托管的资源（Java Beans 或其他资源）

### IoC 容器的实现
- 主要实现
	- Java SE 
		- Java Beans 
		- Java SerivceLoader Interface
		- JNDI （Java Naming and Directory Interface）
	- Java EE
		- EJB（Enterprise Java Beans）
		- Servlet
	- 开源
		- Apache Avalon（http://avalon.apache.org/closed.html）
		- PicoContainer（http://picocontainer.com/）
		- Google Guice（https://github.com/google/guice）
		- Spring Framework（https://spring.io/projects/spring-framework）

### 传统 IoC 容器的实现
- Java Beans 作为 IoC 容器
	- 特性
		- 依赖查找
		- 生命周期管理
		- 配置原信息
		- 事件
		- 自定义
		- 资源管理
		- 持久化
	- 规范
		- JavaBeans：https://www.oracle.com/technetwork/java/javase/tech/index-jsp-138795.html
		- BeanContext：https://docs.oracle.com/javase/8/docs/technotes/guides/beans/spec/beancontext.html

### 轻量级 IoC 容器
- 定义
	- A container that can manage application code.  一个可以管理应用的代码（控制启停，生命周期）的容器
	- A container that is quick to star tup. 一个可以快速的启动的容器

	- A container that doesn't require any special deployment steps to deploy objects with init. 一个不需要靠任何特殊依赖启动的容器 
		- EJB 容器启动的时候，需要大量的 XMl 配置

	- A container that has such a light footprint and minimal API dependencies that it can be run in a variety of environments. 在不同环境下轻量级的内存占用和最小化的依赖。
		- EJB 容器依赖 Servlet API

	- A container that sets the bar for adding a managed object so low in terms of deployment effort and performance overhead that it's possible to deploy and manage fine-grained objects, as well as coarse-grained components.”

- 优点
	- Escaping the monolithic container 释放单体容器
	- Maximizing code reusability 最大化代码复用
	- Greater object or ientation	更好的面向对象
	- Greater productivity	更好的产品化
	- Better testability	更好的可测试性

### 依赖查找 vs 依赖注入

- 优缺点对比：
	- ![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-20-060441.png)
- 依赖查找：
	- 主动的查找特定的 Bean BeanFactory.getBean()

- 依赖注入
	- @Autowired 注解

### 构造器注入 vs Setter 注入

