---
title: Android开发者必备工具-常见Android模拟器
date: 2021-09-30 10:18:44
categories: Android
tags: 模拟器
---

### 模拟器的用途介绍

作为一名Android开发者，模拟器是我们调试和测试应用必备的神器。

模拟器主要有以下用途：

第一个是用于**游戏**。游戏玩家可以在他们的电脑上使用模拟器来让一些游戏更容易玩。他们不必依赖设备的电池寿命，借助于键盘、鼠标以及更大的屏幕视野等技巧有助于更好的游戏体验。在大多数情况下，这些小技巧并不违法（在大多数游戏中），所以没有人真正有问题。比较不错的安卓游戏模拟器包括 LDPlayer、BlueStacks、MeMu、KoPlayer 和 Nox等。

第二个最常见的场景是**开发**。 Android 应用和游戏开发人员在发布前需要尽可能在更多的设备上测试应用和游戏。然后现实情况是并没有太多的手机供开发人员测试，幸运的是，Android Studio 附带的模拟器以及其他众多厂商开发的模拟器为我们解决了这个问题。可以通过设置模拟器的分辨率、设备尺寸、API版本等属性来模拟不同场景的运行效果。

目前除了Android Studio自带的模拟器外，没有模拟器可以运行最新版本的 Android。 幸运的是，大多数应用程序和游戏仍然可以在旧版本的 Android 上运行，所以这应该不是什么大问题。 而且现在大多数模拟器已经支持 Android 7.0 Nougat 和 Android 9.0 Pie 运行，这些模拟器制造商也一直对模拟器不断的更新升级，相信以后肯定会对Android新版本支持得更加完善。

另外，从 Windows 11 开始，Windows 将允许使用原生 Android 应用程序。 Windows 11 于 2021 年 10 月 6 日发布，并且应该会在几个月后的某个时间推出 Android 应用程序支持。

下面列举的这些常见的模拟器各有优势，具体选择用什么模拟器根据实际需求和个人喜好来定。
<br></br>

