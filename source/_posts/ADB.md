---
title: 常用ADB命令
date: 2021-09-24 15:31:06
categories: 实用工具
tags: 
  - ADB
  - 命令
---

### ADB(Android Debug Bridge)

**ADB**是Android开发者和测试人员必不可少的工具。熟悉ADB命令将会给日常开发带来很多帮助，下面是我整理的一些平时使用比较多的ADB命令，当然ADB命令远不止这些，更多的命令可以参考`Zhuang Ma`的[GitHub项目](https://github.com/mzlogin/awesome-adb) 。

在介绍ADB命令之前先让我们打开命令运行窗口：

- Windows：win + R 打开运行窗口，然后输入`cmd`回车即可。
- Mac：cmd⌘ + space打开spotlight，然后输入`Terminal`回车即可。
- Android Studio：Terminal窗口。




下面正式开始ADB命令介绍：

#### 查看adb的路径

``` shell
where adb
```

如：D:\project\sdk\platform-tools\adb.exe，adb位于SDK安装路径的platform-tools目录下面。



####查看adb版本信息

``` shell
adb version
```

直接输入`adb`或者`adb help`也可以打印adb的版本和路径信息。



####列出所有连接的Android设备或模拟器

``` shell
adb devices
```

如: 

List of devices attached
b7d5caee            device
127.0.0.1:7555   device
emulator-5554   device

上面是我电脑连接的3个设备，第一个是手机设备；第二个是网易mumu模拟器；第三个是Android Studio自带模拟器。



####连接设备

``` shell
adb connect HOST[:PORT]
```

PORT不写默认为5555.

如：连接网易mumu模拟器

`adb connect 127.0.0.1:7555` 



#### 断开连接

``` shell
adb disconnect [HOST[:PORT]]
```

PORT不写默认为5555，如果HOST和PORT都不写则表示断开所有连接

如：断开连接网易mumu模拟器

`adb disconnect 127.0.0.1:7555` 

> 注: `connect` 和 `disconnect`只能连接和断开TCP/IP设备,  对USB连接设备无效。



####将电脑上的文件复制到设备中

``` shell
adb push <电脑上的文件路径>  <设备里的目录>
```

 如：将桌面上一张图片拷贝到设备内存卡的picture目录中。

adb  push  C:\Users\Administrator\Desktop\test.gif  /sdcard/picture

> - 如果picture目录不存在，但上级目录存在,则会在拷贝到根目录，但最终的文件会以这个目录命名
> - 如果picture目录不存在,且上级目录也不存在，则会直接报错找不到目录
> - 如果目录中已存在同名文件，则会覆盖原来的文件

如果同时连接了多个设备，则需要指定具体的设备，不然会提示`more than one device/emulator` 错误，`b7d5caee`为上面`adb devices` 得到的设备名称：

adb -s b7d5caee  push  C:\Users\Administrator\Desktop\test.gif  /sdcard/picture



####将设备中的文件复制到电脑中

``` shell
adb pull <设备里的文件路径> [电脑上的目录]
```

如：将设备中的test.txt拷贝到电脑桌面上

adb  pull   /storage/sdcard0/head/test.txt  C:\Users\Administrator\Desktop



####安装应用

``` shell
adb install [-rtdg] apk_path
```

-r: replace existing application 覆盖安装

-t: allow test packages 

-d: allow version code downgrade 降级安装,仅debuggable包

-g: grant all runtime permissions 允许所有运行时权限



####卸载运用

``` shell
adb uninstall [-k] packagename
```

-k: keep the data and cache directries 表示卸载应用但保留数据和缓存目录。



####启动ADB服务server

``` shell
adb start-server
```



####停止ADB服务server

``` shell
adb kill-server
```



#### 重启设备

``` shell
adb  reboot 
```



#### 打印日志

``` shell
adb logcat *:(V|D|I|W|E|F|S) 
```

V: Verbose

D: Debug

I: Info

W: Warn

E: Error

F: Fatal

S: Silent



**S级别最高，什么日志都不会打印 **

`adb logcat *:S`

**打印tag为haha的所有debug级别日志，其他日志不打印**

`adb logcat haha:d *:s` 

**将控制台打印的日志保存到文件中**

`adb logcat  -> D:/log.txt`

**清空控制台打印的日志**

`adb logcat -c`



####shell 的使用

Android系统是基于Linux系统开发的，所以支持常见的Linux的命令，我们连接设备之后就可以使用`adb shell` 来执行这些命令。

##### 显示设备上所有的应用包名
``` shell
adb shell pm list package        
```

##### 列出系统应用
``` shell
adb shell pm list package -s    
```
##### 列出第三方应用
``` shell
adb shell pm list package -3    
```
##### 列出包名及存放位置
``` shell
adb shell pm list package -f     
```
##### 列出包名及安装来源
``` shell
adb shell pm list package -i     
```
##### 显示设备上所有的包含name关键字的应用包名
``` shell
adb shell pm list packages name 
```
##### 过滤包含com的包的详细信息
``` shell
adb shell pm list package -3 -f  -i  com   
```
##### 列出安装包在设备中的路径
``` shell
adb shell pm path <applicationId> 
```
如：

adb shell pm path com.example.app

打印：package:/data/app/com.example.app/base.apk

##### 清除应用数据与缓存

``` shell
adb shell pm clear <applicationId> 
```
可能会遇到没有权限的情况`java.lang.SecurityException: PID 14299 does not have permission android.permission.CLEAR_APP_USER_DATA to clear data of package XXX` ,这种情况需要在**开发者选项**里面关闭权限限制，每个手机的位置和描述可能不一样，如

![](https://img-blog.csdnimg.cn/20210422210029264.png)



![](https://img-blog.csdnimg.cn/20210422210405733.png)

##### 查看所有的危险权限列表

``` shell
adb shell pm list permissions -g -d  
```
##### monkey测试
``` shell
adb shell monkey -p package count  
```
package：进行monkey测试的应用包名

count：测试次数

如：

adb shell monkey -p com.example.test 200



##### 查看设备前台activity

``` shell
adb shell dumpsys activity activities  
```
##### 查询设备型号
``` shell
adb shell getprop ro.product.model 
```
##### 查询设备品牌
``` shell
adb shell getprop ro.product.brand 
```
##### 查询系统版本号
``` shell
adb shell getprop ro.build.version.release 
```
##### 查看电池状态
``` shell
adb shell dumpsys battery 
```
##### 获取设备屏幕分辨率
``` shell
adb shell wm size  
```
##### 获取设备屏幕密度
``` shell
adb shell wm density 
```
##### 查询设备IP地址
``` shell
adb shell ifconfig  
```
##### 查询设备CPU信息
``` shell
adb shell cat /proc/cpuinfo 
```
##### 查询设备CPU处理器架构

```shell
adb shell getprop ro.product.cpu.abi
```

##### 查询内存信息

``` shell
adb shell cat /proc/meminfo 
​``` shell
##### 启动Activity
​``` shell
adb shell am start [-S] applicationId/启动入口Activity的全路径
```
 -S：表示重启当前应用

如：

adb shell am start -S com.example.test/com.example.test.MainActivity

##### 测量启动时间

``` shell
adb shell am start -W applicationId/启动入口Activity的全路径
```

如：

adb shell am start -W com.example.test/com.example.test.MainActivity

运行成功后将会返回3个测量时间:

- **ThisTime**：一般和 TotalTime 时间一样。除非在应用启动时开了一个透明的 Activity 预先处理一些事再显示出主 Activity，这样将比 TotalTime 小。
- **TotalTime**：应用的启动时间。包含创建进程 + Application 初始化 + Activity 初始化到界面显示。
- **WaitTime**：一般比 TotalTime 大点，包含系统影响的耗时。


TotalTime就是从点击手机桌面应用图标开始到看见第一个页面所消耗的时间。




##### 模拟点按键操作

``` shell
adb shell input keyevent keycode(3||24 音量控制键-|25 音量控制键+|26 电源键...)
```

如：模拟点击电源键

adb shell input keyevent 26

keycode常见值有：

3    HOME键

4    返回键

24  音量控制键+

25  音量控制键-

26  电源键



上面就是整理的一些常用的ADB命令了, 下面再介绍2个与Android应用签名相关的实用命令:

### 查看自己的keystore的别名及相关信息

``` shell
keytool -list  -v -keystore xxxx.keystore -storepass 密码
```



### Android签名命令

``` shell
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore 密钥路径  -storepass  keystore文件密码  待签名的APK路径  密钥别名
```

运行成功之后apk还是原文件

例如：

jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore test.jks -storepass test_keystore_pass C:\Users\Administrator\Desktop\test   test_alias