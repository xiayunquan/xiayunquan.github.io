---
title: Android简介及发展历程
tags: Android
abbrlink: 48641
date: 2021-09-22 17:38:45
---

### 简介
**Android**是基于Linux系统的开源操作系统，是由Andy Rubin于2003年在美国加州创建，后被Google于2005年收购。在2008年的时候发布了第一部Android智能手机，随后Android不断发展更新，占据了全球大部分的手机市场。
Android每一个版本都会用一个按照A-Z开头顺序的甜品来命名，但从Android P之后Google改变了这一传统的命名规则，可能是没有那么多让人熟知的甜品代号供使用以及甜品名字并不能让人直观的了解到哪一个甜品有什么特性，于是Google直接采用数字来命令系统，并且加深了logo的颜色，不再使用甜品作为代号。

下面是Android各版本代号、图片及市场占有率
| Platform Version | API      | Version Code                | Logo                                     | Publish Date | 各系统版本比率（2020-07） |
| ---------------- | -------- | --------------------------- | ---------------------------------------- | ------------ | ---------------- |
| 11.0             | 30       | android 11                  | ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730190850516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70) | 2020（Q3）     | <1%              |
| 10.0             | 29       | android 10                  | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfYW5kcm9pZF8xMF8yOS5wbmc?x-oss-process=image/format,png) | 2019         | 8.2%             |
| 9.0              | 28       | pie（红豆派）                    | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfcGllJUU3JUJBJUEyJUU4JUIxJTg2JUU2JUI0JUJFXzkuMF8yOC5wbmc?x-oss-process=image/format,png) | 2018         | 31.3%            |
| 8.0/8.1          | 26/27    | Oreo（奥利奥饼干）                 | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3Rfb3JlbyVFNSVBNSVBNSVFNSU4OCVBOSVFNSVBNSVBNV84LjBfMjYucG5n?x-oss-process=image/format,png) | 2017         | 31.3%            |
| 7.0/7.1          | 24/25    | Nougat（牛轧糖）                 | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3Rfbm91Z2F0JUU3JTg5JTlCJUU4JUJEJUE3JUU3JUIzJTk2XzcuMF8yNC5wbmc?x-oss-process=image/format,png) | 2016         | 13%              |
| 6.0              | 23       | Marshmallow（棉花糖）            | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfbWFyc2htYWxsb3clRTYlQTMlODklRTglOEElQjElRTclQjMlOTZfNi4wXzIzLnBuZw?x-oss-process=image/format,png) | 2015         | 11.2%            |
| 5.0/5.1          | 21/22    | Lollipop（棒棒糖）               | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfbG9sbGlwb3AlRTYlQTMlOTIlRTYlQTMlOTIlRTclQjMlOTZfNS4wXzIxLnBuZw?x-oss-process=image/format,png) | 2014         | 9.2%             |
| 4.4              | 19/20    | Kitkat（奇巧）                  | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3Rfa2l0a2F0JUU1JUE1JTg3JUU1JUI3JUE3XzQuNF8xOS5wbmc?x-oss-process=image/format,png) | 2013         | 4%               |
| 4.1/4.2/4.3      | 16/17/18 | Jelly_Bean（果冻豆）             | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfamVsbHlfYmVhbiVFNiU5RSU5QyVFNSU4NiVCQiVFOCVCMSU4Nl80LjFfMTYucG5n?x-oss-process=image/format,png) | 2012         | 1.7%             |
| 4.0.x            | 14/15    | Ice_Cream_Sandwich （冰淇淋三明治） | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cuY25ibG9ncy5jb20vaW1hZ2VzL2NuYmxvZ3NfY29tL3h5cTYvMTU1NjYzMi90X2ljZV9jcmVhbV9zYW5kd2ljaF8lZTUlODYlYjAlZTYlYjclODclZTYlYjclOGIlZTQlYjglODklZTYlOTglOGUlZTYlYjIlYmJfNC4wXzE0LnBuZw?x-oss-process=image/format,png) | 2011         | 0.2%             |
| 3.0/3.1/3.2      | 11/12/13 | Honeycomb（蜂巢）               | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfaG9uZXlvb21iJUU4JTlDJTgyJUU4JTlDJTlDJUU1JTg2JUIwJUU2JUI3JTg3JUU2JUI3JThCXzMuMF8xMS5wbmc?x-oss-process=image/format,png) | 2011         | N/A              |
| 2.3.x            | 9/10     | Gingerbread  （姜饼）           | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfZ2luZ2VyYnJlYWQlRTUlQTclOUMlRTklQTUlQkNfMi4zXzkucG5n?x-oss-process=image/format,png) | 2010         | N/A              |
| 2.2.x            | 8        | Froyo （冻酸奶）                 | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfZnJveW8lRTUlODYlQkIlRTklODUlQjglRTUlQTUlQjZfMi4yXzgucG5n?x-oss-process=image/format,png) | 2010         | N/A              |
| 2.0/2.1          | 5/6/7    | Eclair （泡芙）                 | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfZWNsYXRyJUU2JUIzJUExJUU4JThBJTk5XzIuMF81LnBuZw?x-oss-process=image/format,png) | 2009         | N/A              |
| 1.6              | 4        | Donut （甜甜圈）                 | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfZG9udXQlRTclOTQlOUMlRTclOTQlOUMlRTUlOUMlODhfMS42XzQucG5n?x-oss-process=image/format,png) | 2009         | N/A              |
| 1.5              | 3        | Cupcake （纸杯蛋糕）              | ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL3RfY3VwY2FrZSVFNyVCQSVCOCVFNiU5RCVBRiVFOCU5QiU4QiVFNyVCMyU5NV8xLjVfMy5wbmc?x-oss-process=image/format,png) | 2009         | N/A              |

