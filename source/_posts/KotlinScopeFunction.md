---
title: Kotlin作用域函数--let、with、run、apply、also
date: 2021-09-24 17:16:55
categories: Kotlin
tags: 作用域函数
---

## Kotlin作用域函数

Kotlin 提供了一系列用来在给定对象上下文中执行代码块的函数，包括let、with、run、apply、also。每个库函数都有它的实际应用场景，使用它们能让你的代码会更具有可读性、更优雅、更简洁。善于合理使用标准库函数，也是衡量对Kotlin掌握程度标准之一。 下面是每个函数的基本用法和适用场景，最后对他们进行对比总结以及在实际编码中该如何选择哪个函数。

### let

let函数只接收一个Lambda参数，并且会在Lambda表达式中提供调用对象的上下文it。返回值为函数块的最后一行或指定return表达式。

**底层内联结构：**

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)
```

**语法结构：**

```kotlin
 val result = obj?.let { 
	// it指代obj的上下文
 	it.toString()
    // let函数的返回值 
 }
```

**let的适用场景：**

使用let函数处理需要针对一个可null的对象统一做判空处理。



### with

with函数接收两个参数： 第一个参数可以是一个任意类型的对象， 第二个参数是一个Lambda表达式。

with函数会在Lambda表达式中提供第一个参数对象的上下文，并使用Lambda表达式中的最后一行代码或return语句作为返回值返回。

**底层内联结构：**

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```

**语法结构：**

```kotlin
 val result = with(obj) { 
    // 这里是obj的上下文 "value" 
    // with函数的返回值 
 }
```

**with的适用场景：**

适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，数据model的属性映射到UI上。

例：

```kotlin
with(item) {
	holder.tvTitle.text = "标题: $title"
	holder.tvDescription = "描述: $description"
}
```

在with块中可以直接访问item的实例的公有属性和方法。



### run

run函数的用法和使用场景其实和with函数是非常类似的，只是稍微做了一些语法改动而已。

首先run函数是不能直接调用的，而是一定要调用某个对象的run函数才行；

其次run函数只接收一个Lambda参数，并且会在Lambda表达式中提供调用对象的上下文。

其他方面和with函数是一样的，包括也会使用Lambda表达式中的最后一行代码或者return表达式作为返回值返回，run函数实际上可以说是let和with两个函数的结合体。 

**底层内联结构：**

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R = block()
```

**语法结构：**

```kotlin
val result = obj.run { 
    // 这里是obj的上下文 
    "value" // run函数的返回值 
}
```

**run的适用场景：**

适用于let,with函数任何场景。因为run函数是let,with两个函数结合体，准确来说它弥补了let函数在函数体内必须使用it参数替代对象，在run函数中可以像with函数一样可以省略，直接访问实例的公有属性和方法，另一方面它弥补了with函数传入对象判空问题，在run函数中可以像let函数一样做判空处理。

```kotlin
item?.run {
	holder.tvTitle.text = "标题: $title"
	holder.tvDescription = "描述: $description"
}
```



### apply

apply函数和run函数也是极其类似的，都是要在某个对象上调用，并且只接收一个Lambda参数，也会在Lambda表达式中提供调用对象的上下文，

但是apply函数无法指定返回值，而是会自动返回调用对象本身。代码块中上下文对象是this

**底层内联结构：**

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
```

**语法结构：**

```kotlin
val result = obj.apply { 
    // 这里是obj的上下文 
}
//result是obj对象本身
```

**apply的适用场景：**

整体作用功能和run函数很像，唯一不同点就是它返回的值是对象本身，而run函数是一个闭包形式返回，返回的是最后一行的值。正是基于这一点差异它的适用场景稍微与run函数有点不一样。apply一般用于一个对象实例初始化的时候，需要对对象中的属性进行赋值。或者动态inflate出一个XML的View的时候需要给View绑定数据也会用到，这种情景非常常见。特别是在我们开发中会有一些数据model向View model转化实例化的过程中需要用到。

例：

```kotlin
mSheetDialogView = View.inflate(activity, R.layout.layout_sheet, null).apply{
   isFakeBoldText = true
   max = 10
   progress = 0
}
```



### also

also函数的结构实际上和let很像唯一的区别就是返回值的不一样，let是以闭包的形式返回，返回函数体内最后一行的值，如果最后一行为空就返回一个Unit类型的默认值。而also函数返回的则是传入对象的本身

**底层内联结构：**

```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin(“1.1”)
public inline fun T.also(block: (T) -> Unit): T { block(this); return this }
```

**语法结构：**

```kotlin
val result = obj.also { 
    // it是obj的上下文 
}
//result是obj对象本身
```

**also的适用场景：**

适用于let函数的任何场景，also函数和let很像，只是唯一的不同点就是let函数最后的返回值是最后一行的返回值而also函数的返回值是返回当前的这个对象。一般可用于多个扩展函数链式调用。

例：

```kotlin
items.apply {
	//初始化配置
}.also {
	//附加操作
}.forEach {
	//it.something
}
```



### 总结对比

| 函数名   | 函数体内对象         | 返回值                    | 是否是扩展函数 | 适用场景                                     |
| ----- | :------------- | :--------------------- | :-----: | :--------------------------------------- |
| let   | it指代当前对象       | Lambda表达式最后一行或return语句 |    是    | 适用于处理不为null的操作场景                         |
| with  | this指代当前对象或者省略 | Lambda表达式最后一行或return语句 |    否    | 适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法        |
| run   | this指代当前对象或者省略 | Lambda表达式最后一行或return语句 |    是    | 适用于let,with函数任何场景。                       |
| apply | this指代当前对象或者省略 | 上下文对象(自身)              |    是    | 1、适用于run函数的任何场景，一般用于初始化一个对象实例的时候，操作对象属性，并最终返回这个对象。<br />2、动态inflate出一个XML的View的时候需要给View绑定数据也会用到。<br />3、一般可用于多个扩展函数链式调用。  <br />4、数据model多层级包裹判空处理的问题。 |
| also  | it指代当前对象       | 上下文对象(自身)              |    是    | 适用于let函数的任何场景，一般可用于多个扩展函数链式调用            |



### 如何选择？

要选择正确的函数， 请考虑以下几点：
是否在块中的多个对象上调用方法， 或者将上下中对象的实例作为参数传递？ 如果是， 那么使用以it  而不是  this  形式访问上下文对象的函数之一  （  also  或  let  ） 。 如果在代码块中根本没有用到接收者， 那么使用also。下面是一张来自其他博客的标准函数选择图：

![](https://img-blog.csdnimg.cn/img_convert/688157a7a55c08b243606f9ae5b4dae5.png)

通过这张图可以更加清晰的知道该如何选择使用哪个函数。



**其他注意点：**

- 建议尽量不要使用多个标准库函数进行嵌套，不要为了简化而去做简化，否则整个代码可读性会大大降低，一会是it指代，一会又是this指代，估计隔一段时间后连自己都不知道指代什么了。
- let、with和run函数之所以能够返回其他类型的值，其原理在于lambda表达式内部返回最后一行表达式的值，所以只要最后一行表达式返回不同的对象，那么它们就返回不同类型，表现上就是返回其他类型。
- 关于T.also和T.apply函数为什么都能返回自己本身，是因为在各自Lambda表达式内部最后一行都调用return this,返回它们自己本身，这个this能被指代调用者，是因为它们都是扩展函数特性。
