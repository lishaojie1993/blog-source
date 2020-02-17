---
title: 线程池实现原理与源码分析
date: 2019-03-11 21:26:44
categories: 
  - Java
  - 高并发
tags: 
  - 高并发
  - 线程池
top_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gash4k2x0uj31900u0b1g.jpg
---

## 线程池原理

### 使用线程池的好处

1. **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
3. **提高线程的可管理性**。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源,还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。

### 线程池的实现原理

1. 线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
2. 
   线程池判断工作队列是否已经满。如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。
3. 
   线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。<!--more-->

对应到代码层面就是`ThreadPoolExecutor`执行`execute()`方法

1. 当<kbd>workerCount</kbd> < <kbd>corePoolSize</kbd>，创建新线程执行任务。
2. 当<kbd>workerCount</kbd> >= <kbd>corePoolSize</kbd>，并且阻塞队列workQueue未满，把新的任务放入阻塞队列。
3. 当workQueue已满，并且<kbd>workerCount</kbd> >= <kbd>corePoolSize</kbd>，并且<kbd>workerCount</kbd> < <kbd>maximumPoolSize</kbd>，创建新线程执行任务。
4. 当workQueue已满，并且<kbd>workerCount</kbd> >= <kbd>maximumPoolSize</kbd>，采取拒绝策略,根据不同的拒绝策略处理,默认拒绝策略是直接抛异常。

由于创建新线程时（第1步、第3步）需要获取全局锁，所以ThreadPoolExecutor采取上述步骤的总体设计思路，是为了在执行execute()方法时，尽可能地避免获取全局锁（那将会是一个严重的可伸缩瓶颈）。

### 线程池状态转换模型

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gb1qb2e8lbj31lo0jiu0x.jpg)

- `RUNNING` 自然是运行状态，指可以接受任务执行队列里的任务
- `SHUTDOWN` 指调用了 `shutdown()` 方法，不再接受新任务了，但是队列里的任务得执行完毕。
- `STOP` 指调用了 `shutdownNow()` 方法，不再接受新任务，同时抛弃阻塞队列里的所有任务并中断所有正在执行任务。
- `TIDYING` 所有任务都执行完毕，在调用 `shutdown()/shutdownNow()` 中都会尝试更新为这个状态。
- `TERMINATED` 终止状态，当执行 `terminated()` 后会更新为这个状态。

## 线程池的使用

### 创建线程池

我们可以通过ThreadPoolExecutor的**构造方法**来创建线程池

```java
public ThreadPoolExecutor(
   int corePoolSize,
   int maximumPoolSize,                              
      long keepAliveTime,
   TimeUnit unit,
   BlockingQueue<Runnable> workQueue,
   ThreadFactory threadFactory,
   RejectedExecutionHandler handler){
}
```

下面介绍一下代码中的参数：

1. corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，如果当前poolSize<corePoolSize时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。

2. maximumPoolSize（线程池最大数量）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数就没什么效果。

3. keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。

4. TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）。

5. runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。

   - ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。

   - LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
   - SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于Linked-BlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
   - PriorityBlockingQueue：一个具有优先级的无限阻塞队列。

6. ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架guava提供的ThreadFactoryBuilder可以快速给线程池里的线
   程设置有意义的名字，代码如下。

   ```java
   new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
   ```

7. RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。在JDK 1.5中Java线程池框架提供了以下4种策略。

   - AbortPolicy：直接抛出异常。(默认)
   - CallerRunsPolicy：只用调用者所在线程来运行任务。
   - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
   - DiscardPolicy：不处理，丢弃掉。


当然，也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化存储不能处理的任务。

### 向线程池提交任务

可以使用两个方法向线程池提交任务，分别为execute()和submit()方法。

区别:

- execute()方法用于提交**不需要返回值**的任务;
- submit()方法用于提交**需要返回值**的任务。

execute方法：


execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功。通过以下代码可知execute()方法输入的任务是一个Runnable类的实例。

```java
threadsPool.execute(new Runnable() {
  @Override
  public void run() {
  	// TODO Auto-generated method stub
  }
});
```

submit方法：


线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，get()方
法会阻塞当前线程直到任务完成，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

```java
Future<Object> future = executor.submit(haveReturnValuetask);
try {
	Object s = future.get();
} catch (InterruptedException e) {
	// 处理中断异常
} catch (ExecutionException e) {
	// 处理无法执行任务异常
} finally {
	// 关闭线程池
	executor.shutdown();
}
```