### 1. MuMu（网易MuMu）
---
官方下载链接：[http://mumu.163.com/](http://mumu.163.com/) 或 [https://www.mumuglobal.com/](https://www.mumuglobal.com/)

>  第一个是国内中文网站，第二个是全球英文网站

支持平台：Windows、Mac		

目前最新支持：Android 6.0.1，Android 9.0版本还处于测试阶段

是否收费：免费

![MuMu模拟器](https://img-blog.csdnimg.cn/77fd6c6e895c4ec789eef9d72b4528fd.jpg)

**基本介绍**

网易MuMu目前旗下有2款产品，一款MuMu模拟器，一款MuMu手游助手，都是针对手游玩家开发的安卓模拟器类软件，可在电脑上大屏体验各类游戏与应用。

MuMu模拟器基于传统安卓模拟器引擎、Android6.0内核、x64架构，是一款相对稳定，能够适配市面99%主流手游和应用的安卓模拟器；

MuMu手游助手在MuMu模拟器的基础上，增配网易自研星云引擎、Android7.1内核、x64架构，拓展性更强，更能迎合热门新游的配置要求；

2款产品加持几乎100%覆盖你想体验的热门手游，兼容性超越同类手游安卓模拟器，还可以享受120帧高带来的丝滑感受，免费的海外加速、操作录制、多开、智能键鼠功能满足你不同的游戏需求。

总体来说MuMu模拟器是一款很不错的模拟器，使用起来很流畅，页面做的很好。而且网易目前一直在对它进行维护更新。

**使用MuMu调试Android应用**

默认MuMu不能自动连接ADB，我们需要在CMD窗口中手动执行下面命令来连接

```shell
adb connect 127.0.0.1:7555
```

`7555`是MuMu模拟器默认端口，然后使用`adb devices`就可以看到电脑已经连接的所有设备了，如果要断开模拟器ADB连接，只需要执行下面的命令即可。

```shell
adb disconnect 127.0.0.1:7555
```

<br></br>

### 2. BlueStacks（蓝叠）
---
官方下载链接：[https://www.bluestacks.cn/](https://www.bluestacks.cn/) 或 [https://www.bluestacks.com/](https://www.bluestacks.com/)

支持平台：Windows		

目前最新支持：Android 7.1.2

是否收费：免费
![BlueStacks](https://img-blog.csdnimg.cn/fdf2bfd2f2e347b898c7980031813a8a.jpg)


**基本介绍**

蓝叠安卓模拟器是全球唯一一个拥有核心技术专利的安卓模拟器，并获得高通、英特尔、AMD等行业巨头的投资。
“安卓模拟器中的英特尔”、“安卓模拟器的心脏”，这既是合作伙伴对蓝叠安卓模拟器的描述，也是给予的荣誉称号。 由于具有核心技术优势，以及快速的服务响应，经过近年的快速发展，腾讯、网易、阿里巴巴等都成为蓝叠安卓模拟器重要的合作伙伴。与此同时，蓝叠安卓模拟器相比于其他同类产品具有更加良好的兼容性、稳定性和流畅度，以及更好的游戏体验，因此，蓝叠安卓模拟器在普通玩家中拥有良好的口碑和许多忠实的用户，其中不乏痴迷于安卓模拟器引擎的技术极客粉丝。

**连接ADB调试应用**

和MuMu模拟器一样，蓝叠默认也不会自动建立ADB连接，我们需要找到它对应的端口号进行，然后手动执行连接，一般情况下有2种方法找到对应模拟器的端口号，如上这个参考链接[https://www.cnblogs.com/rogunt/p/13047394.html](https://www.cnblogs.com/rogunt/p/13047394.html) 。这里我用的是第一种方式。

首先在cmd窗口（快捷键：Win + R），输入`regedit`打开注册表编辑器

然后定位到如下目录

`计算机\HKEY_LOCAL_MACHINE\SOFTWARE\BlueStacks_china_gmgr\Guests\Android\Network\0`
![注册表编辑器](https://img-blog.csdnimg.cn/bb5146fc4e644ac2b08b2c9d69d978fe.jpg)

最后找到”InboundRules“的值，打开它可以看到其中有很多端口号，一般第一个的选项就是可以用adb连接的端口号，然后同样执行下面命令连接即可。

```shell
adb connect 127.0.0.1:5555
```

<br></br>
### 3. Nox（夜神）
---
官方下载链接：[https://www.yeshen.com/](https://www.yeshen.com/) 或者 [https://www.bignox.com/](https://www.bignox.com/)

支持平台：Windows、Mac		

目前最新支持：Android 5.1/7.1/9.0

是否收费：免费
![Noxi模拟器](https://img-blog.csdnimg.cn/d66e4323f8964f37a99b88618533b74f.jpg)


**基本介绍**

夜神模拟器进行了全面的优化，无论是游戏还是应用，用起来都会更稳定、更流畅。除此之外更有超凡的端游操作体验，让你快人一步。

- 键鼠操控
  - 一键应用云端键盘，即刻享受端游般的游戏体验。使用键盘鼠标，操作快人一步。
- 极致多开
  - 打开多个模拟器，每个模拟器可独立运行游戏。更有多开同步，轻松实现多账号。
- 操作助手
  - 记录下你的复杂操作，下次执行时只需轻轻一点， 即可解放双手。


夜神模拟器默认会自动连接ADB的，所以不用手动连接，一般模拟器都是支持多开的，即可以同时打开运行模拟器，夜神模拟器也是支持多开的，可以同时创建运行不同的模拟器。
![Nox多开器](https://img-blog.csdnimg.cn/c77d7125fcb141b5a79a1d0ce9b2325f.jpg)
<br></br>

### 4. MEmu（逍遥安卓模拟器）
---
官方下载链接：[http://www.microvirt.com/](http://www.microvirt.com/) 或 [https://www.memuplay.com/](https://www.memuplay.com/)

支持平台：Windows		

目前最新支持：Android 7.1

是否收费：免费
![逍遥模拟器](https://img-blog.csdnimg.cn/55f2b53d8eae4c7eaa59cfb75fa963f4.jpg)

**基本介绍**

- 逍遥模拟器7
  - 全新
    引擎，跑分领先；性能更强，多开挂机更省资源。
- 超清画质
  - 支持OpenGL和DirectX渲染模式；畅享120帧超清电影画质。
- 智能按键
  - 电脑键鼠玩手游，轻松易上手；零延迟电竞级体验，真正端游操控。
- 全面兼容
  - 高配、低配电脑都能玩；适配更多手游应用，稳定更流畅。

**连接ADB调试应用**

和MuMu模拟器一样，默认也不会自动建立ADB连接，我们需要找到它对应的端口号进行，然后手动执行连接，一般情况下有2种方法找到对应模拟器的端口号，如上这个参考链接[https://www.cnblogs.com/rogunt/p/13047394.html](https://www.cnblogs.com/rogunt/p/13047394.html) 。这里我用的是第二种方式。

首先打开任务管理器窗口（快捷键：Ctrl + Shift + Esc；

然后切换到`详细信息`栏目，并找到逍遥模拟器对应的进程PID；
![任务管理器](https://img-blog.csdnimg.cn/64ce5053e45445acb8c55f7ce8cf699c.jpg)
最后在cmd窗口（win + r）中执行下面的命令，`17748`为找到的对应PID。

```shell
netstat -ano|findstr "17748"
```

然后可以看到一些端口号，使用这些端口号执行ADB连接命令就行了，有时候端口号太多，不确定是哪个只能一个一个试了。

![ADB Connect](https://img-blog.csdnimg.cn/d0450bf51cee4f73b2c1103bf0f67aba.jpg)

<br></br>

### 5. LDPlayer（雷电模拟器）
---
官方下载链接：[https://www.ldmnq.com](https://www.ldmnq.com/) 或 [https://www.ldplayer.net](https://www.ldplayer.net/)

支持平台：Windows

目前最新支持：Android 7.1.2

是否收费：免费
![LDPlayer](https://img-blog.csdnimg.cn/fdf6600550164541b89d66cc09cb0abc.jpg)

**基本介绍**

LDPlayer是一款轻量级的安卓模拟器，专注于游戏性能。运行 Android Nougat 7.1，它具有一系列面向游戏玩家的常用功能，包括良好的键盘映射控件、多实例、高 FPS 和图形支持。它借鉴了 Bluestacks 的一些设计，但这并不是一件坏事。这是一个很好的多面手，应该能满足大多数人的需求。

雷电模拟器默认是开启ADB调试的，可以在模拟器设置中开启或关闭ADB本地链接。


<br></br>

### 6. Android Studio 模拟器
---
官方下载链接：[https://developer.android.google.cn/studio?hl=en](https://developer.android.google.cn/studio/releases/emulator?hl=en)

支持平台：Windows、Mac、Linux

目前最新支持：Android 12.0
![Android Studio Emulator](https://img-blog.csdnimg.cn/fd37ac2a31f84f66a2924af967e82033.jpg)

**基本介绍**

Android Studio 是 Android 的默认集成开发环境或 IDE。它附带了一系列工具，可帮助开发人员专门为 Android 制作应用程序和游戏。Android Studio内置的模拟器就是为了更加方便的测试应用程序或游戏。

它的功能非常强大，设置比较复杂，而且需要依赖于Android Studio开发环境，因此它的使用对象一般都是Android开发人员。但它是Google官方打造的模拟器，功能丰富，支持添加自定义启动器和键盘，并模拟任何尺寸或外形的设备，包括手机、电视大屏、穿戴设备、车载设备等，甚至可以模拟可折叠设备、挖孔屏！还有一个其他模拟器不能与之匹敌的特点就是它能一直支持最新的Android版本，可以抢先模拟体验Android最新开发的功能及特性。

![Android Studio Emulators](https://img-blog.csdnimg.cn/e4bd43db84b24144bb66e9f6f8ea9090.jpg)
![New Android Studio Emulator](https://img-blog.csdnimg.cn/b4d3c5a4e0e44092a9a0256a3563a71c.jpg)

另外，Android Studio模拟器可以支持常见的手机CPU架构（ABI：x86、`x86_64`、armeabi、armeabi-v7a、arm64-v8a）。x86 、`x86_64` 在平板和模拟器中用得比较多；armeabi是第5代、第6代的ARM处理器，早期的手机用的比较多；armeabi-v7a是第7代ARM处理器，2011年以后的Android设备基本都使用它；arm64-v8a是第8代64位的ARM处理器，是目前主流的版本。Android Studio模拟器推荐使用x86的CPU架构，运行更加快一点。

可以使用如下ADB命令查看设备的ABI：

```shell
adb shell getprop ro.product.cpu.abi
```

<br></br>

### 7. Genymotion模拟器
---
官方下载链接：[https://www.genymotion.com/download/](https://www.genymotion.com/download/)

支持平台：Windows、Mac、Linux

目前最新支持：Android 10.0

是否收费：个人使用免费
![Genymotion](https://img-blog.csdnimg.cn/dd0e7a77406b4e89b723f97daa792a24.jpg)


**基本介绍**

Genymotion 模拟器也是主要面向Android开发人员。 它的功能非常强大，可以创建不同设备尺寸、分辨率、API版本任意组合的模拟器，支持各种常见的设备尺寸及分辨率，满足日常开发需求。
![Genymotion Emulators](https://img-blog.csdnimg.cn/b3e2f0eaf27d40d596fa0bb2176e8ce0.jpg)


**连接ADB调试应用**

我们可以在Android Studio上安装Genymotion插件，然后可以很方便的像内置模拟器一样在Android Studio开发工具上面调试我们的应用程序及游戏，具体的安装步骤及Genymotion常见的使用问题可以参考[这篇博客](https://www.cnblogs.com/whycxb/p/6850454.html)。


<br></br>

### 8. Phoenix OS（凤凰系统）
---
官方下载链接：[http://www.phoenixos.com/download_x86](http://www.phoenixos.com/download_x86/)

支持平台：Windows、Mac

目前最新支持：Android 7.1

是否收费：免费
![Phoenix OS](https://img-blog.csdnimg.cn/37db7148e664485488e39f5447b43a20.png)


**基本介绍**

Phoenix OS 是适用于 PC 的较新的 Android 模拟器之一，实际上它更像一个操作系统。 像现在的大多数情况一样，它拥有游戏玩家体验。 然而，它也拥有类似桌面的体验，因此它实际上也能很好地提高生产力。 它有 Google Play 服务，虽然更新这些服务有时会有点痛苦。 这意味着您可以在 Google Play 商店中获得所有应用和游戏。 Phoenix OS 支持 Android 5.1和7.1。



### 总结
以上介绍的8个常见模拟器各自有自己独特的优势，有些适合游戏玩家，有些更利于Android开发调试，下面以一张表格整理一下它们的特点及区别。


| 模拟器                                      | 支持系统              | 安卓系统    | 是否收费 | 优势特点           |
| ---------------------------------------- | ----------------- | ------- | ---- | -------------- |
| [MuMu](http://mumu.163.com)              | Windows、Mac       | 6.0、9.0 | 免费   | 稳定快速、网易出品      |
| [蓝叠](https://www.bluestacks.cn/)         | Windows           | 7.1.2   | 免费   | 游戏、开发调试均可      |
| [夜神](https://www.yeshen.com/)            | Windows、Mac       | 7.1、9.0 | 免费   | 稳定流畅、游戏、开发调试均可 |
| [逍遥](http://www.microvirt.com/)          | Windows           | 7.1     | 免费   | 游戏、开发调试均可      |
| [雷电](https://www.ldmnq.com/)             | Windows           | 7.1.2   | 免费   | 游戏、开发调试均可      |
| [AS模拟器](https://developer.android.google.cn/studio/releases/emulator?hl=en) | Windows、Mac、Linux | 支持所有    | 免费   | 官方、适合开发调试      |
| [Genymotion](https://www.genymotion.com/download) | Windows、Mac、Linux | 几乎所有    | 个人免费 | 稳定快速、适合开发调试    |
| [Phoenix OS](http://www.phoenixos.com/download_x86) | Windows、Mac       | 7.1     | 免费   | 类似于操作系统        |


好了，关于常见的Android模拟器就介绍到这里了。Android模拟器远不止这些，有一些模拟器已经不再维护，还有一些通过安装Chrome插件在浏览器上运行Android应用，我只是整理了比较常见且一直在维护更新的模拟器。






