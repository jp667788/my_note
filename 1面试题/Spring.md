## Spring
### Spring Bean 生命周期
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-05-15-154744.jpg)

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-05-15-154827.jpg)

Bean 加载过程：
- 创建对象
- 属性赋值
- 初始化
	- 检查 Aware 配置
	- 前置处理、后置处理
	- 调用 Init Method
	- 后置处理
- 注销接口注册

### Spring AOP

### Spring 循环依赖



## Spring Boot
### Spring Boot 特性
- 独立运行的 Spring 应用
- 嵌入 Tomcat 
- 提供限定性的 starter 依赖简化配置
- 自动化配置

### 什么是自动装配
- Spring boot 定义了一套接口规范	
	- SpringBoot 在启动时会扫描外部引用 jar 保重的 META-INF/spring.factories
	- 再讲配置的类型信息加载到 Spring 容器
	- 没有的情况下，需求xml配置

### 自动装配原理
- @SpringBootApplication 
	- 说明这个类是 SpringBoot 的主配置类
	- @SpringBootConfiguration（@Configuration）
		- 允许在上下文中注册额外的 bean 或 导入其他配置类
	- @ComponentScan
		- 扫描呗 @Componet 注解的 bean
	- @EnableAutoConfiguration
		- 启用 SpringBoot的自动配置机制
		- @Import{AutoConfigurationImportSelector.class}
			- 加载 所有 META-INF /Spring.factories 的配置类，结合 @Conditional 注解

#### @EnableAutoConfiguration:实现自动装配的核心注解
- AutoConfigurationImportSelector （
	- ConfigurationClassPostProcessor
		- BeanDefinitionRegistryPostProcessor
			- BeanFactoryPostProcessor
	- 实现了 ImportSelector 接口的 selectImports() 方法
		- 加载 spring.factories 下路径的类
			- exclude、excludeName
			- @Condition

### Spring boot starter 
- 可以看做一个依赖描述符
	- 引入模块所需的相关 jar 包
	- 自动装配模块所需的配置到容器