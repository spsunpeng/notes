# 2021.08

​    由 java.lang （java.lang 是java语言的核心，它提供了java中的基础类。包括基本Object类、Class类、String类、基本类型的包装类、基本的数学类等等最基本的类） 提供。

​    在 Java 中实现多线程有两种手段，一种是继承 Thread 类，另一种就是实现 Runnable 接口。下面我们就分别来介绍这两种方式的使用。

Thread

```java
new Thread(){
    @Override
    public void run() {
        System.out.println("Thread 1");
    }
}.start();
```

Runnable

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Runnable 1");
    }
};
Thread thread = new Thread(runnable);
thread.start();
```



Thread 和 Runnable的异同

```java
public class Thread extends Object implements Runnable{
    
    Private Runnable target； 
     public Thread(Runnable target,String name){ 
        init(null,target,name,0); 
     } 
     private void init(ThreadGroup g,Runnable target,String name,long stackSize){ 
         ... 
         this.target=target; 
     } 
     public void run(){ 
         if(target!=null){ 
             target.run(); 
         } 
 	}
    
}
```

- Thread 实现 Runnable

-  run() 方法 需要重写

- 如果一个类继承 Thread类，则不适合于多个线程共享资源，而实现了 Runnable 接口，就可以方便的实现资源的共享。



# 2021.09

偏向锁

自旋锁（执行时间短，等待线程少）

系统锁（重量锁）



synchronized 锁的是对象不是代码

this、Class

基本类型用不了，也别用String、Interger、Long，可能锁的是第三方类库

锁粒度变细（锁的力度大导致速度编码，同时出现问题不好定位）

锁粒度变粗（方法中有太多的细粒度锁，锁的创建销毁消耗太多资源）





volatile 

保证线程一致性

禁止指令重排序



# 2021.12

### 1、线程池

#### 1.1 创建

```java
public ThreadPoolExecutor(
int corePoolSize,                    //核心池大小                         
int maximumPoolSize,                 //池中允许的最大线程数                            
long keepAliveTime,                  //当线程数大于核心时，多于的空闲线程最多存活时间
TimeUnit unit,                       //keepAliveTime单位                       
BlockingQueue<Runnable> workQueue,   //当线程数目超过核心线程数时用于保存任务的队列
ThreadFactory threadFactory,         //执行程序创建新线程时使用的工厂
RejectedExecutionHandler handler) {} //阻塞队列已满且线程数达到最大值时所采取的饱和策略
```

- size 是正常情况下线程池维护的线程大小
- 当任务大于线程的执行速度后，多余的线程会保存在缓存队列 workQueque
- 当缓存队列满后，会开辟新的线程，最大线程数 maxSize
- 新开辟的线程处理完任务后，没有新任务，经过 keepAliveTime/timeUnit 时间后，会销毁。
- 当最大线程也用完后，多余出的任务会执行放弃策略 policy

##### 1.1.1 workQueque

- LinkedBlockingQueue
- ArrayBlockingQueue
- SynchronousQueue

##### 1.1.2 policy

- AbortPolicy中止策略
- DiscardPolicy抛弃策略
- DiscardOldestPolicy抛弃旧任务策略
- CallerRunsPolicy调用者运行

#### 1.2 示例

```java
//创建
ExecutorService executorService = new ThreadPoolExecutor(3,5,10，TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(1000),
            new ThreadPoolExecutor.DiscardPolicy());
//执行
executorService.submit(()-{});
```











run: Thread[hystrix-ext-1,5,main]
getFallback: Thread[hystrix-ext-1,5,main]
result:getFallback
main: Thread[main,5,main]











