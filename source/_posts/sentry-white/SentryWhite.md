---
title: 哨兵小白-错误日志高效上报的白嫖方案
description: 利用一系列服务打造基本免费且高效的错误日志上报系统
keywords: Sentry, sentry-white, ServerLess, 哨兵小白, 错误上报, 白嫖
top_img: /images/sentry-white/mind-map.png
cover: /images/sentry-white/mind-map.png
tags:
  - Sentry
  - sentry-white
categories:
  - sentry-white
date: 2022-01-11 15:30:27
updated: 2022-01-11 15:30:27
swiper_index:
swiper_desc:
swiper_cover:
---
## 使用效果

在项目中部署了`sentry`之后，当线上项目报错时，错误信息会第一时间发送至`钉钉群`内，并且会`@项目负责人`。
项目负责人的电脑与手机均会出现钉钉醒目提醒。

* PC端提示
![PC端提醒](/images/sentry-white/SentryWhite01.png)

* 手机端钉钉内提示
![钉钉提醒](/images/sentry-white/SentryWhite02.jpg)

* 手机通知栏提示
![状态栏提醒](/images/sentry-white/SentryWhite04.jpg)

## 部署步骤

### 必备环境

* Golang 环境，用以打包编辑了阿里云密钥服务的`serverless`模块
* Linux 环境，由于阿里云`FC云函数`的`BUG`,`serverless`必须在`linux`中打包。在`windows`中打包即便修改了`GOOS`也会报错。
* 若无法使用`Golang`环境，可等待作者开发`Python`版本的`serverless`模块(大概)
* 阿里云`OSS`拥有一个私有读写权限的`Bucket`，用以保存提供给`serverless`的伪接口。可以与其他服务公用`Bucket`，本项目仅在`Bucket`中占用`api`文件夹。
* 阿里云开启`FC云函数`服务，用以部署`serverless`服务

> 本项目消耗的`OSS`与`FC云函数`资源每月月末基本都会抹零。

### 创建 Sentry 账号

访问 [`Sentry`官网](https://sentry.io)，创建一个账号。

### 克隆项目

```bash
git clone git@github.com:CuratorC/sentry-white.git
```

### 配置伪接口

* 在`sentry-white`文件夹中打开命令行模式，运行以下命令

```bash
./sentry-white-go.exe
```

* 浏览器会自动打开`HTML`页面，在此页面配置`阿里云 OSS`信息。

![项目设置](/images/sentry-white/SentryWhite03.png)

* 进入`负责人`界面，将相关项目负责人添加到系统中。

* 进入`组织信息`界面，将刚才创建的`Sentry`账号视作一个组织新建出来。

### 创建云函数

* 将`sentry-white`文件夹中的`sentry-white-serverless`文件夹复制至`GOPATH`中，打开`main.go`文件，将阿里云相关的四条参数填写至文件头的变量中。
* 在`linux`环境中将`sentry-white-serverless`打包。相关命令如下
  ```bash
  // 修改 Go 环境参数
  go env -w GOOS=linux
  go env -w CGO_ENABLED=0
  // 打包项目
  go build main.go
  // 压缩项目
  zip sentry-white-serverless.zip main
  ```
* 将打包出来的`sentry-white-serverless.zip`复制出来备用。

* 进入阿里云`函数计算FC`-`服务及函数`-`创建服务`
  * `创建函数`，选择`从零开始创建`。
    * 运行环境选择`Go 1.X`
    * 函数触发方式选择`通过 HTTP 请求触发`
    * 实例类型选择`弹性实例`，内存规格选择`128M`
  * 创建成功后选择`上传代码`，将`sentry-white-serverless.zip`上传上去。
  * 上传完成后点击`测试函数`，当运行成功并看到`部署成功`后，`sentry-white`已完成部署。
  * 选择`触发器管理`，复制`公网访问地址`备用。此地址之后每次配置项目都需要使用。

## 项目接入

### Sentry

* 在`Sentry`中创建需要监控的项目，按照`Sentry`官网或者[『ThreeParty』Sentry 接入](/three-party/SentryUse)指引将脚本加入项目中。
> 建议采用[『ThreeParty』Sentry 接入](/three-party/SentryUse)指引接入，因为官网指引缺乏项目环境方面的配置。
* 开启`Webhooks`功能
  * 在项目设置中找到`Alerts`(从上往下第三条)，打开后找到`WEBHOOKS`配置。
  * 将阿里云`函数计算FC`中的`公网访问地址`填入`Callback URLs`点击`Save Changes`
* 开启消息通知
  * 继续在本页面点击右上角的`View Alert Rules`按钮。
  * 新页面继续点击右上角的`Create Alert`按钮。
  * 新页面点击中间偏下的`Set Conditions`按钮。
  * 按照下图提示配置好消息通知。其中蓝框内都是需要修改的地方。
  ![Alters 配置](/images/sentry-white/SentryWhite05.png)

### 钉钉

* 在钉钉中创建一个群，将相关项目负责人拉入群内。
  * 在`群设置`-`智能群助手`-`添加机器人`-`自定义`-`添加`。
  * `安全设置`中选择`自定义关键词`，输入`详情`点完成。
  * 添加成功后会给个`Webhook`地址，复制出来可见此地址为一个固定地址加`access_token`参数组成。复制`access_token`等待稍后使用。

### sentry-white

* 点击组织的`查看项目`，点击`新建`创建需要监控的项目。
  * `项目名（中文）`指项目中文代称，方便浏览。
  * `项目名（英文）`指项目文件夹名称/git仓库名/sentry上填写的项目名等。
  * `项目负责人`选择项目报错后需要通知的人。当选择了多个人之后，报错发生时每个人都会被艾特到。

* 编辑机器人信息
  * 将`钉钉群名称`与`access_token`参数填入，并保存。

## 项目设计思路
![思维导图](/images/sentry-white/mind-map.png)
