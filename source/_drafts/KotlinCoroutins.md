---
title: Kotlin协程
categories: Kotlin
tags: 协程
---

#### 协程是什么?

kotlin协程是一套有kotlin官方提供的线程API、线程框架，像Java的Executor和Android的AsyncTask一样让我们不用不用过多的关心线程也可以方便的写出并发操作。

```kotlin
//Thread
Thread {
  ...
}.start()

//Executor
val executor = Excutors.newCacheThreadPool()
executor.execute {
  ...
}

//协程
launch {
  ...
}
```

既然有了像Java的Executor和Android的AsyncTask这样的API，那为什么还需要弄一个协程呢？原因是Kotlin协程可以结合Kotlin语言的优势，写出类似于写同步代码的形式来实现异步任务，使用起来非常方便。



#### 添加协程依赖到项目中

要使用Kotlin，我们需要在`build.gradle(Module:app)`添加如下2个依赖：

- **kotlinx-coroutines-core**  使用协程的主要接口库
- **kotlinx-coroutines-android**  在协程中支持Android主线程

```groovy
dependencies {
  def coroutinesVersion = "1.3.9"
  ...
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutinesVersion"
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$coroutinesVersion"
}
```

#### 第一个协程示例

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L)
        println("World!")
    }
    println("Hello,") // 主线程中的代码会立即执行
    runBlocking {     // 但是这个表达式阻塞了主线程
        delay(2000L)  // ……我们延迟 2 秒来保证 JVM 的存活
    } 
}
```

这里使用了`GlobalScope.launch`来开启一个协程。

调用了 `runBlocking` 的主线程会一直 *阻塞* 直到 `runBlocking` 内部的协程执行完毕。上面的示例也可以使用下面的形式:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // 开始执行主协程
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L)
        println("World!")
    }
    println("Hello,") // 主协程在这里会立即执行
    delay(2000L)      // 延迟 2 秒来保证 JVM 存活
}
```

这里的 `runBlocking<Unit> { …… }` 作为用来启动顶层主协程的适配器。
我们显式指定了其返回类型 `Unit`，因为在 Kotlin 中 `main` 函数必须返回 `Unit` 类型。

延迟一段时间来等待另一个协程运行并不是一个好的选择。让我们显式（以非阻塞方式）等待所启动的后台Job执行结束：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = GlobalScope.launch { // 启动一个新协程并保持对这个作业的引用
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // 等待直到子协程执行结束
}
```

现在，结果仍然相同，但是主协程与后台作业的持续时间没有任何关系了。

协程的实际使用还有一些需要改进的地方。当我们使用 `GlobalScope.launch` 时，我们会创建一个顶层协程。虽然它很轻量，但它运行时仍会消耗一些内存资源。如果我们忘记保持对新启动的协程的引用，它还会继续运行。如果协程中的代码挂起了会怎么样（例如，我们错误地延迟了太长时间），如果我们启动了太多的协程并导致内存不足会怎么样？必须手动保持对所有已启动协程的引用并join之很容易出错。

有一个更好的解决办法。我们可以在代码中使用结构化并发。我们可以在执行操作所在的指定作用域内启动协程，而不是像通常使用线程（线程总是全局的）那样在 GlobalScope中启动。

在我们的示例中，我们使用runBlocking协程构建器将  `main` 函数转换为协程。包括 `runBlocking` 在内的每个协程构建器都将CoroutineScope的实例添加到其代码块所在的作用域中。我们可以在这个作用域中启动协程而无需显式 `join` 之，因为外部协程（示例中的 `runBlocking`）直到在其作用域中启动的所有协程都执行完毕后才会结束。因此，可以将我们的示例简化为：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { // 在 runBlocking 作用域中启动一个新协程
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

协程可以指定代码执行的线程

```kotlin
//IO线程执行
launch(Dispatchers.IO) {
  ...
}

//主线程执行
launch(Dispatchers.Main) {
  ...
}
```

使用withContext函数可以在协程内部自动切换线程

```kotlin
launch(Dispatchers.Main) {
  val image = withContext(Dispatchers.IO) {
  	//IO线程
    getImage()
  }
  //自动切回主线程
  avatarIv.setImageBitmap(image)
}
```

我们也可以提取我们的代码到协程外面，但是需要使用`suspend`关键字来修饰方法，标识这个方法是一个挂起函数

```kotlin
launch(Dispatchers.Main) {
  val image = suspendGetImage()
  //自动切回主线程
  avatarIv.setImageBitmap(image)
}

suspend fun suspendGetImage() : Bitmap{
  return withContext(Dispatchers.IO) {
  	//IO线程
    getImage()
  }
}
```



#### 挂起函数

在Android中，主线程是处理所有UI更新的单个线程。它也是调用所有点击处理程序和其他UI回调的线程。

因此，为了保证它平稳运行，给用户良好的体验，我们需要将一些耗时的操作放到其他线程中处理，在其他线程处理结束之后需要通知主线程执行更新操作，如何通知呢？常见的一种模式就是通过回调的形式，类似下面的代码：

```kotlin
getUserInfo(object : CallBack {
  override fun onSuccess(user: String) {
    getFriendList(user, object: CallBack {
      override fun onSuccess(friendList: String) {
        getFeedList(friendList, object: CallBack {
          override fun onSuccess(feed: String) {
            System.out.println(feed);
          }
        });
      }
    });
  }
});

