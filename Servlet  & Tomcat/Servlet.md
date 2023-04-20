
## Servlet 的作用

- Servlet ：Service + Applet 
- Servlet 本身只是一种规范
- Http 服务器接受到请求后，不会处理业务逻辑，交给 **Servlet 来处理业务逻辑**
- 和 http 服务器解耦


## Servlet 容器

- Servlet 容器用来管理和加载 Servlet
- Servlet 容器将请求转发到指定的 servlet 中
- 如果 servlet 还没有实例化，Servlet 容器负责实例化并初始化 servlet

常见 servlet 容器： Tomcat 、 Jetty

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-12-19-080628.jpg)

## Servlet 接口
``` Java
	public interface Servlet {
		void init(ServletConfig config) throws ServletException; 
		
		ServletConfig getServletConfig(); 
		
		void service(ServletRequest req, ServletResponse res）throws ServletException, IOException; 
					 
		String getServletInfo(); 
	 	void destroy();
	}
```

- **init(ServiceConfig) **方法: Servlet 容器在实例化 servlet 后，会且只调用一次 init 方法
- **getServletConfig(ServletRequest, ServletResponse)**: 返回一个包含 Servlet 初始化和启动参数的 ServletConfig 对象
	- ServiceRequest 封装请求信息
	- ServiceResponse 封装
- **service()**: 负责处理具体的业务逻辑。只会在 init() 方法执行过且执行成功的情况下才会被（Servlet 容器）调用。
- **getServletInfo()**: 返回 servlet 的一些信息，默认返回空字符串
- **destroy()**: 销毁方法，释放资源，也只会调用一次

### ServletConfig 

ServletConfig: A servlet configuration object used by a servlet container to pass information to a servlet during initialization.


web.xml 中 init-param 就会保存在 ServletConfig

``` xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>demo-servlet.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
```

ServiceConfig 接口定义：

```java
public interface ServletConfig {  
	public String getServletName();
	
	public ServletContext getServletContext();
	
	public String getInitParameter(String name);
	
	public Enumeration<String> getInitParameterNames();  
}
```

- **getSerlvetName()**: 获取 Servlet 名字，定义在 web.xml 的 \<servlet-name\>
- **getInitParameterNames()**: 获取 init-param 中所有 param-name 的集合（枚举）
- **getServletContext()**: 返回调用者当前所处的 servlet 上下文


### ServletContext

- ServletContext 代表应用本身
- ServletContext 的配置参数是所有 Serlvet 共用的
- ServletConfig 是 Servlet 级别
- ServletContext 是 Application 级别


```xml
<display-name>intiParam demo</display-name>
	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>application-context.xml</param-value>
    </context-param>
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>demo-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

```

以上配置获取 ServletContext 和 ServletConfig 下的配置地址：

``` java
String contextConfigLocation = getServletConfig().getServletContext().getInitParameter("contextConfigLocation");

String servletConfigLocation = getServletConfig().getInitParameter("contextConfigLocation");
```


### GenericServlet
GenericServlet 是 Servlet 的默认实现

- 实现了 ServletConfig 接口
	- GenericServlet 实现了 getInitParameter()，返回 getServletConfig().getInitParameter(name)，对于 GenericServlet 直接调用 getInitParameter() 即可。
- 提供了无参的 init() 方法
- 提供 了 log 方法
	- 默认调用 ServletContext 的 log 方法


#### init(ServletConfig) 的实现 

``` java
@Override  
public void init(ServletConfig config) throws ServletException {  
	this.config = config;  
	this.init();  
}
```

在对 SerivceConfig 赋值后，调用了 init() 无参的初始化方法。这里如果对 init() 重写，就不需要再调用 super.init(ServletConfig) 方法

### HttpServlet
Http 协议实现的 Servlet，Spring MVC 中的 DispatcherServlet 就是继承自 HttpServlet
主要重写了 service() 方法

#### service() 实现
service() 方法根据不同的请求方式，路由到了不同的处理方法，调用了不同的 doXXX 方法。

doGet、doPost、doPut、doDelete 方法为**模板方法**，子类必须实现，否则抛出异常。


### Context容器

Context 不直接持有 Servlet 的实例，而是通过子容器 Wrapper 进行管理 Servlet 。

**Wrapper** 可以看做是 Servlet 的包装，包含 Servlet 的实例、URL 映射、初始化参数等。

除了 Servlet 的创建和管理，Context 还需要创建 Listener 和 Filter 实例，并调用。

#### Servlet 管理

Wrapper 中会持有一个 Servlet 的引用：

``` java
protected volatile Servlet instance = null;
```