### 关闭线程池

- shutdown()
- shutdownNow()

它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。只要调用了这两个关闭方法中的任意一个，isShutdown方法就会返回true。当所有的任务都已关闭后，才表示线程池关闭成功，这时调用isTerminaed方法会返回true。

区别：

shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

总结：

至于应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调shutdown方法来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow方法。

### 线程池实例Demo

1. 首先构造一个线程池，用ArrayBlockingQueue作为其等待队列，队列初始化容量为1。该线程池核心容量为 10，最大容量为20，线程存活时间为1分钟。

   ```java
   static BlockingQueue blockingQueue=new ArrayBlockingQueue<>(1);
   static ThreadPoolExecutor threadPoolExecutor=new ThreadPoolExecutor(10, 20, 1, TimeUnit.MINUTES, blockingQueue);
   ```

2. 另外构造了一个实现Runable接口的类TaskBusyWithoutResult类，其模拟一个繁忙的任务：

   ```java
   static class TaskBusyWithoutResult implements Runnable
   {
      public TaskBusyWithoutResult()
      {
      }
      @Override
      public void run() 
      {
          System.out.println("线程"+Thread.currentThread()+"开始运行");
          int i=10000*10000;
          while(i>0)
          {
              i--;
          }
          System.out.println("线程"+Thread.currentThread()+"运行结束");
      }
   }
   ```

3. 向线程池提交20个任务，执行任务

   ```java
   public static void main(String[] args) {
       for (int i = 0; i < 20; i++) {
           Runnable runnable = new TaskBusyWithoutResult();
           threadPoolExecutor.submit(runnable);
       }
   }
   ```

## 线程池源码分析

**基于JDK1.8**

### ctl 变量

后面execute方法会用到ctl，这个变量是为了把工作线程数量和线程池状态放在一个int型变量存储而设置的一个原子类型的变量。 在ctl中，低位的29位表示工作线程的数量，高位用来表示RUNNING、SHUTDOWN、STOP等状态。上面定义的三个方法只是为了计算得到线程池的状态和工作线程的数量，以及得到ctl。

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
   private static final int COUNT_BITS = Integer.SIZE - 3;
   private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

   // runState is stored in the high-order bits
   private static final int RUNNING    = -1 << COUNT_BITS;
   private static final int SHUTDOWN   =  0 << COUNT_BITS;
   private static final int STOP       =  1 << COUNT_BITS;
   private static final int TIDYING    =  2 << COUNT_BITS;
   private static final int TERMINATED =  3 << COUNT_BITS;

   // Packing and unpacking ctl
   private static int runStateOf(int c)     { return c & ~CAPACITY; }
   private static int workerCountOf(int c)  { return c & CAPACITY; }
   private static int ctlOf(int rs, int wc) { return rs | wc; }