下面是截止到2020年7月份各版本市场占有率，最新数据可以在[官网](https://developer.android.google.cn/about/dashboards/)上查询，或者在Android Studio里面新建一个项目，当选择支持最低的SDK（Minimum SDK）版本的时候，就可以看到当前选择的SDK版本及以上的版本在市场上面的占有率。从图中可以看到Android每个系统版本都有一定的占有率，这就给手机应用开发者针对不同版本的适配带来很多麻烦；同时可以看出较新的系统版本占有率相当高，这就要求开发者尽早的针对新版本进行学习和适配，让我们的应用支持新的系统版本带给我们的新功能和特性。
![系统版本市场占有率](https://img-blog.csdnimg.cn/20200730193259154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)

### Android平台架构
Android 是一种基于 Linux 的开放源代码软件栈，为各类设备和机型而创建。
下图所示为 Android 平台的主要组件。 [官网地址](https://developer.android.google.cn/guide/platform/)
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMuY25ibG9ncy5jb20vY25ibG9nc19jb20veHlxNi8xNTU2NjMyL29fYW5kcm9pZC1zdGFja18yeC5wbmc?x-oss-process=image/format,png)
#### Linux内核层
Android平台的基础是Linux内核，例如Android Runtime（ART）依靠Linux内核来执行底层功能，如线程和底层内层管理。使用 Linux 内核可让 Android 利用主要安全功能，并且允许设备制造商为著名的内核开发硬件驱动程序。如蓝牙、相机、WiFi等驱动


#### 硬件抽象层（HAL） 
硬件抽象层 (HAL) 提供标准界面，向更高级别的 Java API 框架显示设备硬件功能。HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个界面，例如相机或蓝牙模块。当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块。 

#### Android Runtime
 对于运行 Android 5.0（API 级别 21）或更高版本的设备，每个应用都在其自己的进程中运行，并且有其自己的 Android Runtime (ART) 实例。ART 编写为通过执行 DEX 文件在低内存设备上运行多个虚拟机，DEX 文件是一种专为 Android 设计的字节码格式，经过优化，使用的内存很少。编译工具链（例如 Jack）将 Java 源代码编译为 DEX 字节码，使其可在 Android 平台上运行。

ART 的部分主要功能包括：
- 预先 (AOT) 和即时 (JIT) 编译    
- 优化的垃圾回收 (GC)
- 在 Android 9（API 级别 28）及更高版本的系统中，支持将应用软件包中的 Dalvik Executable 格式 (DEX) 文件转换为更紧凑的机器代码。
- 更好的调试支持，包括专用采样分析器、详细的诊断异常和崩溃报告，并且能够设置观察点以监控特定字段

在 Android 版本 5.0（API 级别 21）之前，Dalvik 是 Android Runtime。如果您的应用在 ART 上运行效果很好，那么它应该也可在 Dalvik 上运行，但反过来不一定。

Android 还包含一套核心运行时库，可提供 Java API 框架所使用的 Java 编程语言中的大部分功能，包括一些 Java 8 语言功能。 

#### 原生C/C++库
 许多核心 Android 系统组件和服务（例如 ART 和 HAL）构建自原生代码，需要以 C 和 C++ 编写的原生库。Android 平台提供 Java 框架 API 以向应用显示其中部分原生库的功能。例如，您可以通过 Android 框架的 Java OpenGL API 访问 OpenGL ES，以支持在应用中绘制和操作 2D 和 3D 图形。

如果开发的是需要 C 或 C++ 代码的应用，可以使用 Android NDK 直接从原生代码访问某些原生平台库。 

#### Java API框架
我们可通过以 Java 语言编写的 API 使用 Android OS 的整个功能集。这些 API 形成创建 Android 应用所需的构建块，它们可简化核心模块化系统组件和服务的重复使用，包括以下组件和服务： 

- 丰富、可扩展的视图系统，可用以构建应用的 UI，包括列表、网格、文本框、按钮甚至可嵌入的网络浏览器
- 资源管理器，用于访问非代码资源，例如本地化的字符串、图形和布局文件
- 通知管理器，可让所有应用在状态栏中显示自定义提醒
- Activity 管理器，用于管理应用的生命周期，提供常见的导航返回栈
- 内容提供程序，可让应用访问其他应用（例如“联系人”应用）中的数据或者共享其自己的数据

开发者可以完全访问 Android 系统应用使用的框架 API。 无论系统内置或者我们自己编写的App，都需要使用到这层，比如我们想弄来电黑名单，自动挂断电话，我们就需要用到电话管理(TelephonyManager) 通过该层我们就可以很轻松的实现挂断操作，而不需要关心底层实现。

#### 系统应用层
每个 Android手机默认都会有一套用于电子邮件、短信、日历、互联网浏览和联系人等核心应用。这些应用与用户可以选择安装的应用一样，没有特殊状态。但一般没有root权限不能卸载这些系统应用。
我们自己开发的APP也是属于这一层，我们可以在自己的应用中使用一些系统应用的主要功能。例如我们的应用需要发短信，我们无需自己构建该功能，而是调用已安装的短信应用向指定的接收者发送消息。 

作为普通的应用层开发者，我们一般只会与应用层和Java API系统框架能打交道；而底层开发者还需要涉及到原生C/C++库层进行NDK开发。