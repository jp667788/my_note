### [[JVM 类加载]]器
 - Java 的类加载：把字节码格式的 .class 文件加载到方法区，并在堆中创建一个 java.lang.class 的对象。
 - Class 对象：理解成是类的模板，JVM 根据 class 创建实例对象


#### ClassLoader
JDK 提供了一个抽象类 ClassLoader ，来负责类的加载。

``` java
	
public abstract class ClassLoader {

    //每个类加载器都有个父加载器
    private final ClassLoader parent;
    
    public Class<?> loadClass(String name) {
  
        //查找一下这个类是不是已经加载过了
        Class<?> c = findLoadedClass(name);
        
        //如果没有加载过
        if( c == null ){
          //先委托给父加载器去加载，注意这是个递归调用
          if (parent != null) {
              c = parent.loadClass(name);
          }else {
              // 如果父加载器为空，查找Bootstrap加载器是不是加载过了
              c = findBootstrapClassOrNull(name);
          }
        }
        // 如果父加载器没加载成功，调用自己的findClass去加载
        if (c == null) {
            c = findClass(name);
        }
        
        return c；
    }
    
    protected Class<?> findClass(String name){
       //1. 根据传入的类名name，到在特定目录下去寻找类文件，把.class文件读入内存
          ...
          
       //2. 调用defineClass将字节数组转成Class对象
       return defineClass(buf, off, len)；
    }
    
    // 将字节码数组解析成一个Class对象，用native方法实现
    protected final Class<?> defineClass(byte[] b, int off, int len){
       ...
    }
}
```

- JVM 的类加载去具有层级，有父子关系。通过 parent  指向父加载器
- defineClass 负责调用 native 方法，将 Java 类的字节码解析成 Class 对象
- findClass 负责找到 .class 文件，可能来自于系统，也可能来自于网络
- loadClass 是 public 的方法，提供给外界的方法。
	- 首先检查这个类是否已被加载，已加载直接返回
	- 否则交给父类加载器加载


#### 双亲委派
当一个类加载器需要加载一个 Java 类时，会先委托父加载器去加载，然后父加载器在自己的加载路径中搜索 Java 类，当父加载器在自己的加载范围内找不到时，才会交还给子加载器加载，这就是**双亲委派**机制。
 

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-12-073511.jpg)


- **BootstrapClassLoader** 是启动类加载器，由 C 语言实现，用来加载 JVM 启动时所需要的核心类，比如rt.jar、resources.jar等。
- **ExtClassLoader** 是扩展类加载器，用来加载\jre\lib\ext目录下 JAR 包
- **AppClassLoader** 是系统类加载器，用来加载 classpath 下的类，应用程序默认用它来加载类。
- **自定义类加载器**，用来加载自定义路径下的类。

> 类加载器的父子关系不是通过继承来实现的，比如 AppClassLoader 并不是 ExtClassLoader 的子类，而是说 AppClassLoader 的 parent 成员变量指向 ExtClassLoader 对象

### WebAppClassLoader

Tomcat 自定义了 **WebAppClassLoader， 并且打破了双亲委派**。通过重写 ClassLoader 的 loadClass() 和 findClass() 方法，会先自己尝试加载类，加载失败才会交给父类进行加载。

#### findClass 方法

``` java

public Class<?> findClass(String name) throws ClassNotFoundException {
    ...
    
    Class<?> clazz = null;
    try {
            //1. 先在Web应用目录下查找类 
            clazz = findClassInternal(name);
    }  catch (RuntimeException e) {
           throw e;
       }
    
    if (clazz == null) {
    try {
            //2. 如果在本地目录没有找到，交给父加载器去查找
            clazz = super.findClass(name);
    }  catch (RuntimeException e) {
           throw e;
       }
    
    //3. 如果父类也没找到，抛出ClassNotFoundException
    if (clazz == null) {
        throw new ClassNotFoundException(name);
     }

    return clazz;
}
```

WebAppClassLoader 加载步骤：
- 先在 Web应用目录下查找类
- 如果没有找到，交给父类查找，即 AppClassLoader
- 如果父类也没找到，则抛出 ClassLoaderNotFountException

#### loadClass 方法

