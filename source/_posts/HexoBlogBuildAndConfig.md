---
title: 使用Hexo免费搭建个人博客教程
date: 2021-10-13 14:41:03
categories: 实用工具
tags: 环境搭建
---

[GitCmd]:./HexoBlogBuildAndConfig/GitCmd.jpg	"Git"
[Git]:./HexoBlogBuildAndConfig/Git.jpg
[LivereInstall]:./HexoBlogBuildAndConfig/LivereInstall.jpg
[livere]:./HexoBlogBuildAndConfig/livere.jpg

### 前言

现在各种互联网博客非常的，常见的如CSDN、简书、掘金、博客园等平台，这些博客平台做的都很好，可以直接在上面注册自己的账号写文章，发表的文章在百度、Bing等搜索引擎上也能收到，但缺点是受于平台的各种限制，个人定制化不自由，而且大多数平台都会有各种广告。

于是想到自己搭建一个博客网站，但对大多数人而言，自己购买服务器和域名来搭建博客成本实在太高的了，而且有点浪费，还需要定期维护，这时一种不错的选择就是使用第三方开源托管平台（GitHub、GitLab等）来当作我们的服务器，并使用快速简洁的博客搭建框架-[Hexo](https://hexo.io/zh-cn/)来搭建博客网站就非常容易了，下面就来介绍一下使用hexo搭建博客的步骤及一些配置吧。

### Hexo简介

Hexo是由台湾大佬开发的一款基于Node.js的静态博客框架，依赖少易于安装使用，可以方便的生成静态网页托管在GitHub、Coding、Gitlab等平台上，是搭建博客的首选框架。而且Hexo支持Markdown的所有语法功能来编辑网页内容，编辑的内容生产静态网页速度很快，上百个页面在几秒内瞬间完成渲染。Hexo部署发布也很方便，只需要一条指令即可发布到配置GitHub、 Heroku 等平台。下面就开始一步步的完成搭建吧，大家也可以进入[Hexo官网](https://hexo.io/zh-cn/)查看详细的搭建教程。

### 1. 安装Git

[Git](https://git-scm.com/downloads)是目前世界上最先进的分布式版本控制系统，可以有效、快速的处理各种项目版本管理。在这里就是用来管理我们写的Hexo博客文章，并上传到GitHub等平台的工具。

在Mac和Linux系统上安装Git非常容易。

**Mac**电脑上安装只需要执行下面的命令即可，使用[Homebrew](https://brew.sh/index_zh-cn)来安装

```shell
brew install git
```

**Linux**是一个开源的操作系统，市面上有很多优秀的Linux系统，不同的系统上安装Git执行的命令可能略有不同，具体的查看[Git官网](https://git-scm.com/download/linux)上对应系统的安装命令，这里以Ubuntu系统安装Git为例。

```shell
sudo apt-get install git
```

**Windows** 上安装需要先[下载Git安装包]([https://gitforwindows.org/)，然后点击安装包使用默认的配置一步步安装就可以了。

安装完成之后在命令执行窗口输入`git --version`即可检查是否安装成功，并能看到安装的git版本号。

![GitCmd]

Git有一个专门命令行工具Git Bash，Windows在任何地方只要鼠标右键，然后点击`Git Bash Here`就可以在当前目录路径下打开命令行窗口了。

![Git](./HexoBlogBuildAndConfig/Git.jpg)



### 为文章添加评论功能

Hexo评论模块的配置有很多种方式，这里推荐一个[livere](https://www.livere.com/apply) ，是韩国的一个评论系统，支持免费和收费2中模式，一般个人博客使用免费版本就可以了。

首先需要注册一个账号并登录，然后安装免费版本（City版），

![LivereInstall]

点击现在安装然后填写相关信息就可以看到一个配置js代码了，我们这里只需要使用代码里面的uid就可以了。

![livere]

拿到uid之后，我们需要配置一下我们的themes目录下面的_config.yml文件中的livere信息就好了，我使用的主题是hexo-theme-matery，配置如下：

```shell 
livere:
  enable: true
  uid: 刚刚拿到的uid
```

最后执行`hexo g` 和 `hexo d`命令来成本网页并发布就可以看到每篇文章后面出现来必力评论模块了。

### 安装本地图片插件

在Markdown语法中显示一张图片，需要这张图片的链接地址，如果是本地使用，则直接使用本地图片的绝对路径就可以了，而如果生成静态网页就需要先把图片上传到服务器获取图片链接，市面上有一些免费的图片服务器，你也可以购买自己的服务器专门来管理图片，其实我们可以为Hexo安装一个插件[hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image) 把本地图片直接发布到托管平台，然后生成图片链接。

首先需要安装hexo-asset-image插件：

```shell
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

然后我们新建一篇文章“HexoImageExample”的时候，会在`_posts` 目录下面同时生成一个以“HexoImageExample”命名的文件夹以及一个HexoImageExample.md文件，这个文件夹就是用来存放图片的。

我们打开HexoImageExample.md文件，使用下面的语法来定义图片及显示图片：

```markdown
---
title: HexoImageExample
date: 2021-10-13 14:41:03
---

// 定义图片
[image]: ./HexoImageExample/image.jpg	"ImageTitle"
[image2]: ./HexoImageExample/image2.jpg

// 显示图片
![image]

// 或者直接使用下面这种常规方式显示
![image3]( ./HexoImageExample/image3.jpg)
```

这里需要注意的是图片的路径需要加上`./HexoImageExample`

最后改一下配置文件`_config.yml`里面的`post_asset_folder`属性值，改成`true`。

最后直接执行`hexo g` 和 `hexo d`命令来成本网页并发布就可以了。



>  参考链接：[https://blog.csdn.net/sinat_37781304/article/details/82729029](https://blog.csdn.net/sinat_37781304/article/details/82729029)