fun getUserInfo(callback: CallBack) {
    
}
fun getFriendList(user: String, callback: CallBack) {
   
}
fun getFeedList(list: String, callback: CallBack) {
    
}
interface CallBack {
  fun OnSuccess(result: String)
}
```

可以看到使用回调的方式会使得代码嵌套层级变深，这还是仅包含onSuccess 的情况，实际情况会更复杂，因为我们还要处理异常，处理重试，处理线程调度，甚至还可能涉及多线程同步。Kotlin 1.3引入了协程的特性，解决了这个问题，使用协程来实现上面的逻辑，只需要如下代码即可：

```kotlin
val user = getUserInfo()
val friendList = getFriendList(user)
val feedList = getFeedList(friendList)

suspend fun getUserInfo(): String {
    withContext(Dispatchers.IO) {
        //耗时操作
    }
    return "user info"
}

suspend fun getFriendList(user: String): String {
    withContext(Dispatchers.IO) {
        //耗时操作
    }
    return "friend list"
}

suspend fun getFeedList(list: String): String {
    withContext(Dispatchers.IO) {
        ////耗时操作
    }
    return "feed list"
}
```

使用协程之后，我们的代码就可以顺序编写了，这主要是因为每个函数前面使用了`suspend`关键字进行修饰，这种函数就是一个挂起函数。

#### 挂起函数的本质

上面使用了挂起函数的代码看起来就像是同步的代码，但实际上却是异步执行的，协程内部给我们做了线程切换，如下图所示：

![coroutines](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60453cfebece44779b6581aefef14284~tplv-k3u1fbpfcp-watermark.image)

挂起，只是将程序执行流程转移到了其他线程，主线程并未被阻塞。

`suspend` 的本质，就是 `CallBack` 自动帮我们切线程。

虽然我们写出来的代码没有 CallBack，但 Kotlin 的编译器检测到 `suspend` 关键字修饰的函数以后，会自动将挂起函数转换成带有 CallBack 的函数。

如果我们将上面的挂起函数反编译成 Java，结果会是这样：

```java
// Continuation 等价于 CallBack   
public static final Object getUserInfo(Continuation $completion){
  ...
  return "user info";
}
```

从反编译的结果来看，挂起函数确实变成了一个带有 `CallBack` 的函数，只是这个 `CallBack` 的真实名字叫 `Continuation`。

Continuation 在 Kotlin 中的定义：

```java
public interface Continuation<in T> {
    public val context: CoroutineContext
//      相当于 onSuccess     结果   
//                 ↓         ↓
    public fun resumeWith(result: Result<T>)
}
```

以上这个从`挂起函数`转换成`CallBack 函数`的过程，被称为：CPS 转换(Continuation-Passing-Style Transformation)。它其实就是：`将程序接下来要执行的代码进行传递的一种模式。`而`CPS 转换`，就是将原本的`同步挂起函数`转换成`CallBack 异步代码`的过程。这个转换是编译器在背后做的，我们程序员对此无感知。

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de2b6b97c0284becbc6d329cbd66e4ab~tplv-k3u1fbpfcp-watermark.image)

这种转换不会为每个挂起函数都生成一个CallBack，而是采用协程状态机，通过 label 代码段嵌套，配合 switch 巧妙构造出一个状态机结构进行控制，实际上用的是同一个continuation实例。

#### 协程很轻量

```kotlin
repeat(100_000) { // 启动大量的协程
	launch {
    	delay(1000L)
        print(".")
    }
}
```

上面代码启动了 10 万个协程，每延迟1 秒钟创建一个协程。

如果使用线程来实现上面的逻辑，则在性能耗时方面会相差很大。

```kotlin
repeat(100_000) {
	thread {
    	Thread.sleep(1000L)
    	print(".")
	}
}
```

但是如果使用线程池来实现，则协程和线程池的性能差别相对不大。

```kotlin
val executor = Executors.newCachedThreadPool()
val task = Runnable {
	Thread.sleep(1000L)
	print(".")
}
repeat(100_1000) {
	executor.execute(task)
}
```

由于Thread.sleep()方法会有一定的性能消耗，如果使用性能更高的newSingleTheadScheduledExecutor来实现，则和上面的协程实现性能基本上没什么区别，所以协程的轻量级也是相对于直接用线程而言。

```kotlin
val executor = Executors.newSingleThreadScheduledExecutor()
val task = Runnable {
	print(".")
}
repeat(100_1000) {
	executor.schedule(task, 1, TimeUnit.SECONDS)
}
```


