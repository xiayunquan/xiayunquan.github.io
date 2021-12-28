---
title: 多线程同步与多线程通信本质（待完善）
date: 2021-09-29 11:16:42
categories: Java
tags: 多线程
---

### 进程与线程的概念

**进程**

操作系统中运行的软件就是进程，一个运行的软件可能包含多个进程，至少有一个进程。

比如Windows电脑运行的文件管理器、浏览器等，下面是我电脑上打开任务管理器（Ctrl + Shift + Esc）所看到的进程信息。

![Windows进程](https://img-blog.csdnimg.cn/37999eab1a4b4785a0f075e6ee4dce90.jpg)

![Windows进程详细信息](https://img-blog.csdnimg.cn/7a66da87d02a48cdb3833154b56fba07.jpg)

再比如Android系统正在运行的日历、电话等也都是进程，要查看Android设备上所有的进程信息，只需要执行`adb shell ps`命令即可。

![Android进程](https://img-blog.csdnimg.cn/c01a624b24d34adaa6be2a5a19f01b69.jpg)

从上面的图中可以看到设备上有很多的进程，每一个进程的id（PID）、父进程id（PPID）、进程名称（NAME）、进程的虚拟内存大小（VSIZE）以及进程驻留在内存中的实际内存大小等信息都能看见。

为了简洁，我去掉了部分打印信息，可以看到图中用1、2、3、4标识的位置，首先位置１对应的是**init**进程，它是Android系统的根进程；然后位置2对应的是**zygote**进程，它的PPID是1，即init进程的PID，说明zygote的父进程是init进程；最后看看位置3、4，位置3是一些系统应用对应的进程，可以看到它们的父进程id是zygote进程的PID，位置4是我自己的应用，它的PPID同样也是zygote的PID，说明系统默认应用和我们自己安装的应用的进程都是由zygote进程孵化出来的。

**线程**

线程是进程的一个最小执行单元，用编程的思维去理解线程的概念就是程序按代码顺序执⾏下来，执⾏完毕就结束的⼀条线。一个运行中的进程可能包含多个线程，比如一个运行中的App包含一个主线程（UI线程），包含一个GC线程，还可能包含一个或多个子线程（工作线程）。

UI 线程为什么不会结束？因为它在初始化完毕后会执⾏死循环，循环的内容是刷新界⾯。

### 如何开启一个线程？

知道了进程和线程的基本概念，然后看看如何创建并开启一个线程，主要有以下几种方式。

- #### Thread和Runnable方式

  ```kotlin
  // 创建线程
  val thread = object : Thread() {
      override fun run() {
          Log.i("Thread", "Thread Started!")
      }
  }
  // 开启线程
  thread.start()

  // 上面的代码可以使用下面的Kotlin扩展函数，效果一样
  thread {
      Log.i("Thread", "Thread Started!")
  }

  // 创建Runnable
  val runnable = Runnable { 
  	println("Thread with Runnable started!") 
  }
  val thread = Thread(runnable)
  thread.start()
  ```

- #### ThreadFactory方式

  ```kotlin
  val factory = object : ThreadFactory() {
      var num = 0
      override fun newThread(r: Runnable?): Thread {
          num ++
          return Thread(r, "Thread $num")
      }
  }
  val runnable = Runnable {
      Log.i("ThreadFactory", Thread.currentThread().name + "started!")
  }
  val thread1 = factory.newThread(runnable)
  thread1.start()
  val thread2 = factory.newThread(runnable)
  thread2.start()
  ```

- #### Executor 和线程池方式

  ```kotlin
  val runnable = Runnable { 
      println("Thread with Runnable started!") 
  }
  // 创建线程池
  val executor: Executor = Executors.newCachedThreadPool()
  // val executor: ExecutorService = Executors.newFixedThreadPool(5)
  // val executor: ExecutorService = Executors.newSingleThreadExecutor()
  // val executor: ExecutorService = Executors.newScheduledThreadPool(2)
  executor.execute(runnable)
  executor.execute(runnable)
  executor.execute(runnable)
  ```

- #### Callable 和 Future方式

  Callable接口实际是属于Executor框架中的功能接口，Callable接口与Runnable接口的功能相似，但功能比Runnable更加强大，主要有以下3点：

  1. Callable可以在任务接受后提供一个返回值，Runnable无法提供这个功能。
  2. Callable中的call()方法可以抛出异常，而Runnable的run()方法不能抛出异常。
  3. 运行Callable可以拿到一个Future对象，Future对象表示异步计算的结果，他提供了检查计算是否完成的方法。由于线程属于异步计算模型，因此无法从别的线程中得到函数的返回值，在这种情况下就可以使用Future来监视目标线程调用call()方法的情况，但调用Future的get()方法以获取结果时，当前线程就会阻塞，直到call()方法的返回结果。

  ​

  ```kotlin
  val callable: Callable<String?> = object : Callable<String?> {
      override fun call(): String? {
          try {
              Thread.sleep(1500)
          } catch (e: InterruptedException) {
              e.printStackTrace()
          }
          return "Done!"
      }
  }
  val executor = Executors.newCachedThreadPool()
  val future: Future<String> = executor.submit<String>(callable)
  try {
      val result: String = future.get()
      Log.i("Callable", "result: $result")
  } catch (e: InterruptedException) {
      e.printStackTrace()
  } catch (e: ExecutionException) {
      e.printStackTrace()
  }
  ```



线程的创建方式主要就上面几种，因为线程的创建需要内存开销，如果在我们的程序中需要开启多个线程，一般建议使用线程池缓存方式来管理线程的创建，直接使用new Thread不仅会带来很大的性能消耗，而且不能主动停止线程，不利于统一管理，而Callable一般使用的场景较少。

### 线程间的交互

知道了如何创建线程，那么当创建了多个线程之后，它们之间如何进行通信，按照什么顺序执行，多个线程执行同一块代码如何保证数据同步，下面就来说说多线程之间的通信。



