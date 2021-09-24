---
title: Kotlin中String的常用方法
categories: Kotlin
tags: String
date: 2021-09-24 16:39:37
---


Kotlin的String类中有非常多的方法，下面列举一些经常用到的方法，首先定义一个字符串变量，后面都以这个变量来验证String相关的方法。

```kotlin
//定义字符串
val str = "123456789"
```
## 字符串截取
字符串截取操作可以使用substring、dropXXX系列和takeXXX系列方法
#### drop(n: Int): String
去掉前n个字符，返回其余的字符串，等同于substring(n)
```kotlin
//删掉前3个字符
println(str.drop(3))
//输出结果：456789
```
#### dropLast(n: Int): String
去掉后n个字符，返回其余的字符串，等同于substring(0, str.length - n)
```kotlin
//删掉后4个字符
println(str.dropLast(4))
//输出结果：12345
```
#### dropWhile(predicate: (Char) -> Boolean): String
根据条件从前往后逐一去掉字符，直到不满足条件时则返回后面的字符串，该方法参数是一个lambda表达式，下面举几个例子
```kotlin
//删掉字符串前面等于1或2的字符
val str = "123456789"
println(str.dropWhile {
    it == '1' || it == '2'
})
//输出结果：3456789

val str = "12111223456789"
println(str.dropWhile {
    it == '1' || it == '2'
})
//输出结果：3456789

val str = "13456781219232"
println(str.dropWhile {
    it == '1' || it == '2'
})
//输出结果：3456781219232
```
#### dropLastWhile(predicate: (Char) -> Boolean): String
和dropWhile相反，dropLastWhile是从后面开始根据条件去掉字符串
```kotlin
val str = "13456781219232"
println(str.dropLastWhile {
    it == '1' || it == '2'
})
//输出结果：1345678121923
```

#### take(n: Int): String
获取前n个字符，如果n大于字符串的长度则会返回整个字符串
```kotlin
//获取前4个字符
println(str.take(4))
//输出结果：1234

//获取前20个字符
println(str.take(20))
//输出结果：123456789
```
#### takeLast(n: Int): CharSequence
和take方法对应，takeLast则是从后开始获取n个字符
```kotlin
//获取后3个字符，输出：789
println(str.takeLast(3))
```
#### takeWhile(predicate: (Char) -> Boolean): String
这个方法会从前往后一直返回满足条件的字符，直到不满足条件为止
```kotlin
//获取以6开头的所有字符
println(str.takeWhile {
	it == '6'
})
//输出结果为空

//获取以1或2开头的所有字符
println(str.takeWhile {
	it == '1' || it == '2'
})
//输出结果为空：12
```
#### takeLastWhile(predicate: (Char) -> Boolean): String
这个方法和takeWhile对应,从后往前匹配，返回符合条件的数据
#### takeIf(predicate: (T) -> Boolean): T?
这个方法会根据lambda参数条件判断，满足条件返回自身，否则返回null。参数对象是一个泛型，不仅适合String类型，其他任何类型都可以。
```kotlin
//判断字符串的长度是否大于5
println(str.takeIf {
	it.length > 5
})
//输入结果：123456789
```

#### first() / first(predicate: (Char) -> Boolean): Char
返回字符串的第一个元素，或者可以传入一个过滤条件，返回满足条件的第一个元素
```kotlin
println(str.first()) //输出：1
println(str.first {it == '5'}) //输出：5
println(str.first {it == 'a'}) 
//输出：
Exception in thread "main" java.util.NoSuchElementException: Char sequence contains no character matching the predicate.
	at com.example.demoapp.DemoCheckKt.main(DemoCheck.kt:19)
	at com.example.demoapp.DemoCheckKt.main(DemoCheck.kt)
```
#### firstOrNull() / firstOrNull(predicate: (Char) -> Boolean): Char
和上面的方法一样，返回字符串的第一个元素，或者可以传入一个过滤条件，返回满足条件的第一个元素，但是不一样的地方在于，上面的方法如果没有匹配的结果则会抛出NoSuchElementException异常，而firstOrNull则不会抛出异常，会直接返回null。
```kotlin
println(str.firstOrNull()) //输出：1
println(str.firstOrNull{it == '5'}) //输出：5
println(str.firstOrNull{it == 'a'}) //输出：null
```
与first()和firstOrNull()相对应的方法，last()、lastOrNull()则是获取最后一个满足条件的元素。还有find(predicate: (Char) -> Boolean): Char?方法也是根据条件查找元素，就不再赘述了。
## 空格处理
### trim
```kotlin
//去掉开头的空格
println(" 123  456 ".trimStart())

//去掉结尾的空格
println(" 123  456 ".trimEnd())

//去掉两端的空格
println(" 123  456 ".trim())

//过滤字符:去掉所有空格,输出：12345678
println("123 456  7   8".filter { !it.isWhitespace() })
```

### reversed
```kotlin
//反转字符串，输出：987654321
println(str.reversed())
```
