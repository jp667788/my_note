## Tomcat 一键启停
一个请求在 Tomvat 中的流转过程：

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-01-04-143533.jpg)

Tomcat 负责创建、组装、启动这些组件，在服务停止时，需要释放资源、销毁组件，所以 Tomcat 需要动态地管理这些组件的生命周期。

设计思想：
- 组件有大有小，大组件管理小组件：Server 管理 Service、Service 管理 Connector 和 Container
- 组件有外由内，外层组件控制内层。外层组件调用内层组件完成业务功能
- 先创建子组件，再创建父组件，子组件被注入都父组件中
- 先创建内层组件，再创建外层组件，内层组件被注入到外层组件中


### Lifecycle 接口

#### 一键启停
生命周期需要的方法：
- init()
- start()
- stop()
- destroy()

父组件的 init 方法创建子组件，并调用子组件的 init 方法。父组件的 start 方法调用子组件的 start 方法。

#### 可扩展性

Lifecycle 中 addLifecycleListener(LifecycleListener listener) 维护一个监听器。容器的生命周期可以看做是状态的流转，可以看做是一个事件。监听器监听事件，这就是[[观察者模式]]
容器的状态由 LifecycleState 表示，有 NOT_CONNECTED, REGISTERED, CONNECTED, ACTIVATING, ACTIVE, DISCONNECTED, DEACTIVATING, DEACTIVATED, CLOSED

监听器加入时机：
- Tomcat 自定义了一些监听器，这些监听器是父组件在创建子组件的过程中注册到子组件的
- server.xml中定义自己的监听器，Tomcat 在启动时会解析server.xml，创建监听器并注册到容器组件

#### 重用性 

LifecycleBase 实现了 Lifecycle 接口，放一些公共的逻辑作为基类。
- 生命周期状态的转变与维护
- 生命时间的触发
- 监听器的添加和删除

子类的方法会加上 Internal 避免与基类重名，如：initInternal，startInternal 

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-01-05-125915.jpg)

LifecycleBase 实现了Lifecycle 接口中的所有方法，并且定义了相应的抽象方法子类实现，这是典型的[[模板方法模式]]。


### 生命周期总体类图

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-01-05-130343.jpg)
