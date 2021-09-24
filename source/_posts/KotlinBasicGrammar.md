---
title: Kotlin基础语法
date: 2021-09-24 16:25:45
categories: Kotlin
tags: Kotlin语法
---

#### 定义包

```kotlin
package my.demo

import java.util.*
```

#### 定义函数

带有两个 Int 参数、返回 Int 的函数：

```kotlin
fun sum(a: Int, b: Int): Int {
	return a + b
}
```

将表达式作为函数体、返回值类型自动推断的函数

```kotlin
fun sum(a: Int, b: Int) = a + b
```

函数返回无意义的值

```kotlin
fun printSum(a: Int, b: Int):Unit{
  	//$varName 表示变量值,${表达式}执行表达式
	println("sum of $a and $b is ${a + b}")
}
```

Unit返回类型可以省略

```kotlin
fun printSum(a: Int, b: Int):Unit{
	println("sum of $a and $b is ${a + b}")
}
```

可变长参数函数,用 vararg 关键字进行标识：

函数的变长参数可以用 vararg 关键字进行标识：

```kotlin
fun vars(vararg v: Int) {
	for(it in v) {
    	print(it)
	}
}
```

vars(1,2,3,4) ,输出1234



#### 定义变量

定义只读局部变量使用关键字 `val` 定义。只能为其赋值一次。

```kotlin
val a: Int = 1 //立即赋值
val b = 3.0 //自动推断类型为Short
val c: Double //变量申明
c = 2.9D //赋值

// Byte 定义字节值
var b:byte = 127

// Short 定义短整型值
var number:Short = 32767

// Int 定义Int值
var age: Int = 2017 或者 var age = 2017 （自动推断类型）

// Long 定义长整型
var money: Long = 9999999999L 或者 var money = 99999999999L 

// Float 定义单精度值
var f: Float = 123456789f 或者 var f = 123456789f

// Double 定义双精度值
var d: Double = 66666666.0 或者 var d = 66666666.0

// boolean 定义布尔值
var biu = true/false

// Char 定义字符
var cili = "c"

```

定义重复赋值变量使用`var` 关键字

```kotlin
var a: Int = 32
a = 2
```

#### 延迟初始化属性 （Late-Initialized Properties）

在Kotlin中，声明为具有非空类型的属性必须在构造函数中初始化，但是往往不希望在构造函数中初始化，例如在通过依赖注入或单元测试的设置方法来初始化属性的时候，不能在构造器中提供一个非空的初始化语句，为了处理这种情况，就要在属性上加lateinit关键字来延迟初始化
```
public class MyTest {
    lateinit var subject: TestSubject

    @Before fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method() 
    }
}
```
注：**lateinit**只能够在var类型的属性中，不能用于构造函数，而且属性不能有自定义的getter和setting，这些属性必须是非空类型，并且不能是基本类型。 如果在一个延迟初始化的属性初始化前调用,会导致一个特定异常,调用的时候值还没有初始化.

#### NULL检查机制

Kotlin的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式，字段后加!!像Java一样抛出空异常，另一种字段后加?可不做处理返回值为 null或配合?:做空判断处理

```kotlin
//类型后面加?表示可为空
var age: String? = "23" 
//抛出空指针异常
val ages = age!!.toInt()
//不做处理返回 null
val ages1 = age?.toInt()
//age为空返回-1
val ages2 = age?.toInt() ?: -1
```

#### 类型检测及自动转换

```kotlin
fun getStringLength(obj: Any): Int? {
  if (obj !is String)
    return null
  // 在这个分支中, `obj` 的类型会被自动转换为 `String`
  return obj.length
}
```

#### 基本数据类型

Kotlin 的基本数值类型包括 Byte、Short、Int、Long、Float、Double 等

| 类型     | 位宽度  | 示例   |
| :----- | :--- | ---- |
| Double | 64   | 2.0  |
| Float  | 32   | 2f   |
| Long   | 64   | 3L   |
| Int    | 32   | 3    |
| Short  | 16   | 4    |
| Byte   | 8    | 5    |