Wrapper 通过 laodServlet 来实例化 Servlet ：

``` java

public synchronized Servlet loadServlet() throws ServletException {
    Servlet servlet;
  
    //1. 创建一个Servlet实例
    servlet = (Servlet) instanceManager.newInstance(servletClass);    
    
    //2.调用了Servlet的init方法，这是Servlet规范要求的
    initServlet(servlet);
    
    return servlet;
}
```

loadServlet 方法：
- 创建 Servlet 实例
- 初始化 Servlet 

Tomcat 默认采用懒加载（除非 loadOnStrartup = true），启动时不会调用 loadServlet 加载 Servlet。

##### StandardWrapperValve
Wrapper 中的 BasicValve 就是 StandardWrapperValve， Context 容器中的 BasicValve 会调用 Wrapper 中的 BasicValve。

StandardWrapperValve 的 invoke 方法：

``` java

public final void invoke(Request request, Response response) {

    //1.实例化Servlet 
    servlet = wrapper.allocate();
   
    //2.给当前请求创建一个Filter链
    ApplicationFilterChain filterChain =
        ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);

   //3. 调用这个Filter链，Filter链中的最后一个Filter会调用Servlet
   filterChain.doFilter(request.getRequest(), response.getResponse());

}
```

- 实例化 Servlet，此处调用了 loadServlet() 方法
- 创建 Filter 链
- 调用 Filter 链

#### Filter 管理
Filter 的作用域是整个 Web 应用，所以 Filter 实例也是在 Context 中被管理。
Context 中使用 Map 保存 Filter ：

``` java
private Map<String, FilterDef> filterDefs = new HashMap<>();
```

FilterChain 是 多个 Filter 组成的 Filter 链，Filter 链的存活周期很短，每个请求都会动态的创建一个 FilterChain，请求返回后销毁。因为每个请求的请求路径都不一样，而 Filter 都有相应的路径映射，因此不是所有的 Filter 都需要来处理当前的请求，我们需要根据请求的路径来选择特定的一些 Filter 来处理。

``` java

public final class ApplicationFilterChain implements FilterChain {
  
  //Filter链中有Filter数组，这个好理解
  private ApplicationFilterConfig[] filters = new ApplicationFilterConfig[0];
    
  //Filter链中的当前的调用位置
  private int pos = 0;
    
  //总共有多少了Filter
  private int n = 0;

  //每个Filter链对应一个Servlet，也就是它要调用的Servlet
  private Servlet servlet = null;
  
  public void doFilter(ServletRequest req, ServletResponse res) {
        internalDoFilter(request,response);
  }
   
  private void internalDoFilter(ServletRequest req,
                                ServletResponse res){

    // 每个Filter链在内部维护了一个Filter数组
    if (pos < n) {
        ApplicationFilterConfig filterConfig = filters[pos++];
        Filter filter = filterConfig.getFilter();

        filter.doFilter(request, response, this);
        return;
    }

    servlet.service(request, response);
   
}
```

- FilterChain 中 保存了 Filter 数组，同时使用 pos 来标记当前执行位置。
- FilterChain 持有一个 Servlet 实例，因为需要调用 Servlet 的 service 方法。
- FilterChain 实现了 doFilter 方法，调用内部 internalDoFilter()
- internalDoFilter 方法中会判断 pos 是否小于 数组大小，小于代表该调用链未执行完，继续执行下一个。
- 最后调用 Servlet 的 service 方法。

Filter 中的 doFilter() 方法会调用 FilterChain 的 doFilter() 方法，完成循环调用。

#### Listener 管理
Listener 主要监听容器内的两类事件：
- 生命状态的变化：Context 容器启动和停止、Session 的创建和销毁。
- 属性的变化：Context 容器某个属性值变了、Session 的某个属性值变了以及新的请求来了等

Context 中讲负责两类的监听器分开存放：
``` java

//监听属性值变化的监听器
private List<Object> applicationEventListenersList = new CopyOnWriteArrayList<>();

//监听生命事件的监听器
private Object applicationLifecycleListenersObjects[] = new Object[0];
```

Context 在启动时，会触发所有 ServletContextListener：

``` java

//1.拿到所有的生命周期监听器
Object instances[] = getApplicationLifecycleListeners();

for (int i = 0; i < instances.length; i++) {
   //2. 判断Listener的类型是不是ServletContextListener
   if (!(instances[i] instanceof ServletContextListener))
      continue;

   //3.触发Listener的方法
   ServletContextListener lr = (ServletContextListener) instances[i];
   lr.contextInitialized(event);
}
```

