---
title: Kotlin条件控制与循环
date: 2021-09-24 16:45:57
categories: Kotlin
tags: Kotlin语法
---

## Kotlin 的逻辑控制

程序的执行语句主要分为3种：**顺序语句**、**条件语句**和**循环语句**。顺序语句很好理解，就是代码一行一行地往下执行就可以了，但是这种执行方式在很多情况下并不能满足我们的编程需求，这时就需要引入条件语句和循环语句了。

### 条件控制

##### if条件语句

Kotlin中的条件语句主要有两种实现方式：if和when。

一个 if 语句包含一个布尔表达式和一条或多条语句。

```kotlin
fun largerNumber(num1: Int, num2: Int): Int {
	var value = 0
  	if (num1 > num2) {
 		value = num1
 	} else {
 		value = num2
 	}
 	return value
}
```

从上面的示例中可以看出Kotlin中的if用法和Java中基本上是一样的。但是Kotlin中的if语句相比于Java有一个额外的功能，它是可以有返回值的，返回值就是if语句每一个条件中最后一行代码的返回值。因此，上述代码就可以简化成如下形式：

```kotlin
fun largerNumber(num1: Int, num2: Int): Int {
 	val value = if (num1 > num2) {
 		num1
 	} else {
 		num2
 	}
 	return value
}
```

由于没有重新赋值的情况了，因此可以使用val关键字来声明value变量，最

终将value变量返回。而且value其实也是一个多余的变量，我们可以直接将if语句返回，这样代码将会变得更加精简，如下所示：

```kotlin
fun largerNumber(num1: Int, num2: Int): Int {
   return if (num1 > num2) {
 		num1
 	} else {
 		num2
 	}
}
```

或者直接写成一行， 返回值类型可以自动推导出来，所以也可以省略：

```kotlin
fun largerNumber(num1: Int, num2: Int) = if (num1 > num2) num1 else num2
```

##### when表达式

在条件分支变得很多的时候， 使用if语句就会产生大量的冗余代码， 而且if语句也有一定的局限性，这时就需要使用when表达式了。

Kotlin中的when语句有点类似于Java中的switch语句，但它又远比switch语句强大得多。 Java中的switch语句并不怎么好用。

首先，switch只能传入整型或短于整型的变量作为条件，JDK 1.7之后增加了对字符串变量的支持，如果不是这几种类型的变量，则不能使用switch。

其次，switch中的每个case条件都要在最后主动加上一个break，否则执行完当前case之后会依次执行下面的case，这一特性可能导致许多奇怪的bug，就是因为有人忘记添加break，执行了多余的case语句。

而Kotlin中的when语句不仅解决了上述痛点，还增加了许多更为强大的新特性。when运行原理就是将它的参数依次和所有的分支条件顺序比较，直到某个分支满足条件，when语句允许传入一个任意类型的参数，可以是具体的值或某个类型；而when的分支条件语句可以是判断某个具体的值、或着是判断是否为某个类型、或者是判断是否在某个区间。当所有分支语句都不满足时，可以使用一个else语句执行匹配失败的流程，类似于Java的switch中的default语句。

下面是一个完整的示例：

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    is String -> print("x是一个string")
  	in 1..4 -> print("在1到4之间")
    else -> {
        print("x 不是 1 ，也不是 2")
    }
}
```

when语句的基本用法就是这些，但其实when语句还有一种不带参数的用法，虽然这种用法可能不太常用，但有的时候却能发挥很强的扩展性。这种用法是将判断的表达式完整地写在when的结构体当中。如下所示：

```kotlin
when {
    x == 1 -> print("x == 1")
    x == 2 -> print("x == 2")
    x is String -> print("x是一个string")
  	x in 1..4 -> print("在1到4之间")
    else -> {
        print("x 不是 1 ，也不是 2")
    }
}
```



### 循环控制

Java中主要有两种循环语句：while循环和for循环。而Kotlin也提供了while循环和for循环，其中while循环不管是在语法还是使用技巧上都和Java中的while循环没有任何区别，而Kotlin在for循环方面做了很大幅度的修改，Java中最常用的for-i循环在Kotlin中直接被舍弃了，而Java中另一种for-each循环则被Kotlin进行了大幅度的加强，变成了for-in循环，在了解for-in循环之前，需要先了解一下kotlin区间的概念。

##### 区间

kotlin中可以使用`..` 、`until` 、 `downTo`表示一个区间范围， `..`和`until`是从小到大，`downTo`是从大到小。

`..` 表示两个端点都在区间的升序闭区间，如：

```kotlin
val range = 0..5
```

range的取值范围为: 0 1 2 3 4 5.



`until` 表示左闭右开升序区间，如：

```kotlin
val range = 0 until 5
```

range的取值范围为: 0 1 2 3 4.



`downTo` 表示两个端点都在区间的降序闭区间，如：

```kotlin
val range = 0 downTo 5
```

range的取值范围为: 5 4 3 2 1 0.



##### for-in循环

默认情况下，for-in循环每次执行循环时会在区间范围内递增1或递减1，相当于Java for-i循环中的i++或i--效果，而如果要跳过其中的一些元素，可以使用step关键字：step表示步长，默认step为1，现在给step设置为2，如下示例

```kotlin
for (i in 1..6 step 2) {
	...
}
```

i的取值范围为: 1 3 5.



for-in循环除了可以对区间进行遍历之外，还可以用于遍历数组和集合，下面是一些for-in循环示例：

```kotlin
val items = listOf("实", "力", "K", "O")
//实力KO
for (item in items) {
	print(items.get(item))
}

//实力KO
for (index in items.indices) {
	print(items.get(index))
}

//实力KO
for ((index, value) in items.withIndex()) {
	print(value)
}

//实力KO
items.forEach {
	print(it)
}
```

##### while循环

while循环和Java中的while循环一样， 语法如下：

```kotlin
while (条件) {
	执行逻辑
}
```

当满足条件时就会执行结构体中的逻辑，然后再次执行判断条件，直到不满足条件才结束while循环，如下示例：

```kotlin
val items = listOf("实", "力", "K", "O")
var index = 0
while (index < items.size) {
	print(items.get(index))
	index++
}
//当index小于items的size时， 执行index++，一直到index=items.size时结束
```

同Java一样，kotlin也支持do while循环，语法如下：

```kotlin
do {
	执行逻辑
} while (条件)
```

do while循环与上面的while循环的区别在于do while循环会先执行一次结构体中的逻辑，然后再判断条件是否满足进行下一次循环， 而while循环是先判断条件是否满足再执行逻辑，所以do while循环至少会执行一次结构体逻辑， 而while至少执行0次结构体逻辑。

##### 跳出循环

Kotlin 有三种结构化跳转表达式：

- *return*,   默认从最直接包围它的函数或者匿名函数返回。
- *break*,  终止最直接包围它的循环。
- *continue*, 继续下一次最直接包围它的循环。

在 Kotlin 中任何表达式都可以用标签（label）来标记。 标签的格式为标识符后跟 @ 符号，例如：abc@、fooBar@

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop
    }
}
```

标签限制的 break 跳转到刚好位于该标签指定的循环后面的执行点。 continue 继续标签指定的循环的下一次迭代.
