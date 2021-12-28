---
title: Kotlin集合与数组
date: 2021-09-24 17:12:24
categories: Kotlin
tags: Kotlin语法
---

#### List

```kotlin
val items = listOf("apple", "banana", "kiwi")
val sets = setOf(3, 5, 9)
//arraylist
var arrayList = ArrayList<String>()
var arrayList2 = arrayListOf(2, 3, 4)
//mutableList
var mutableList = mutableListOf<String>()
//空list
emptyList<String>()
```

#### Map

```kotlin
var map = mapOf<String,String>()
//HashMap
var hashMap = hashMapOf<Int,String>()
var hashMap2 = HashMap<String,String>()
//MutableMap
val mutableMapOf = mutableMapOf<String, String>()
//LinkedHashMap
val linkedHashMap = LinkedHashMap<String, String>()
val linkedMap = linkedMapOf("key1" to 3, "key2" to 5)

//示例
val maps = mapOf("12" to "label12", "13" to "label13")
for ((key,value) in maps) {
	println("key=$key, value=$value")
	println(maps[key])
}

```
#### Array
##### 数组定义
```kotlin
// 指定数组长度
var a1 = arrayOfNulls<Int>(4)

// 使用装箱操作
var a2: Array<String> = arrayOf("何晓明", "张一嗨", "大桥急多") 
或 
var a2 = arrayOf("何晓明", "张一嗨", "大桥急多")
var a3 = intArrayOf(1,3,5) , 还有floatArrayOf() 等

// 空数组
var a4 = emptyArray<Float>()
```
数组用类 Array 实现，并且还有一个 size 属性及 get 和 set 方法,数组有两种创建方式

```kotlin
//[1,2,3]
val a = arrayOf(1, 2, 3)
//[0,2,4]
val b = Array(3, { i -> (i * 2) }) 或
val b = Array(3) { i -> (i * 2) } 
```

除了类Array，还有ByteArray, ShortArray, IntArray ... ，用来表示各个类型的数组，省去了装箱操作，因此效率更高


##### 访问数组元素
```kotlin
var a = intArrayOf(1,2,3)

print(a[2]) 
或者 
print(a.get(2))
```
##### 修改元素内容
```kotlin
a[4] = 123 
或者
a.set(4,123)
```

##### 遍历数组
```kotlin
for (i in a) {
    println(i)
}
```

##### 遍历数组下标
```kotlin
for (j in a.indices) {
    println(j)
}
```