## Tomcat 总体架构

两个**核心功能**：

- 处理 Socket 连接，负责转化网络字节流与 Request 和 Response 之间的转化
- 加载和管理 Servlet ，处理具体的 Request 对象

分别对应的**两个核心组件**：

- Connector （[[#连接器]]）：负责对外交流
- Container （[[#容器]]）：负责内部处理

**Tomcat 支持的 I/O 模型：**

- NIO：非阻塞 I/O，采用 Java NIO 类库实现。
- NIO.2：异步 I/O，采用 JDK 7 最新的 NIO.2 类库实现。
- APR：采用 Apache 可移运行库实现，是 C/C++ 编写的本地库。


**Tomcat 支持的应用层协议:**

- HTTP/1.1：这是大部分 Web 应用采用的访问协议。
- AJP：用于和 Web 服务器集成（如 Apache）。
- HTTP/2：HTTP 2.0 大幅度的提升了 Web 性能。


为了实现支持多种协议，所以**一个 Container** 对应了**多个 Connector** ，Container 和 Connector 组成 了 Service。Service 没有做太多事情，只是给 Container 和 Connector 包了一层。Tomcat 内也可能存在**多个 Service** ，可以实现一个 Tomcat 通过不同的端口号部署多个 Service。


![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-12-29-130431.jpg)

Server 指的就是一个 Tomcat 实例，连接器和容器之间使用 ServletRequest 和 ServletResponse 进行通信


### 连接器
连接器对 Servlet 屏蔽了 不同协议、不同 I/O 模型的区别，Servlet 收到的都是标准的 ServletRequest 对象。

连接器的功能需求：

- 监听网络端口。接受网络连接请求。
- 读取网络请求字节流。
- 根据具体应用层协议（HTTP/AJP）解析字节流，生成统一的 Tomcat Request 对象。
- 将 Tomcat Request 对象转成标准的 ServletRequest。
- 调用 Servlet 容器，得到 ServletResponse。
- 将 ServletResponse 转成 Tomcat Response 对象。
- 将 Tomcat Response 转成网络字节流。
- 将响应字节流写回给浏览器。

总共分为 3 类需求：
- 网络通信
- 通信协议解析
- Tomcat Request 和 Response 对象 转化为 ServletRequest 和 ServletResponse

分别对应了3个组件：
- Endpoint 
- Processor
- Adpater

Endpoint  提供字节流 -> Processor 提供 Tomcat Request -> Adpater 转化成 ServletRequest 

由于 **I/O 模型 和 应用层协议可以自由组合**，设计了 **ProcessorHandler** 接口、抽象基类 **AbstractProtocol**。每种应用协议都有自己的抽象基类：AbstractAjpProtocol 、 AbstractHttp11Protocol，具体的协议实现抽象基类进行拓展

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-12-30-124159.jpg)


连接器架构：

 - ProcessorHandler
	 - EndPoint
	 - Processor
 - Adapter
 
 
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-12-30-124326.png)



#### ProcessorHandler 

- EndPoint
	- AbstractEndpoint
		- NioEndpoint
		- Nio2Endpoint
		- Acceptor
		- SocketProcessor

- **EndPoint** : 通信端点，通信监听的接口，是 Socket 具体的接收和发送处理器。**是- 对传输层的抽象，用来实现 TCP/IP 协议**

- **Acceptor** : Socket 接受器

- **SocketProcessor** : Socket 处理器，实现了 Runnable 接口，内部调用协议对应的 
Processor 。**被提交到线程池中执行**

- Processor
	- AbstractProcessor
		- AjpProcessor
		- Http11Processor


![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-12-30-131609.png)

1. EndPoint 接收 Socket 
2. 通过 Acceptor 生成 SocketProcessor 对象，交由线程池执行
3. SocketProcessor 调用 Processor 组件解析协议
4. Processor 解析后，生成 Request 对象给 Adapter


#### Adapter
通过调用 CoyoteAdapter 的 service 方法，传入 Tomcat Request 对象，转换成 ServletRequest 对象，作为标准输入调用 Servlet 容器


### 容器
4种容器：
- Engine
	- Host
		- Context
			- Wrapper

Wrapper : 代表一个 Servlet
Context ： 代表一个应用程序
Host : 代表一个虚拟主机
Engine : 用来管理多个虚拟机站点，一个Service 最多只能有一个 Engine

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-12-30-133052.jpg)

Tomcat 使用[[组合模式]] 在管理这些具有**父子级关系的**容器，形成一个**树形结构**。
- 所有容器组件实现 Container 接口，使得容器对象（Wrapper）和组合容器对象（Context 、Host）使用起来**具有一致性**

``` java

public interface Container extends Lifecycle {
    public void setName(String name);
    public Container getParent();
    public void setParent(Container container);
    public void addChild(Container child);
    public void removeChild(Container child);
    public Container findChild(String name);
}
```

### 定位 Servlet 过程
Tomcat 使用 Mapper 组件完成定位 Servlet

- Mapper 组件保存了 Web 应用的配置信息：**容器组件与访问路径的映射关系**
- Mapper 组件解析 URL 里的域名和路径，再到自己保存的 Map 里查找

1. 根据协议和端口号选定 Service 和 Engine
	1. 默认 HTTP 8080
	2. 默认 AJP 8009
2. 根据域名选定 Host
3. 根据 URL 找到 Context
4. 根据URL 找到 Wrapper

#### 调用实现
Tomcat 使用 Pipeline-Valve 实现容器的逐级调用，Pipeline-Valve 采用[[责任链模式]]实现。

Valve 标识一个处理点
``` java
	public interface Valve {
	  public Valve getNext();
	  public void setNext(Valve valve);
	  public void invoke(Request request, Response response)
	}
```

- invoke() 用来处理请求
- getNext()、setNext() 形成链状结构

``` java
	public interface Pipeline extends Contained {
	  public void addValve(Valve valve);
	  public Valve getBasic();
	  public void setBasic(Valve valve);
	  public Valve getFirst();
	}
```

- Pipeline 维护了 Valve 链表
- Pipeline 通过调用 Valve 的 invoke 方法，处理请求
- Valve 中调用 getNext.invoke 触发下一个 Valve 的 调用
- 每一个容器都有一个 Pipeline 对象
- 容器通过 getBasic 获取 一个 BasicValve ，这个 BasicValve 处于 Valve 链末端，**负责调用下一层容器的 Pipeline 的第一个 Valve**
- 整个调用过程由 Connector 中的 Adapter 触发，它会调用 Engine 的 第一个 Valve
- Wrapper 容器的最后一个 Valve 会创建一个 Filter 链，并调用 doFilter 方法，最终调用到 Servlet 的 Service 方法


```java

// Calling the container
connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);
```

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-01-04-142456.png)


#### Valve 与 Filter 区别
- Valve 是 Tomcat 私有机制，与 Tomcat 架构紧耦合。而 Servlet API 是公有标准，所有 Servlet 容器均支持 Filter
- Valve 是容器级别，拦截所有应用请求。Filter 是应用级别，只能拦截某个应用的所有请求。