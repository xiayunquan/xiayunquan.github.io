---
title: Android开发环境搭建
date: 2021-09-24 17:23:00
categories: Android
tags: 环境搭建
---

### Android开发环境搭建
android开发环境搭建需要如下配置:   

- JDK(Java Development Kit)、   
- Android Studio工具   

下面就来一步步实现：

### 安装JDK ###
Android系统底层内核是基于Linux系统开发的，但是API框架是使用Java语言编写的，Java提供了运行环境、java工具以及基础的类库，所以JDK在Android开发中必不可少。下面的图可以看出每一个项目中必须要有JDK和SDK。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191003112404114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
###### 下载JDK
Java语言是SUN公司开发的，而SUN公司在2009年时被甲骨文公司（Oracle）以74亿美元收购了，所以现在Java属于Oracle了，我们可以在[Oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/index.html)下载最新的Java版本，Java有ME、SE、EE等不同的版本服务不同的场景，我们下载Java SE版本即可：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191003113822504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)点击DOWNLOAD根据自己的电脑系统下载对应平台的安装包
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191003114311660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
###### 安装JDK
下载完成之后，直接双击安装即可，默认安装在C:\Program Files\Java\jdk1.8.0_151\目录下，可以自己修改安装路径，这里我修改为D:\jdk1.8.0_151.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191003115323263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
##### 配置环境变量
直接点下一步安装完成，然后需要设置环境变量，我们就可以使用命令行工具来编译和执行ava程序。
**Windows系统设置环境变量：**
控制面板 -> 系统 -> 高级系统设置  ->  高级 -> 环境变量 -> 系统变量

- 新建系统变量：变量名：`JAVA_HOME`；变量值：`D:\jdk1.8.0_151`   
- 编辑Path变量：在末尾添加`%JAVA_HOME%\bin;%JAVA_HOME%\jre;`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191003144500581.gif)
然后开始 -> 运行 -> 输入cmd(Windows + R -> 输入cmd)打开命令行窗口，输入
`java -version` 显示如下图则说明环境变量配置正确
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191003150730767.png)
**Mac平台配置环境变量：**
下载Mac系统的jdk，点击安装，然后打开终端（cmd + 空格输入terminal）
打开配置文件.bash_profile（`open -e .bash_profile`），如果文件不存在则使用`touch .bash_profile`命令创建文件。
输入如下配置然后保存关闭文件：
```
JAVA_HOME=/Library/Java/JavaVirtualMachinesjdk1.8.0_151.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$JAVA_HOME/jre:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPATH
```
在终端执行`source .bash_profile`命令使配置文件生效
可以使用`echo $JAVA_HOME`查看刚刚配置的环境变量
使用`java -version`查看jdk版本号，如果能打印出下面信息说明环境变量配置成功
```
java version '1.8.0_151'
```
### 安装Android Studio ###
我们可以在[Android开发者官网](https://developer.android.google.cn/studio?hl=zh_cn)下载最新的Android Studio稳定版本3.5.1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009115336294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
然后直接点击安装即可，这里我选择安装到D:\Android Studio目录，安装完成之后运行Android Studio。
![Android Studio安装流程](https://img-blog.csdnimg.cn/20191009145844660.gif)
首次打开的页面是下面这样子的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009144131619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
我们现在还没有下载Android开发必须的SDK，第一次打开Android Studio时系统会检测我们本地是否有下载SDK，如果没有则会提示我们下载：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009144433509.gif)等待下载完成自动解压之后就可以在设置页面看到我们已经下载SDK版本及SDK本地路径。我们还可以在这里下载需要的其他SDK版本
![SDK](https://img-blog.csdnimg.cn/20191009145506778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
有些时候在公司上外网需要设置代理则会出现下面的错误：
![SDK代理设置](https://img-blog.csdnimg.cn/20191009144611258.png)
则我们需要打开设置（Configure -> Settings -> Appearance -> System Settings -> HTTP Proxy）设置代理再下载SDK即可
![proxy](https://img-blog.csdnimg.cn/20191009145317752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
### 开启Android世界的大门
到此我们所有的Android开发环境就安装完成了。下面就可以新建一个项目了，我们选择Start a new Android Studio project创建一个新的工程

> 我们还可以选择`Open an existing Android Studio project`打开一个本地已经存在的项目；
> 也可以点击`Check out project from Version Control`从远程版本管理库（SVN、Git等）clone一个项目；
> 或者点击`Import project（Gradle，Eclipse ADT，etc）`导入Eclipse等项目

![new project](https://img-blog.csdnimg.cn/20191009150345133.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009151906808.gif)上图中可以看到有很多场景

 - Phone And Tablet  手机和平板应用
 - Wear OS                穿戴设备智能手表
 - TV                          智能电视应用
 - Android Auto          车载设备应用
 - Android Things       嵌入式设备应用

针对不同场景选择不同的模板，可以看到Google给我们提供了很多模板，这里我们选择新建一个Empty activity
然后输入项目名、包名、以及项目路径。选择开发语言，Android Studio 3.2以上的默认选择kotlin语言，我们根据自己需要选择java或kotlin进行开发。
之后再选择我们的应用支持的最低手机系统，可以看到google最低支持到Android 4.0，那么4.0以下的手机是使用不了我们的应用的，但是现在基本上也不会有4.0以下的手机了。
最后直接点Finish等待项目配置编译成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009155703735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)下面就用USB连接我们的开发手机，安装项目到手机上。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009162306240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)上图中可以看到我已经连接上了我的红米4手机，点击右边的绿色三角形图标即可运行。
新手可能发现用USB连接手机之后，上面并没有出现连接的设备，是因为还需要在手机上打开开发者选项中的USB调试开关，很多手机默认设置里面是隐藏开发者选项的，我们通过关于手机 -> 连按5次Android版本号（有的是MIUI版本号、Flyme版本号等，不同手机系统可能不一致）；然后可以在更多设置（有的在辅助功能等，不同手机系统可能位置不同）中找到开发者选项并打开USB调试和USB安装开关。（注：小米手机USB安装默认是打不开的，需要插入SIM卡才能打开，这也是一个坑啊）
下面是我的Redmi Note 4X示例，我已经打开了USB调试并在录制视频，所以不能关掉，这里只是看看操作流程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009163516651.gif)
由于我的测试机没有装SIM卡，所以不能直接点绿色三角形进行安装，我们可以先build apk打好包拷贝到手机里面安装运行，或者使用模拟器进行安装。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009165650998.gif)最终在手机上的运行结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009170058127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)
**小技巧：** 我们可以使用Android Studio工具对手机进行截图和录屏
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191009170411355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG54aWE=,size_16,color_FFFFFF,t_70)