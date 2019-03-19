# 【Android】关于线程和线程池

## 前言

鉴于自己对线程的了解还停留在使用Thread、Runable、AsyncTask创建线程使用这些简单的理解，为了巩固补充下基础，所以在此上网查询搜索总结下相关信息，补充自己的知识。

## 线程是什么

线程( Thread )是操作系统进行运算调度的最小单位。线程的运行包含在进程中，是进程中的实际运行单位。一个进程中可以并发多个线程，每条线程并行执行不同的任务。同一进程中的多条线程将共享该进程中的全部系统资源，如虚拟地址空间，文件描述符和信号处理等等。但同一进程中的多个线程有各自的调用栈（call stack），自己的寄存器环境（register context），自己的线程本地存储（thread-local storage）。一个进程可以有很多线程，每条线程并行执行不同的任务。

## Android如何开启一个线程

1. **通过继承Thread类，调用start方法开启新线程**

   ```kotlin
   class MyThread : Thread(){
   
       override fun run() {
           super.run()
           println("我是新的线程运行的")
       }
       
   }
   
   fun main(){
       MyThread().start()
   }
   ```

2. **通过实现Runnable接口开启一个新的线程**

   ```kotlin
   class MyRunnable : Runnable{
   
       override fun run() {
           print("我也是新的线程")
       }
   
   }
   
   fun main(){
       Thread(MyRunnable()).start()
   }
   ```

3. **实现Callback接口**

   ```kotlin
   class MyCallable : Callable<Int> {
   
       override fun call() : Int {
           println("我也能开启新的线程")
           return 0
       }
   
   }
   
   fun main(){
   
       val callable = MyCallable()
       val futureTask = FutureTask<Int>(callable)
       Thread(futureTask).start()
   
   
       val threadResult = futureTask.get()
       println(threadResult ?: "empty")
   }
   
   // Callable接口支持返回执行结果，此时需要调用FutureTask.get()方法实现，此方法会阻塞主线程直到获取‘将来’结果；当不调用此方法时，主线程不会阻塞
   
   //运行结果如下
   //
   //我也能开启新的线程
   //0   
   ```

   Runnable和Callable通过实现接口实现多线程，不仅能继承其他类，而且能多个线程共享一个target，很适合资源共享。只不过编程的过程稍稍复杂。如果想获取当前线程，可以使用`Thread.currentThread()`方法

4. **使用AsyncTask创建新的线程**

   Android中轻量级的异步类，不过不同版本的Android实现机制不一样，使用时需要注意

   ```kotlin
   class MyTask : AsyncTask<Unit,Unit,Unit>(){
       override fun doInBackground(vararg params: Unit?) {
           println("AsyncTask执行了")
       }
   }
   
   fun main(){
       MyTask().execute()
   }
   ```

5. **IntentService**

   继承自Service，拥有Service的全部特性，同时又能在新开的线程中执行。它可以用于在后台执行耗时的异步任务，当任务完成后会自动停止，它内部通过HandlerThread和Handler实现异步操作，创建IntentService时，只需实现onHandleIntent和构造方法，onHandleIntent为异步方法，可以执行耗时操作。

   ```kotlin
   class MyService : IntentService(""){
       override fun onHandleIntent(intent: Intent?) {
           // 这里进行处理
       }
   }
   
   val intent = Intent(this, MyService::class.java)
   intent.putExtra("键","值")
   startService(intent)
   ```

6. **通过HandlerThread创建新的线程**

   个人理解是HandlerThread是一个子线程，并且内部初始化了Looper，以方便我们进行相关操作。

   ```kotlin
       val handlerThread = HandlerThread("HandlerThread")
       handlerThread.start()
       val handler = object : Handler(handlerThread.looper){
           override fun handleMessage(msg: Message?) {
               super.handleMessage(msg)
               println("新的事件来了")
           }
       }
       handler.sendEmptyMessage(0)
       // 退出
       handler.removeMessages(0)
       handlerThread.quitSafely()
   ```

## 关于线程池

