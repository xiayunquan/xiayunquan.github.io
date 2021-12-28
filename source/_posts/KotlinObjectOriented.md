---
title: kotlin面向对象编程
date: 2021-09-24 17:15:04
categories: Kotlin
tags: 面向对象
---

### 定义函数
与java函数不同的是，定义函数要在前面加上"***fun***" 关键字，返回值类型放在方法名之后，如：
```kotlin
fun getResult(a: Int, b: Int): Int {
    return a + b
}
```

如果一个函数只有一个并且是表达式函数体并且是返回类型自动推断的话，可以直接这样写:
```kotlin
fun getResult(a: Int, b: Int) = a + b
```

如果一个函数返回一个无意义的值，好比Java中的void，则函数返回值用***Unit***.
```kotlin
fun getResult(a: Int, b: Int): Unit {
   print(a + b)
}
```
而***Unit***关键字可以省略不写
```kotlin
fun getResult(a: Int, b: Int) {
   print(a + b)
}
```

### 创建数据实体类
```kotlin
data class Person(val name: String, var age: Int)
```

kotlin会为Person类提供一下功能:

- 所有属性的getters以及var定义的setters方法
- equals()
- hashCode()
- toString()
- copy(),可对部分属性值进行修改
- 所有属性的component1,component2..