可以使用下划线使数字常量更易读

```kotlin
val oneMillion = 1_000_000
val hexBytes = 0xFF_EC_DE_5E
val phoneNumber = 185_5162_3845
```

在 Kotlin 中，三个等号 === 表示比较对象地址，两个 == 表示比较两个值大小。

```kotlin
val a: Int = 10000
//经过了装箱，创建了两个不同的对象
val boxedA: Int? = a
val anotherBoxedA: Int? = a

//虽然经过了装箱，但是值是相等的
println(boxedA === anotherBoxedA) //  false，值相等，对象地址不一样
println(boxedA == anotherBoxedA) // true，值相等
```

类型转换:每种数据类型都有下面的这些方法，可以转化为其它的类型：

```kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```



从kotlin1.3开始，引入了**无符号整型**数据类型，目前还是***实验性***的。

| 类型     | 位宽度  | 范围        | 示例     |
| ------ | ---- | --------- | ------ |
| UByte  | 8    | 0~255     | 221u   |
| UShort | 16   | 0~2^16^-1 | 12u    |
| UInt   | 32   | 0~2^32^-1 | 36u    |
| ULong  | 64   | 0~2^64^-1 | 1284uL |



#### kotlin修饰符及关键字

```kotlin
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation
companion
inline
infix
operator
data
```
与Java不同的是，kotlin默认的访问修饰符是public,而且多了一个internal（模块）修饰符
其他private 和 protected的作用范围和Java一样
- private：只该类可见（包括该类的所有成员）
- protected：同private 加子类可见
- internal：在同一个模块中可见
- public：公共，所有都可见

在kotlin中增加了top-level属性和方法，与class同级

```kotlin
package foo.bar

val prop: String = "top-level-prop"
fun demo() {
    loge("top-level", "top-level-demo()")
}

class Kot {
    fun v() {
        loge("top-level", prop)
        demo()
    }
}

```
上面的prop属性和demo()方法就是顶级属性及方法，在Kot类编译成class文件的时候，会把Top-level的属性和函数创建到以类名+Kt为名的class文件中，如：KotKt.class


顶级属性和方法前面不能用protected访问修饰符


#### Kotlin随机数

kotlin1.3加入多平台random随机数，kotlin.random.Random

```kotlin
Random.nextInt() //Int.MIN_VALUE ~ Int.MAX_VALUE
Random.nextInt(52) //0 ~ 52
Random.nextBoolean() //true or false
Random.nextFloat() //0 ~ 1
Random.nextDouble() //0 ~ 1
Random.nextDouble(5.0) //0 ~ 5.0

//生成一个长度23的随机数组
val nextBytes = Random.nextBytes(23)
for (it in nextBytes) {
	println(it)
}

```

#### 运算符

##### 单目运算符

| 表达式  | 对应函数           |
| ---- | -------------- |
| +a   | a.unaryPlus()  |
| -a   | a.unaryMinus() |
| in   | a.not()        |
| a++  | a.inc()        |
| a--  | a.dec()        |

##### 双目运算符

| 表达式      | 对应函数                    |
| -------- | ----------------------- |
| a+b      | a.plus(b)               |
| a-b      | a.minus(b)              |
| a*b      | a.times(b)              |
| a/b      | a.div(b)                |
| a%b      | a.rem(b) / a.mod(b) 已过时 |
| a..b     | a.rangTo(b)             |
| a in b   | a.contains(b)           |
| a !in b  | !a.contains(b)          |
| a += b   | a.plusAssign(b)         |
| a -= b   | a.minusAssign(b)        |
| a *= b   | a.timesAssign(b)        |
| a /= b   | a.divAssign(b)          |
| a %= b   | a.remAssign(b)          |
| a && b   | a.and(b)                |
| a \|\| b | a.or(b)                 |
