---
title: Kotlin开发效率提升技巧—扩展函数
date: 2021-09-24 17:18:50
categories: Kotlin
tags: Kotlin扩展函数
---

### 什么是扩展函数？
在介绍什么是Kotlin的扩展函数之前，先来看一个具体的例子。
在日常Android任务开发中，一般在设置View的尺寸时都应该使用**dp**值，但是View系统底层都是使用的**px**值来进行计算的，所以需要把dp值转成对应的px值。
在Java中，可以写一个dp转px的工具类，代码如下：
```java
public class Util {
	public static float dp2px(float dpValue, Context context) {
		return TypedValue.applyDimension(
				TypedValue.COMPLEX_UNIT_DIP,
				dpValue,
				context.getResources().getDisplayMetrics()
		)
	}
}
```
之后，在需要dp转px的地方调用如下方法就可以了。
```java
float pxValue = Util.dp2px(20f, context)
```
这也是在Java中最常规的操作了，现在使用Kotlin之后，同样也可以定义一个一模一样的工具类进行数值转换，但是Kotlin中有更加优雅的写法。
新建一个Extensions.kt文件，编写如下代码：
```kotlin
fun Float.dp2px(context: Context) : Float {
    return TypedValue.applyDimension(
        TypedValue.COMPLEX_UNIT_DIP,
        this,
        context.resources.displayMetrics
    )
}
```
然后，使用下面的写法就可以进行dp转px了
```kotlin
val pxValue = 20f.dp2px(context)
```
Kotlin扩展函数的基本结构如下：
```kotlin
fun ClassName.methodName(param: ParamType) : ReturnType {
    // 具体实现
}
```
- ClassName就是实际调用该方法的具体类的类型
- methodName是方法的名称
- param是方法传入的参数，可以为空，也可以有多个
- ReturnType是方法的返回值类型，可以是任意类型

相比于定义一个普通的函数，定义扩展函数只需要在函数名的前面加上一个**ClassName.** 的语法结构，就表示将该函数添加到指定类当中了。
另外，扩展函数是一个顶层函数，可以将它写在任意一个类中，一般就以ClassName名称新建一个文件，编写跟这个类相关的扩展函数，或者建一个统一管理扩展函数的文件，比如Extensions.kt。

### 扩展函数的本质
 上面的kotlin扩展函数的这种写法给人感觉就是Float这个类中有一个dp2px方法，那么Kotlin是如何实现的这个神奇功能呢，我们看看这个Extensions.kt文件的kotlin字节码反编译成Java语言后都干了些啥。
 具体查看步骤：Android Studio的Tools菜单栏 -> Kotlin -> Show Kotlin Bytecode -> 然后Kotlin Bytecode面板上点击Decompile按钮就可以如下代码（已简化）：
 ```java
public final class ExtensionsKt {
	public static final float dp2px(float $this$dp2px, @NotNull Context context) {
		  Resources var10002 = context.getResources();
		  return TypedValue.applyDimension(1, $this$dp2px, var10002.getDisplayMetrics());
	}
}
 ```
可以看到反编译之后的Java代码，其实就是在ExtensionsKt类中定义了一个**static final**方法，上面写的`20f.dp2px(context)` 就是调用的这个方法，而Float类中并不存在dp2px方法，我们也不可能去修改Float类。我们发现反编译的java函数比我们定义的扩展函数，多了一个float参数，这个参数的类型正是定义扩展函数时的ClassName类型，调用时相当于把20f作为参数传入了静态方法，这样做，一是限定了调用该扩展函数的调用者类型，二是为扩展函数内部计算提供了该对象数据。

对于扩展函数还需要注意以下两点。
**一、扩展函数是静态解析的，是采用静态分派的过程来处理**
这是说调用的扩展函数是由函数调用所在的表达式的类型（定义时的ClassName）来决定的， 而不是由表达式运行时求值结果决定的。如果类本身和其子类都进行了同一个函数的扩展，这函数是不会有重写关系的，在使用的时候，只会根据需要使用该方法的对象的实际类型来决定是调用了哪个扩展函数。
比如下面的代码：

