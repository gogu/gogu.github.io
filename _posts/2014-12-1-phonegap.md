---
layout: post
title:  "Phonegap 战记：初探 && 在线 build"
date:   2014-12-1 23:47:12
categories: phonegap build javascript 
---

JavaScript 现在真是可以做太多事情了，尤其有了 node，服务端、非关系数据库、桌面软件、命令行脚本、工程构建工具甚至嵌入式程序都能让 js 挑大梁，而且并非实验性质，而是真的有适合她发挥的空间。做个跨平台 App 很寻常啦，刚看了 MSDN 的入门还把 js 放在了第一栏！

PhoneGap 就是允许用 HTML + CSS + JavaScript （常规的 web 前端技术）调用移动平台自有 API （比如摄像头、通讯簿...）来构建应用的开发框架。最近一段时间的工作都要围绕这个框架来做原有 web 项目针对 windows phone 平台的移植和重构，最终目的是通过应用商店审核上线。

那么首先来创建一个 demo 并在手机上跑跑看吧！

#### 安装 PhoneGap

PhoneGap 官方提供 npm 方式获取，所以先安装 node 吧。

有了 node 之后，下面的内容和 [官网](http://phonegap.com/install/) 一样

    // 当然 windows 就不用 sudo 了
    $ sudo npm install phonegap -g

#### 然后看看官方 demo 吧

    $ phonegap create my-app
    $ cd my-app
    $ phonegap run android

phonegap 会自动为你加上 android 平台的部署依赖，构建 App 然后运行。不过虽然官方是这么写的，也并不是说在自己的机器上就能好好执行，还需要对不用平台做一些准备的，下次再来研究这些。我们就这么直接在线构建出打包应用，在机器上玩玩吧！

#### Adobe 的在线构建服务

在线构建太省事！在 [https://build.phonegap.com](https://build.phonegap.com) 注册一个账号，免费账户可以 build 一个应用。

两种上传源码的方式，其一是把刚才的 my-app 打包成 .zip 格式压缩包上传；更推荐的是使用远程 git 库（当然如果你用 git 的方式工作），对时常迭代功能的应用更方便！

上传之后，简单 build ！同时出三个平台版本是不是湿了！

如果准备好了 keyfile、publisher ID 之类认证，直接写上就编译出可用于发布的应用。

扫码可以下载应用安装看看效果！恩，这个简单的 demo 只是会显示出 device ready 而已，这就是第一步。

// todo 补充文章

