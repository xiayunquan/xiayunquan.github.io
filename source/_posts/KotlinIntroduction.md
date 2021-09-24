---
title: Kotlin简介及配置
tags: Kotlin
abbrlink: 36009
date: 2021-09-23 17:18:38
---

### 发展历程

![](https://img-blog.csdnimg.cn/2021050611440858.jpg)

**Kotlin**是一种运行在java虚拟机上的静态类型编程语言，由**JetBrains**公司设计并开源。

在2011年，JetBrains就公布了Kotlin的第一个版本；

并在2012年将其开源，2013年Android Studio也加入了对Kotlin的支持；

2016年Kotlin发布了1.0正式版；

2017年Google宣布Kotlin正式成为Android一级开发语言；

2019年的时候，Google在I/O大会上宣布Kotlin成为Android开发的第一语言，Android官网文档的代码已优先显示Kotlin版本，官方的视频教程以及Google的一些开源项目，也改用了Kotlin来实现。

到2021年，Kotlin不知不觉已经迈过了10年历程，现在 Kotlin语言在Android领域已经相当完善，目前Google play上面排名前1000的应用中，已经有超过80%的App使用了kotlin语言，而且这个比例还在不断上升。


### Kotlin工作原理

在Kotlin语言出来之前， Android开发主要使用Java语言进行开发，Java语言的运行机制是将Java源代码编译成特殊的class文件，然后通过Java虚拟机（JVM）将class文件解释成计算机可识别的二进制文件再执行。类似于这种机制，JetBrains设计了一门新的语言—Kotlin，Kotlin源文件通过Kotlin编译器编译出同样的class文件，然后自然就可以运行在JVM了。随着Kotlin不断迭代更新完善，Kotlin编译器可以编译出更多的语言来支持多平台，比如JavaScript、Native等。

![](https://img-blog.csdnimg.cn/img_convert/4178bac4ec20519074a9a3862af417b8.png)

### 使用kotlin进行Android开发

![](https://img-blog.csdnimg.cn/2021050611235519.png)



kotlin非常适合android开发，在2017 Google IO大会上，Google宣布kotlin成为android开发一级语言。使用kotlin进行开发主要有以下优势：

- **兼容性：**kotlin与jdk6完全兼容，保障了kotlin应用程序在旧的android设备上运行没有问题，kotlin工具在android studio上完全支持，并且兼容android构建系统


- **性能：**kotlin与java有相似的字节码结构，运行速度与java类似，kotlin对内联函数的支持，使用lambda表达式的代码通常会比java代码运行的更快


- **互操作性：**kotlin可与java进行100%互操作，Kotlin可以直接调用使用Java编写的代码，允许在kotlin应用中使用所有现有的android库

- **占用：**Kotlin的语法更加简洁，对于同样的功能，使用Kotlin开发的代码量

  可能会比使用Java开发的减少50% 甚至更多，kotlin具有非常紧凑的运行时库，可以通过Proguard进一步减小，在实际应用中，apk体积会减小很多

- **编译时长：**kotlin支持高效的增量编译

### 使用kotlin进行服务端开发

Kotlin ⾮常适合开发服务器端应⽤程序， 允许编写简明且表现⼒强的代码，  同时保持与现有基于 Java 的技术栈的完全兼容性以及平滑的学习曲线

- **表现力**：Kotlin 的革新式语言功能，例如支持[类型安全的构建器](https://www.kotlincn.net/docs/reference/type-safe-builders.html)和[委托属性](https://www.kotlincn.net/docs/reference/delegated-properties.html)，有助于构建强大而易于使用的抽象。
- **可伸缩性**：Kotlin 对[协程](https://www.kotlincn.net/docs/reference/coroutines.html)的支持有助于构建服务器端应用程序，伸缩到适度的硬件要求以应对大量的客户端。
- **互操作性**：Kotlin 与所有基于 Java 的框架完全兼容，可以让你保持熟悉的技术栈，同时获得更现代化语言的优势。
- **迁移**：Kotlin 支持大型代码库从 Java 到 Kotlin 逐步迁移。你可以开始用 Kotlin 编写新代码，同时系统中较旧部分继续用 Java。
- **工具**：除了很棒的 IDE 支持之外，Kotlin 还为 IntelliJ IDEA Ultimate 的插件提供了框架特定的工具（例如Spring）。
- **学习曲线**：对于 Java 开发人员，Kotlin 入门很容易。包含在 Kotlin 插件中的自动 Java 到 Kotlin 的转换器有助于迈出第一步。

### kotlin对JavaScript平台支持

Kotlin 提供了 JavaScript 作为⽬标平台的能⼒。 它通过将 Kotlin 转换为 JavaScript 来构建web应用程序。


### Android Studio配置Kotlin
随着Kotlin的快速发展，新版的Android Studio早已内置了Kotlin插件， 我们只需要升级Android Studio，然后新建一个Kotlin项目就可以进行Kotlin开发了，但如果还是使用Android Studio 3.0 版本以下的，需要首先安装Kotlin插件。 settings -> plugins -> 搜索kotlin 进行安装，

插件安装好了之后需要如下的配置（新版的Android Studio新建Kotlin项目之后自动给我们配置好了）

#### 1.project目录下的build.gradle文件中添加kotlin的classpath.
```kotlin
buildscript {

    ext.kotlin_version = "1.3.41"
    
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.2'
        //添加kotlin的依赖
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:${kotlin_version}"
    
    }
}
```

#### 2.然后在module目录下面的build.gradle文件中添加依赖

```kotlin
apply plugin: 'kotlin-android'

android {
    ...
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib:${kotlin_version}"
}
```

[kotlin官网地址](http://kotlinlang.org/)
[kotlin中文学习网址](https://www.kotlincn.net/docs/reference/android-overview.html)