```kotlin
open class Shape

class Rectangle: Shape()

fun Shape.getName() = "Shape"

fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

printClassName(Rectangle())
```
结果会打印“Shape”，相当于是调用了Shape类中的静态方法，子类并不会影响父类。

**二、如果一个类中定义了一个和成员函数一模一样的扩展函数，那么调用的时候始终会调用成员函数。**
就是说一个类中定义了一个函数，然后又定义了一个和这个成员函数一样结构的扩展函数，调用的时候将不会调用到扩展函数，举个例子

```kotlin
class Example {
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType() { println("Extension function") }

// 调用方法
Example().printFunctionType()
```
这段代码执行结果会打印"Class method"

### 更优雅的写法
对于上面dp转px的方法，其实还可以进一步优化，可以定义一个顶层属性。

```kotlin
val Float.dp
    get() = TypedValue.applyDimension(
        TypedValue.COMPLEX_UNIT_DIP,
        this,
        Resources.getSystem().displayMetrics
    )
```
定义了一个dp属性，然后将dp转px的结果赋值给它的get方法，最后调用的时候只需要像下面这种方式写就可以了。

```kotlin
val pxValue = 20f.dp
```
这种写法是不是非常简洁！
这里有个小技巧，我们将传入参数Context来获取Resources的方式改为了**Resources.getSystem()** 的方式，这样就不用每次调用时传入一个context对象了，通过Resources.getSystem()获取的Resources不能用来获取App应用相关的东西，比如包名，但是获取手机系统相关的数据还是可以的，因为这里只需要用到displayMetrics，所以用这种方式获取Resources完全没问题。
其实这种顶层属性的写法同样也是会生成一个静态方法，看下反编译之后的Java代码就明白了。

```java
public final class ExtensionsKt {
   public static final float getDp(float $this$dp) {
      Resources var10002 = Resources.getSystem();
      Intrinsics.checkNotNullExpressionValue(var10002, "Resources.getSystem()");
      return TypedValue.applyDimension(1, $this$dp, var10002.getDisplayMetrics());
   }
}
```

通过这种方法，我们同样可以定义许多其他的扩展函数，sp转px，px转dp，px转sp等，Kotlin也给我们内置了很多实用的扩展函数。
### Kotlin内置扩展函数
Google给我们提供了非常多的扩展函数，这些函数都包含在KTX扩展库中，这个扩展库会在Android Studio创建项目的时候自动引入到build.gradle的dependencies中。
```groovy
implementation "androidx.core:core-ktx:1.3.2"
```
![core-ktv](https://img-blog.csdnimg.cn/ae2cc0d236db4a09898220b8c34d3786.png)
如果能利用好扩展函数这个功能，将会大幅度地提升你的代码质量和开发效率。
简单举几个例子：
**1、关闭文件流**
```kotlin
val output = openFileOutput("data", Context.MODE_PRIVATE)
val writer = BufferedWriter(OutputStreamWriter(output))
writer.use {
	it.write(inputText)
}
```
这里使用了一个use内置扩展函数。它是一个实现Closeable接口的对象可以使用的扩展函数，它会保证在Lambda表达式中的代码全部执行完之后自动将外层的流关闭，这样就不需要我们再编写一个finally语句，手动去关闭流了，是一个非常好用的扩展函数。

**2、监听TextView文本内容变化**
```kotlin
edittext.doBeforeTextChanged{ text, start, count, after -> }
edittext.doOnTextChanged { text, start, before, count -> }
edittext.doAfterTextChanged{}
```
可以很方便实现对TextView的文本内容改变进行监听

**3、SharePreferences保存数据**
在不使用扩展函数的情况下
```kotlin
val editor = getSharedPreferences("test", Context.MODE_PRIVATE).edit()
editor.putString("name", "张三")
editor.putInt("age", 18)
editor.apply()
```
使用了扩展函数之后，edit函数会自动完成提交操作
```kotlin
getSharedPreferences("test", Context.MODE_PRIVATE).edit {
	putString("name", "张三")
	putInt("age", 18)
}
```