如果我们需要做很多的事情，而因此每做一件事情就开启一个新的线程的话，频繁的创建和销毁线程，对CPU性能的消耗也不容小觑。这时，我们就可以使用线程池进行对时间的处理。当我们有时间需要处理的时候，线程池会调度核心线程进行工作，核心线程有一定的数量，默认一旦创建了就会保留持续运行。如果并发时间过多，核心线程都处理不过来的话，将会进行非核心线程的创建，进行处理更多事件。与核心线程不同的是，非核心线程一旦工作完成后，将会进行等待，默认在指定的超时时间内都没有事件需要处理的话，则进行销毁。如果时间多到连非核心线程池都处理不过来的话，将会把时间存放到消息队列中进行排队等待。总的来说，使用线程池能

+ 重用线程池中的线程，避免频繁地创建和销毁线程带来的性能消耗
+ 有效控制线程的最大并发数量，防止线程过大导致抢占资源造成系统阻塞
+ 可以对线程进行一定地管理

## 线程池的创建

ExecutorService是最初的线程池接口，ThreadPoolExecutor类是对线程池的具体实现，它通过构造方法来配置线程池的参数，其构造函数如下

```kotlin
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
 this(corePoolSize,maximumPoolSize,keepAliveTime,unit,workQueue,Executors.defaultThreadFactory(), defaultHandler);
}
```

**corePoolSize** : 线程池中核心线程的数量，默认情况下，即使核心线程没有任务在执行它也存在的，我们固定一定数量的核心线程且它一直存活这样就避免了一般情况下CPU创建和销毁线程带来的开销。我们如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程就会有超时策略，这个时间由keepAliveTime来设定，即keepAliveTime时间内如果核心线程没有回应则该线程就会被终止。allowCoreThreadTimeOut默认为false，核心线程没有超时时间。

**maximumPoolSize** : 线程池中的最大线程数，当任务数量超过最大线程数时其它任务可能就会被阻塞。最大线程数=核心线程+非核心线程。非核心线程只有当核心线程不够用且线程池有空余时才会被创建，执行完任务后非核心线程会被销毁。

**keepAliveTime** : 非核心线程的超时时长，当执行时间超过这个时间时，非核心线程就会被回收。当allowCoreThreadTimeOut设置为true时，此属性也作用在核心线程上。

**unit** : 枚举时间单位，TimeUnit。

**workQueue** : 线程池中的任务队列，我们提交给线程池的runnable会被存储在这个对象上

## 线程池的分配规则

+ 当线程池中的核心线程数量未达到最大线程数时，启动一个核心线程去执行任务；
+ 如果线程池中的核心线程数量达到最大线程数时，那么任务会被插入到任务队列中排队等待执行；
+ 如果在上一步骤中任务队列已满但是线程池中线程数量未达到限定线程总数，那么启动一个非核心线程来处理任务；
+ 如果上一步骤中线程数量达到了限定线程总量，那么线程池则拒绝执行该任务，且ThreadPoolExecutor会调用RejectedtionHandler的rejectedExecution方法来通知调用者。

## 线程池的分类

1. **FixedThreadPool**

   通过Executors的newFixedThreadPool()方法创建，它是个线程数量固定的线程池，该线程池的线程全部为核心线程，它们没有超时机制且排队任务队列无限制，因为全都是核心线程，所以响应较快，且不用担心线程会被回收。其创建方法如下

   ```java
       public static ExecutorService newFixedThreadPool(int nThreads) {
           return new ThreadPoolExecutor(nThreads, nThreads,
                                         0L, TimeUnit.MILLISECONDS,
                                         new LinkedBlockingQueue<Runnable>());
       }
   ```

   要创建这样的线程池，我们可以这样

   ```kotlin
   Executors.newFixedThreadPool(30)
   ```

   

2. **CachedThreadPool**

   通过Executors的newCachedThreadPool()方法来创建，它是一个数量无限多的线程池，它所有的线程都是非核心线程，当有新任务来时如果没有空闲的线程则直接创建新的线程不会去排队而直接执行，并且超时时间都是60s，所以此线程池适合执行大量耗时小的任务。由于设置了超时时间为60s，所以当线程空闲一定时间时就会被系统回收，所以理论上该线程池不会有占用系统资源的无用线程。其创建方法如下

   ```java
       public static ExecutorService newCachedThreadPool() {
           return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                         60L, TimeUnit.SECONDS,
                                         new SynchronousQueue<Runnable>());
       }
   ```

   要创建这样的线程池，我们可以这样

   ```kotlin
   Executors.newCachedThreadPool()
   ```

