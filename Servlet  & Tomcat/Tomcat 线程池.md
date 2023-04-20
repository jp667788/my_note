### Java [[线程池]]
- ThreadPoolExecutor
	- FixedThreadPool
		``` java
		public static ExecutorService newFixedThreadPool(int nThreads) { 
			return new ThreadPoolExecutor(nThreads, nThreads, 0L, 				TimeUnit.MILLISECONDS, new 	LinkedBlockingQueue());
		}
		```
		- corePoolSize = maximumPoolSize LinkedBlockingQueue 无线队列。达到核心线程数之后就放到无线队列中

	- CachedThreadPool
		``` java
		public static ExecutorService newCachedThreadPool() { 
			return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue());
		}
		```
		
		- 永远是新的线程 


FixedThreadPool/CachedThreadPool 可以理解为定制的 ThreadPoolExecutor

### Tomcat 线程池 

ThreadPoolExecutor 两个关键点：
- 是否限制线程个数
- 是否限制队列长度

Tomcat 需要对上述两个都要限制
``` java

//定制版的任务队列
taskqueue = new TaskQueue(maxQueueSize);

//定制版的线程工厂
TaskThreadFactory tf = new TaskThreadFactory(namePrefix,daemon,getThreadPriority());

//定制版的线程池
executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), maxIdleTime, TimeUnit.MILLISECONDS,taskqueue, tf);

```

- Tomcat 有自己的定制任务队列，最大为 maxQueueSize
- Tomcat 设置了最大线程数和核心线程数

#### Tomcat 线程池和 Java 线程池quiet
Java 线程池执行逻辑：
1. 前 corePoolSize 个任务，创建新的线程
2. 把任务放到队列
3. 队列满了创建线程到最大线程数，之后执行拒绝策略

Tomcat 线程池执行步骤
1. 前 corePoolSize 个任务，创建新的线程
2. 把任务放到队列
3. 队列满了创建线程到最大线程数，**再次尝试放入队列**
4. **队列满了，执行拒绝策略**

``` java

public class ThreadPoolExecutor extends java.util.concurrent.ThreadPoolExecutor {
  
  ...
  
  public void execute(Runnable command, long timeout, TimeUnit unit) {
      submittedCount.incrementAndGet();
      try {
          //调用Java原生线程池的execute去执行任务
          super.execute(command);
      } catch (RejectedExecutionException rx) {
         //如果总线程数达到maximumPoolSize，Java原生线程池执行拒绝策略
          if (super.getQueue() instanceof TaskQueue) {
              final TaskQueue queue = (TaskQueue)super.getQueue();
              try {
                  //继续尝试把任务放到任务队列中去
                  if (!queue.force(command, timeout, unit)) {
                      submittedCount.decrementAndGet();
                      //如果缓冲队列也满了，插入失败，执行拒绝策略。
                      throw new RejectedExecutionException("...");
                  }
              } 
          }
      }
}
```

submittedCount：维护已经提交到了线程池，但是还没有执行完的任务个数

#### Tomcat 定制版的任务队列 TaskQueue

- TaskQueue 继承 LinkBlockingQueue
- LinkBlockingQueue 默认情况下没有长度限制
- Tomcat 的参数 maxQueueSize 限制了 TaskQueue 的 capacity，但是默认情况下是 Integer.MAX_VALUE，相当于无线队列，不会再创建大于 corePoolSize 的新线程

Tomcat 的 TaskQueue 重写了 LinkBlockingQueue 中的 offer 方法， 并通过submittedCount 来控制创建新线程的时机 ，可以让线程池有机会继续创建线程

``` java
	
public class TaskQueue extends LinkedBlockingQueue<Runnable> {

  ...
   @Override
  //线程池调用任务队列的方法时，当前线程数肯定已经大于核心线程数了
  public boolean offer(Runnable o) {

      //如果线程数已经到了最大值，不能创建新线程了，只能把任务添加到任务队列。
      if (parent.getPoolSize() == parent.getMaximumPoolSize()) 
          return super.offer(o);
          
      //执行到这里，表明当前线程数大于核心线程数，并且小于最大线程数。
      //表明是可以创建新线程的，那到底要不要创建呢？分两种情况：
      
      //1. 如果已提交的任务数小于当前线程数，表示还有空闲线程，无需创建新线程
      if (parent.getSubmittedCount()<=(parent.getPoolSize())) 
          return super.offer(o);
          
      //2. 如果已提交的任务数大于当前线程数，线程不够用了，返回false去创建新线程
      if (parent.getPoolSize()<parent.getMaximumPoolSize()) 
          return false;
          
      //默认情况下总是把任务添加到任务队列
      return super.offer(o);
  }
  
}
```