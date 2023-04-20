
### Tomcat 启动步骤

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-01-06-060423.jpg)

1. Tomcat 本质是一个 Java 程序。 start.sh 会运行 Bootstrap 运行类 （main 方法）。
2. Bootstrap 初始化 Tomcat 的类加载器，并 创建 Catalina
3. Catalina 解析 server.xml，创建相应的组件，并调用 Server 的 start 方法
4. Server 负责管理 Service 组件，调用 Service 的 start 方法
5. Service 负责管理 Contector 和 顶层容器 Engine，会调用 Conector 和 Engine 的 start 方法


### Catalina 

- Catalina 负责创建 Server ，通过解析 server.xml，把 server.xml 配置的组件一一创建出来，并且调用 Server 组件的 init 和 start 方法。
- Catalina 会在 JVM 中注册一个 CatalinaShutdownHook，在关闭进程时进行资源清理。

```java

public void start() {
    //1. 如果持有的Server实例为空，就解析server.xml创建出来
    if (getServer() == null) {
        load();
    }
    //2. 如果创建失败，报错退出
    if (getServer() == null) {
        log.fatal(sm.getString("catalina.noServer"));
        return;
    }

    //3.启动Server
    try {
        getServer().start();
    } catch (LifecycleException e) {
        return;
    }

    //创建并注册关闭钩子
    if (useShutdownHook) {
        if (shutdownHook == null) {
            shutdownHook = new CatalinaShutdownHook();
        }
        Runtime.getRuntime().addShutdownHook(shutdownHook);
    }

    //用await方法监听停止请求
    if (await) {
        await();
        stop();
    }
}
```

CatalinaShutdownHook 本质上就是一个线程， JVM 在停止之前会尝试调用这个线程的 run 方法。

```java

protected class CatalinaShutdownHook extends Thread {

    @Override
    public void run() {
        try {
            if (getServer() != null) {
                Catalina.this.stop();
            }
        } catch (Throwable ex) {
           ...
        }
    }
}
```


### Server 
StandardServer 是 Server 的实现类。

- Server 继承了 LifecycleBase，生命周期被统一管理
- Server 内部维护了 Service 组件，通过一个数组维护，addService 方法
- Server 需要管理 Service 的生命周期，在启动和停止是调用 Service 的启动和停止
- Server 会启动一个 Socket 来监听停止端口

addService 方法：

```java

@Override
public void addService(Service service) {

    service.setServer(this);

    synchronized (servicesLock) {
        //创建一个长度+1的新数组
        Service results[] = new Service[services.length + 1];
        
        //将老的数据复制过去
        System.arraycopy(services, 0, results, 0, services.length);
        results[services.length] = service;
        services = results;

        //启动Service组件
        if (getState().isAvailable()) {
            try {
                service.start();
            } catch (LifecycleException e) {
                // Ignore
            }
        }

        //触发监听事件
        support.firePropertyChange("service", null, service);
    }

}
```

### Service 

StandardService 实现了 Service 实现

```java

public class StandardService extends LifecycleBase implements Service {
    //名字
    private String name = null;
    
    //Server实例
    private Server server = null;

    //连接器数组
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();

    //对应的Engine容器
    private Engine engine = null;
    
    //映射器及其监听器
    protected final Mapper mapper = new Mapper();
    protected final MapperListener mapperListener = new MapperListener(this);
```

MapperListener 为了支持热部署， Web 应用部署发生变化是， Mapper 映射信息变化，MapperListener 监听容器变化。

``` java

protected void startInternal() throws LifecycleException {

    //1. 触发启动监听器
    setState(LifecycleState.STARTING);

    //2. 先启动Engine，Engine会启动它子容器
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }
    
    //3. 再启动Mapper监听器
    mapperListener.start();

    //4.最后启动连接器，连接器会启动它子组件，比如Endpoint
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```

Service 会先启动内层容器，在启动外层组件：
 
1. 先启动 Engine 组件
2. 再启动 Mapper 监听器
3. 启动 Connector


### Engine

Engine 是最顶层的容器，继承了 ContainerBase，并实现了 Engine 接口

- ContainerBase 中 有一个 HashMap<String, Container>  children 数组，用来维护 Engine 的下一级容器 Host
- ContainerBase 实现了对子容器的 增删查改
- ContainerBase 提供了子容器的启动、停止的默认实现
- Engine 通过 Valve 根据 request 中的 Host，调用对应的 Host。
- Request 在 Mapper 组件中，Mapper 定位到了对应的 Host，保存在了 Request 中

``` java

final class StandardEngineValve extends ValveBase {

    public final void invoke(Request request, Response response)
      throws IOException, ServletException {
  
      //拿到请求中的Host容器
      Host host = request.getHost();
      if (host == null) {
          return;
      }
  
      // 调用Host容器中的Pipeline中的第一个Valve
      host.getPipeline().getFirst().invoke(request, response);
  }
  
}
```