```

### execute()方法

- if (! isRunning(recheck) && remove(command))这句。由于&&是短路与，意思就是前面如果非真，后面不会执行。所以如果线程池不是Running状态时，才把现成移出工作队列，再使用饱和策略。
- addWorker(command, false)：我们看到有addWorker(command, true)和addWorker(command, false)。true和false分别代表在检查工作线程数量的时候是应该与corePoolSize对比还是应该maximumPoolSize对比。只有第一个if时，也就是当前线程数小于corePoolSize时才用addWorker(command, true)。

```java
public void execute(Runnable command) {
   //如果提交了空的任务 抛出异常
 if (command == null)
   throw new NullPointerException();
 int c = ctl.get();//获取当前线程池的状态
   //检查当前工作线程数量是否小于核心线程数量
 if (workerCountOf(c) < corePoolSize) {
   //通过addWorker方法提交任务
   if (addWorker(command, true))
     return;
   c = ctl.get();//如果提交失败 需要二次检查状态
 }
 //向工作线程提交任务 
 if (isRunning(c) && workQueue.offer(command)) {
   // 再次检查状态
      int recheck = ctl.get();
      
      if (! isRunning(recheck) && remove(command))
        reject(command);
      else if (workerCountOf(recheck) == 0)
        addWorker(null, false);
 }
 //扩容失败 则拒绝任务
 else if (!addWorker(command, false))
   reject(command);
}
```

### addWorker()方法

这个方法是任务提交的一个核心方法，在里面完成了状态检查、新建任务、执行任务等一系列动作，主要工作是在线程池中创建一个新的线程并执行。

这个方法可以分为两个阶段来看，第一个阶段是判断是否有必要新增一个工作线程，如果有则利用CAS更新工作线程的数量；第二部分是将提交的任务封装成一个工作线程Worker然后加入到线程池的容器中，开始执行新提交的任务。这个Worker在执行完任务后，还会循环地获取工作队列里的任务来执行。

```java
private boolean addWorker(Runnable firstTask, boolean core) {
       retry:
   //死循环更新状态
       for (;;) {
           int c = ctl.get();
           int rs = runStateOf(c);//获取运行状态

       //检查线程池是否处于关闭状态
           if (rs >= SHUTDOWN &&
               ! (rs == SHUTDOWN &&
                  firstTask == null &&
                  ! workQueue.isEmpty()))
               return false;
   
           for (;;) {
         //获取当前工作线程数量
               int wc = workerCountOf(c);
       //如果已经超过corePoolSize获取maximumPoolSize 返回false
               if (wc >= CAPACITY ||
                   wc >= (core ? corePoolSize : maximumPoolSize))
                   return false;
       //CAS增加一个工作线程
               if (compareAndIncrementWorkerCount(c))
                break retry;
       //再次获取状态
               c = ctl.get();  // Re-read ctl
       //如果状态更新失败 则循环更新
               if (runStateOf(c) != rs)
                   continue retry;
               // else CAS failed due to workerCount change; retry inner loop
           }
       }

       boolean workerStarted = false;
       boolean workerAdded = false;
       Worker w = null;
       try {
           w = new Worker(firstTask);//初始化一个工作线程
           final Thread t = w.thread;
           if (t != null) {
        //获得锁
               final ReentrantLock mainLock = this.mainLock;
               mainLock.lock();
               try {
                   // Recheck while holding lock.
                   // Back out on ThreadFactory failure or if
                   // shut down before lock acquired.
                   int rs = runStateOf(ctl.get());

                   if (rs < SHUTDOWN ||
                       (rs == SHUTDOWN && firstTask == null)) {
                       if (t.isAlive()) // precheck that t is startable
                           throw new IllegalThreadStateException();
           //添加工作这到hashset中保存
                       workers.add(w);
                       int s = workers.size();
                       if (s > largestPoolSize)
                           largestPoolSize = s;
                       workerAdded = true;
                   }
               } finally {
                   mainLock.unlock();
               }
               if (workerAdded) {
       //工作线程启动 执行第一个任务 就是新提交的任务
                   t.start();
                   workerStarted = true;
               }
           }
       } finally {
           if (! workerStarted)
               addWorkerFailed(w);
       }
       return workerStarted;
   }
```

Worker的构造方法如下：

```java
/**
 * Creates with given first task and thread from ThreadFactory.
 * @param firstTask the first task (null if none)
 */
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);
}
```

### runWorker()方法

在addWorker方法快要结束的地方，调用了t.start()方法，我们知道它实际执行的就是Worker对象的run()方法，而worker的run()方法是这样定义的：

```java
/** Delegates main run loop to outer runWorker  */
public void run() {
    runWorker(this);
}
```

它实际上是将自己委托给线程池的runWorker方法

```java
final void runWorker(Worker w) {
       
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
       try {
       //不断地从blockingQueue获取任务
           while (task != null || (task = getTask()) != null) {
               w.lock();
               // If pool is stopping, ensure thread is interrupted;
               // if not, ensure thread is not interrupted.  This
               // requires a recheck in second case to deal with
               // shutdownNow race while clearing interrupt
               if ((runStateAtLeast(ctl.get(), STOP) ||
                    (Thread.interrupted() &&
                     runStateAtLeast(ctl.get(), STOP))) &&
                   !wt.isInterrupted())
                   wt.interrupt();
               try {
           //执行beforeExecute方法
                   beforeExecute(wt, task);
                   Throwable thrown = null;
                   try {
           //调用Runable的run方法
                       task.run();
                   } catch (RuntimeException x) {
                       thrown = x; throw x;
                   } catch (Error x) {
                       thrown = x; throw x;
                   } catch (Throwable x) {
                       thrown = x; throw new Error(x);
                   } finally {
           // 执行aferExecute方法
                       afterExecute(task, thrown);
                   }
               } finally {
                   task = null;
                   w.completedTasks++;
                   w.unlock();
               }
           }
           completedAbruptly = false;
       } finally {
           processWorkerExit(w, completedAbruptly);
       }
}
```

总结一下runWorker方法的执行过程：

1. while循环中，不断地通过getTask()方法从workerQueue中获取任务
2. 如果线程池正在停止，则中断线程。否则调用3.
3. 调用task.run()执行任务；
4. 如果task为null则跳出循环，执行processWorkerExit()方法，销毁线程workers.remove(w);

它在不断执行我们提交的任务的run方法。而这个任务可能是我们新提交的，也有可能是从等待队列中获取的。这样就实现了线程池的完成逻辑。

## ExecutorService

**创建线程池的5个方法：**

### newSingleThreadExecutor

只有一个线程的线程池，因此所有提交的任务是**顺序执行**。

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
```

