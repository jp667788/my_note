### Spring Bean作用域
|  作用域 | 说明 |
|-|-|
| singleton | 默认 Spring Bean 作用域，一个 BeanFactory 有且仅有一个实例 |
| prototype | 原型作用域，每次依赖查找和依赖注入生成新 Bean 对象 |
| request | 将 Spring Bean 存储在 ServletRequest 上下文中 |
| session | 将 Spring Bean 存储在 HttpSession 中 |
| application | 将 Spring Bean 存储在 ServletContext 中 |

#### singleton
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-28-154629.png)

#### prototype
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-28-154700.png)

- 注意事项
	- Spring 容器没有办法管理 prototype Bean 的完整生命周期，也没有办法记录实例的存 在。销毁回调方法将不会执行，可以利用 BeanPostProcessor 进行清扫工作。

##### 结论
- Singleton Bean 无论依赖查找还是依赖注入，均为同一个对象
- Prototype Bean 无论依赖查找还是依赖注入，均为新生成对象
- 如果依赖注入集合类型的对象，Singleton Bean 和 Prototype Bean 均会存在一个
- Prototype Bean 有别于其他地方的依赖注入的 Prototype Bean
- Singleton Bean 和 Prototype Bean 均会执行初始化方法回调
- 但是 Prototype Bean 不会执行销毁方法的回调
 

#### request
- 配置
	- XML - \<bean class=”...” scope= “request” />
	- Java 注解 - @RequestScope 或 @Scope(WebApplicationContext.SCOPE_REQUEST)
- 实现
	- API - RequestScope (AbstractRequestAttributesScope#get())

#### session 
- 配置
	- XML ：\<bean class=”...” scope= “session” />
	- Java 注解 - @SessionScope 或 @Scope(WebApplicationContext.SCOPE_SESSION)
- 实现
- API - SessionScope

#### application
- 配置 
	- XML : \<bean class=”...” scope= “application” />
	- Java 注解 - @ApplicationScope 或 @Scope(WebApplicationContext.SCOPE_APPLICATION)
- 实现
	- API - ServletContextScope
	- 直接与 ServletContext 绑定


### 自定义 Bean 作用域
- 实现 Scope
	- org.springframework.beans.factory.config.Scope
- 注册 Scope
	- API
		- org.springframework.beans.factory.config.ConfigurableBeanFactory#registerScope
- 配置
	```xml
		<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
			<property name="scopes">
				<map>
					<entry key="...">
					</entry> 
				</map> 
			</property> 
		</bean>
	```

### Spring Cloud RefreshScope是如何控制Bean的动态刷新


### Spring 內建的 Bean 作用域有几种？
singleton、prototype、request、session、application 以及 websocket

### singleton Bean 是否在一个应用是唯一的
singleton bean 仅在当前 Spring IoC 容器（BeanFactory）中 是单例对象。

### “application”Bean 是否被其他方案替代
可以的，实际上，“application” Bean 与“singleton” Bean 没有 本质区别