3. **ScheduledThreadPool**

  通过Executors的newScheduledThreadPool()方法来创建，ScheduledThreadPool线程池像是上两种的合体，它有数量固定的核心线程，且有数量无限多的非核心线程，但是它的非核心线程超时时间是0s，所以非核心线程一旦空闲立马就会被回收。这类线程池适合用于执行定时任务和固定周期的重复任务。

  ```java
      public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
          return new ScheduledThreadPoolExecutor(corePoolSize);
      }
      
      public ScheduledThreadPoolExecutor(int corePoolSize) {
          super(corePoolSize, Integer.MAX_VALUE,
                DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                new DelayedWorkQueue());
      }
  ```

  添加任务时，可以调用schecule()方法和scheduleAtFixedRate()方法。

  **command** : 要执行的事情

  **delay** : 第一次执行的延迟时间

  **unit** : 时间的单位枚举，TimeUnit

  **period** : 两个事件之间的执行间隔

  要创建这样的线程池，我们可以

  ```kotlin
  Executors.newScheduledThreadPool(10)
  ```

4. **SingleThreadExecutor**

   通过Executors的newSingleThreadExecutor()方法来创建，它内部只有一个核心线程，它确保所有任务进来都要排队按顺序执行。它的意义在于，统一所有的外界任务到同一线程中，让调用者可以忽略线程同步问题。

   ```java
       public static ExecutorService newSingleThreadExecutor() {
           return new FinalizableDelegatedExecutorService
               (new ThreadPoolExecutor(1, 1,
                                       0L, TimeUnit.MILLISECONDS,
                                       new LinkedBlockingQueue<Runnable>()));
       }
   ```

   要创建这样的线程池，我们可以

   ```kotlin
   Executors.newSingleThreadExecutor()
   ```

   

5. **SingleThreadScheduledExecutor**

   通过Executors的newSingleThreadScheduledExecutor()方法创建，可以调度命令在给定的延迟后执行或定期执行，能保证任务按照顺序进行，并且在任何时间内不会有多个任务处于活动状态。

   ```java
       public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
           return new DelegatedScheduledExecutorService
               (new ScheduledThreadPoolExecutor(1));
       }
       
       public ScheduledThreadPoolExecutor(int corePoolSize) {
           super(corePoolSize, Integer.MAX_VALUE,
                 DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                 new DelayedWorkQueue());
       }
   ```

   要创建这样的线程池，我们可以

   ```kotlin
   Executors.newSingleThreadScheduledExecutor()
   ```

   

6. **WorkStealingPool**

   通过Executor的newWorkStealingPool()方法创建，API24及以上可用。Java1.8后添加，该线程池中每个线程都维护自己的任务队列。当自己的任务队列执行完成时，会帮助其他线程执行其中的任务。主动找活干。它底层是ForkJoinPool的实现。用守护线程daemon的方式来执行。

   ForkJoinPool  分任务进行计算再合并结果。计算任务需要继承RecursiveTask并重写compute()方法 方法内进行递归调用。  RecursiveTask（递归任务） JDK1.7之后新加。

   ```java
       public static ExecutorService newWorkStealingPool() {
           return new ForkJoinPool
               (Runtime.getRuntime().availableProcessors(),
                ForkJoinPool.defaultForkJoinWorkerThreadFactory,
                null, true);
       }
   ```

   要创建这样的线程池，我们可以

   ```kotlin
   Executors.newWorkStealingPool()
   ```

## 线程池的一般使用方法

1. shutDown()，关闭线程池，需要执行完已提交的任务；
2. shutDownNow()，关闭线程池，并尝试结束已提交的任务；
3. allowCoreThreadTimeOut(boolen)，允许核心线程闲置超时回收；
4. execute()，提交任务无返回值；
5. submit()，提交任务并有返回值；

## 参考资料

[Android线程—HandlerThread的使用及原理 - 晴明_](https://www.jianshu.com/p/4a57de01c8f5)

[Android线程与线程池 - 戚继光](https://juejin.im/entry/593109e72f301e005830cd76)

[Java线程池 ThreadPool - maven_hz](https://www.jianshu.com/p/2ce6ca87b685)