构造方法源码

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。

返回单线程的Executor，将多个任务交给此Exector时，这个线程处理完一个任务后接着处理下一个任务，若该线程出现异常，将会有一个新的线程来替代。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

------

### newCachedThreadPool

缓存型线程池，先查看池中有没有以前建立的线程，如果有，就重用，如果没有，就建一个新的线程加入池中。如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

```java
ExecutorService executorService = Executors.newCachedThreadPool();
```

构造方法源码

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0,//线程池维护线程的最少数量
                Integer.MAX_VALUE,//线程池维护线程的最大数量
                60L,//线程池维护线程所允许的空闲时间，60秒
                TimeUnit.SECONDS,//线程池维护线程所允许的空闲时间的单位
                new SynchronousQueue<Runnable>());
}
```

实例

```java
public static void main(String[] args){
        ExecutorService exe= Executors.newCachedThreadPool();
        for(int i=1;i<6;i++){
            final int taskID=i;
            exe.execute(new Runnable() {
                public void run() {
                    for(int j=1;j<4;j++){
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("线程ID："+taskID+",执行第 "+j+" 次");
                    }
                }
            });
        }
    }
```

执行结果

```xml
线程ID：3，执行第 1 次
线程ID：4，执行第 1 次
线程ID：2，执行第 1 次
线程ID：5，执行第 1 次
线程ID：1，执行第 1 次

线程ID：2，执行第 2 次
线程ID：3，执行第 2 次
线程ID：4，执行第 2 次
线程ID：1，执行第 2 次
线程ID：5，执行第 2 次

线程ID：2，执行第 3 次
线程ID：3，执行第 3 次
线程ID：4，执行第 3 次
线程ID：5，执行第 3 次
线程ID：1，执行第 3 次
```

可以看到执行结果是5个任务在交替进行的

newCachedThreadPool的总结：

1. 重用：缓存型池子，先查看池中有没有以前建立的线程，如果有，就reuse；如果没有，就建一个新的线程加入池中。
2. 使用场景：缓存型池子通常用于执行一些生存期很短的异步型任务，因此在一些面向连接的daemon型SERVER中用得不多。
3. 超时：能reuse的线程，必须是timeout IDLE内的池中线程，缺省timeout是60s，超过这个IDLE时长，线程实例将被终止及移出池。
4. 结束：注意，放入CachedThreadPool的线程不必担心其结束，超过TIMEOUT不活动，其会自动被终止。

------

### newFixedThreadPool

定长线程池，可控制线程最大并发数。如果当前需要执行的任务超过池大小，那么多出的任务处于等待状态，直到有空闲下来的线程执行任务，如果当前需要执行的任务小于池大小，空闲的线程也不会去销毁。

```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
```

构造方法源码：有两个重载方法

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(
                nThreads,//线程池维护线程的最少数量
                nThreads,//线程池维护线程的最大数量
                0L, //线程池维护线程所允许的空闲时间
                TimeUnit.MILLISECONDS,//线程池维护线程所允许的空闲时间的单位
                new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
 }
```

实例

```java
public static void main(String[] args) {
        ExecutorService exe = Executors.newFixedThreadPool(3);
        for (int i = 1; i < 6; i++) {
            final int taskID = i;
            exe.execute(new Runnable() {
                public void run() {
                    for (int j = 1; j < 4; j++) {
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("线程ID：" + taskID + "，执行第 " + j + " 次");
                    }
                }
            });
        }
    }
```

执行结果

```xml
线程ID：2，执行第 1 次
线程ID：1，执行第 1 次
线程ID：3，执行第 1 次

线程ID：2，执行第 2 次
线程ID：3，执行第 2 次
线程ID：1，执行第 2 次

线程ID：2，执行第 3 次
线程ID：3，执行第 3 次
线程ID：1，执行第 3 次

线程ID：4，执行第 1 次
线程ID：5，执行第 1 次
线程ID：4，执行第 2 次

线程ID：5，执行第 2 次
线程ID：4，执行第 3 次
线程ID：5，执行第 3 次
```