``` java

public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {

    synchronized (getClassLoadingLock(name)) {
 
        Class<?> clazz = null;

        //1. 先在本地cache查找该类是否已经加载过
        clazz = findLoadedClass0(name);
        if (clazz != null) {
            if (resolve)
                resolveClass(clazz);
            return clazz;
        }

        //2. 从系统类加载器的cache中查找是否加载过
        clazz = findLoadedClass(name);
        if (clazz != null) {
            if (resolve)
                resolveClass(clazz);
            return clazz;
        }

        // 3. 尝试用ExtClassLoader类加载器类加载，为什么？
        ClassLoader javaseLoader = getJavaseClassLoader();
        try {
            clazz = javaseLoader.loadClass(name);
            if (clazz != null) {
                if (resolve)
                    resolveClass(clazz);
                return clazz;
            }
        } catch (ClassNotFoundException e) {
            // Ignore
        }

        // 4. 尝试在本地目录搜索class并加载
        try {
            clazz = findClass(name);
            if (clazz != null) {
                if (resolve)
                    resolveClass(clazz);
                return clazz;
            }
        } catch (ClassNotFoundException e) {
            // Ignore
        }

        // 5. 尝试用系统类加载器(也就是AppClassLoader)来加载
            try {
                clazz = Class.forName(name, false, parent);
                if (clazz != null) {
                    if (resolve)
                        resolveClass(clazz);
                    return clazz;
                }
            } catch (ClassNotFoundException e) {
                // Ignore
            }
       }
    
    //6. 上述过程都加载失败，抛出异常
    throw new ClassNotFoundException(name);
}
```
 
WebAppClassLoder loadClass步骤：
- 在本地 cache 中查找是否已加载过
- 在系统加载器的 cache 中查找是否已加载过
- **先让 ExtClassLoader 加载**。这么做目的是为了防止 Web 应用中的类覆盖了 JRE  核心类库，这是由于 WebAppClassLoader 打破了双亲委派，如果优先加载 Web 应用中的类就会可能覆盖掉 JRE 的类。
- ExtClassLoader 加载失败，说明 JRE 中没有这个类，就在 Web 应用下的目录查找并加载。
- 再交给系统加载器加载，通过 Class.forName() 交给系统加载器加载。
- 全部加载失败，抛出 ClassNotFoundException


### Tomcat 的类加载器层级
Tomcat 类加载器设计需要处理的问题：
1. 假如我们在 Tomcat 中运行了两个 Web 应用程序，两个 Web 应用中有同名的 Servlet，但是功能不同，Tomcat 需要同时加载和管理这两个同名的 Servlet 类，保证它们不会冲突，因此 Web 应用之间的类需要隔离。
2. 假如两个 Web 应用都依赖同一个第三方的 JAR 包，比如 Spring，那 Spring 的 JAR 包被加载到内存后，Tomcat 要保证这两个 Web 应用能够共享，也就是说 Spring 的 JAR 包只被加载一次，否则随着依赖的第三方 JAR 包增多，JVM 的内存会膨胀。
3. 跟 JVM 一样，我们需要隔离 Tomcat 本身的类和 Web 应用的类。


![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-02-12-083257.jpg)

#### [[#WebAppClassLoader]] 
加载目录：\ <Tomcat \>/webapps/\<app\>/WEB-INF/*
Tomcat 会给每个 Web 应用创建一个 WebAppClassLoader 实例，每个应用使用不同的 WebAppClassLoader 进行加载。Context 中会维护一个 WebAppClassLoader 的实例，**不同加载器加载的类会被认为是不同的类**。

#### SharedClassLoader
加载目录：\<Tomcat\>/shared/*
SharedClassLoader 作为 WebAppClassLoader 的父加载器，负责加载 Web 应用间共用的类，  

#### CatalinaClassLoader
加载目录：\<Tomcat\>/server/*
父子关系实现共享，兄弟关系实现隔离。CatalinaClassLoader 和 ShareClassLoader 平行，可以实现 Tomcat 自身的类和 Web 应用的类隔离。

#### CommonClassLoader
加载目录：\<Tomcat\>/common/*
Tomcat 和 Web 应用间需要共享的类 通过 CommonClassLoader 加载，CommonClassLoader 是 CatalinaClassLoader 和  SharedClassLoader 的父加载器。

### 线程上下文加载器
JVM 默认类加载器在加载类A的时候，A 类依赖的类也是由该类加载器进行加载。
Spring 作为 Web 间共享的 jar 包，由 SharedClassLoader 进行加载。Spring 中又会进行业务类的加载，此时会取出线程中的线程上下文加载器，进行业务类的加载。

Tomcat 为每个 Web 应用创建一个 WebAppClassLoader 类加载器，并在启动 Web 应用的线程里设置线程上下文加载器。Spring 在加载业务类时可以通过：

``` java
cl = Thread.currentThread().getContextClassLoader();
```

获取线程上下文加载器进行业务类的加载。