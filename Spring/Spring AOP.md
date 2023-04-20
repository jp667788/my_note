### Java AOP 设计
- 代理模式：静态和动态代理
- 判断模式：类、方法、注解、参数、异常
- 拦截模式：前置、后置、返回、异常

#### 代理模式
- Java 静态代理
	- 常用 OOP 继承和组合相结合结合

	```java
	public interface EchoService {
		String echo(String message);
	}
	
	public class DefaultEchoService implements EchoService {
		@Override  
		public String echo(String message) {  
			return "[ECHO] " + message;  
		}
	}
	/**
	 * 静态代理
	 **/
	public class ProxyEchoService implements EchoService {  
		private final EchoService echoService; 
		public ProxyEchoService(EchoService echoService) {  
			this.echoService = echoService;  
		}
		@Override  
	 	public String echo(String message) {  
			long startTime = System.currentTimeMillis();  
	 		String result = echoService.echo(message);  
	 		long costTime = System.currentTimeMillis() - startTime;  
	 		System.out.println("echo 方法执行的实现：" + costTime + " ms.");  
	 		return result;  
		}  
	}
	```

- Java 动态代理
	- JDK 动态代理

	```
	```

	- 字节码提升 CGLIB 

### Java AOP 判断模式 （Predicate）
- 判断来源
	- 类型 （Class）
	- 方法 （Method）
	- 注解 （Annotation）
	- 参数 （Parameter）
	- 异常 （Exception）

### Java AOP 拦截器模式 （Interceptor）
- 拦截类型
	- 前置拦截 （Before）
	- 后置拦截 （After）
	- 异常拦截 （Exception）

### Spring AOP 功能概述
- 核心特性
	- 纯 Java 实现、无编译时特殊处理、不修改和控制 ClassLoader
	- 仅支持方法级别的 Join Points
	- 非完整的 AOP 实现框架
	- Spring IoC 容器整合
	- AspectJ 注解驱动整合

### Spring AOP 编程模型
- 注解驱动
- 实现：Enable 模块驱动 @EnableAspectJAutoProxy
- 注解：
	- 激活 AspectJ 自动代理：@EnableAspectJAutoProxy
	- Aspect：@Apsect
	- 配置：<aop:config >
	- PonitCut：<aop:aspect/>

###

### Spring AOP 代理实现
- JDK 动态代理
	- JDKDynamicProxy

- CGLIB 动态代理实现
	- CglibAopProxy

- AspectJ 自适应 
	- AspectJ

### AspectJ 代理
- 为什么 Spring 推荐 AspectJ 注解
	- 有很好、完整的编程模型

### AspectJ 基础
- Aspect 语法
	- Aspect
	