创建了一个固定大小的线程池，容量为3，然后循环执行了5个任务。由输出结果可以看到，前3个任务首先执行完，然后空闲下来的线程去执行第4,5个任务。
newFixedThreadPool的总结：

1. 重用：fixedThreadPool与cacheThreadPool差不多，也是能reuse就用，但不能随时建新的线程。
2. 固定数目：其独特之处在于，任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，直到当前的线程中某个线程终止直接被移出池子。
3. 超时：和cacheThreadPool不同，FixedThreadPool没有IDLE机制（可能也有，但既然文档没提，肯定非常长，类似依赖上层的TCP或UDP IDLE机制之类的）。
4. 使用场景：所以FixedThreadPool多数针对一些很稳定很固定的正规并发线程，多用于服务器。

------

### newScheduledThreadPool

调度型线程池,支持定时及周期性任务执行，也是一个固定长度的线程池。

```java
ScheduledExecutorService exe= Executors.newScheduledThreadPool(3);
```

构造方法源码

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, // 线程池维护线程的最少数量
        Integer.MAX_VALUE, //线程池维护线程的最大数量
        0, // 线程池维护线程所允许的空闲时间
        NANOSECONDS,//线程池维护线程所允许的空闲时间的单位
        new DelayedWorkQueue());
    }
```

实例

```java
public static void main(String[] args){
        ScheduledExecutorService exe= Executors.newScheduledThreadPool(3);
            for(int i=1;i<6;i++){
                final int taskID=i;
                exe.scheduleAtFixedRate(new Runnable() {
                    public void run() {
                        try {
                            Thread.sleep(2000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("线程："+taskID+",时间："+ LocalDateTime.now()+" 执行一次");
                    }
                }, 0, 2, TimeUnit.SECONDS);
            }
    }
```

代码scheduleAtFixedRate后面的参数 0 表示立即执行，2表示2秒执行一次调度。执行结果：

```xml
线程：1,时间：2016-12-11T13:09:18.544 执行一次
线程：2,时间：2016-12-11T13:09:18.544 执行一次
线程：3,时间：2016-12-11T13:09:18.544 执行一次

线程：5,时间：2016-12-11T13:09:20.557 执行一次
线程：1,时间：2016-12-11T13:09:20.557 执行一次
线程：4,时间：2016-12-11T13:09:20.557 执行一次

线程：4,时间：2016-12-11T13:09:22.572 执行一次
线程：2,时间：2016-12-11T13:09:22.572 执行一次
线程：3,时间：2016-12-11T13:09:22.572 执行一次

线程：5,时间：2016-12-11T13:09:24.586 执行一次
线程：1,时间：2016-12-11T13:09:24.586 执行一次
线程：2,时间：2016-12-11T13:09:24.586 执行一次
```

newScheduledThreadPool的线程池大小只设置了3，所以一次只能执行3个线程，然后可以看到每2秒执行一次任务调度。

------

### newSingleThreadScheduledPool

单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。如果当前线程意外终止，会创建一个新线程继续执行任务，这和我们直接创建线程不同，也和newFixedThreadPool(1)不同。

```java
ExecutorService exe= Executors.newSingleThreadExecutor();
```

构造方法源码

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

实例

```java
public static void main(String[] args){
        ExecutorService exe= Executors.newSingleThreadExecutor();
        for(int i=1;i<6;i++){
            final int taskID=i;
            exe.execute(new Runnable() {
                public void run() {
                    for(int j=1;j<4;j++){
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("线程ID："+taskID+"，执行第 "+j+" 次");
                    }
                }
            });
        }
    }
```

执行结果

```xml
线程ID：1，执行第 1 次
线程ID：1，执行第 2 次
线程ID：1，执行第 3 次
线程ID：2，执行第 1 次
线程ID：2，执行第 2 次
线程ID：2，执行第 3 次
线程ID：3，执行第 1 次
线程ID：3，执行第 2 次
线程ID：3，执行第 3 次
线程ID：4，执行第 1 次
线程ID：4，执行第 2 次
线程ID：4，执行第 3 次
线程ID：5，执行第 1 次
线程ID：5，执行第 2 次
线程ID：5，执行第 3 次
```

每个结果都是相隔0.5秒打印出来的，顺序执行下去。



- 

