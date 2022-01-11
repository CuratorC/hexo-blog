---
title: OSS 前端静态托管全流程
description: OSS 前端静态托管保姆级教程
keywords: 'OSS, Web, 静态托管'
top_img: /images/three-party/ThreePartyCover4.png
cover: /images/three-party/ThreePartyCover4.png
tags:
  - OSS
  - Web
  - 入门
categories:
  - ThreeParty
date: 2021-12-20 09:31:29
updated: 2021-12-20 09:31:29
---
# 开通 OSS 服务
* 访问 [阿里云官网](https://www.aliyun.com)，找到 `对象存储 OSS`
  ![阿里云官网](/images/three-party/OSSWeb1.png)
* 点击`立即开通`完成账号注册及开通后，进入`OSS 控制台`


# 创建 Bucket
* 按照指引点击`创建 Bucket`来创建一个部署前端代码的`容器`
  ![Bucket 创建表单](/images/three-party/OSSWeb2.png)
* 版本控制选择`不开通`之后，参考下列提示一项项填写表单
  * `Bucket 名称`: 填写一个全网唯一英文的名称，单词之间用中横线`-`连接。
  * `地域`: 选择一个离自己最近的网点。
  * `Endpoint`: 这里显示的是`OSS`域名的一、二级域名。随地域自动变换。
  * `存储类型`: 作为一个经常被访问的前端页面，需要选择`标准存储`。其他两种虽然价格更低，但是读取方面有些限制。
  * `同城冗余存储`、`版本控制`:不开启
  * `读写权限`: 因为前端资源需要公开到网络上被大家自由获取，所以选择`公共读`，弹出的警告框选`继续修改`
  * `服务端加密方式`、`实时日志查询`、`定时备份`均为默认的无或者不开通


# Bucket 设置
* 创建成功后进入`基础设置`->`静态页面`
  ![Bucket 静态页面](/images/three-party/OSSWeb3.png)
  * 点击`静态页面`的`设置`，`默认首页`设置为`index.html`,其他保持不变点`保存`
* 选择`文件管理`的`上传文件`，将`index.html`及同文件夹的所有文件拖拽至上传页面，点击`上传文件`
  ![Bucket 上传文件](/images/three-party/OSSWeb4.png)
  ![Bucket 上传文件](/images/three-party/OSSWeb5.png)
> 阿里云处于安全考虑，通过自己域名获取的资源默认下载。所以我们需要绑定自己的域名
* 等待上传完成后找到`传输管理`->`域名管理`->`绑定域名`，填写要绑定的域名。
  ![Bucket 域名管理](/images/three-party/OSSWeb6.png)
  * 若绑定的域名在当前账号内，可以自动完成添加`CNAME`记录的功能，打开自动添加 CNAME 记录的功能后点击`提交`即可。
    ![Bucket 绑定域名](/images/three-party/OSSWeb7.png)
  * 否则请在点击`提交`后，前往个人域名的控制台手动添加一条`CNAME`记录。


## 手动添加域名 CNAME
* 在`OSS 控制台`->`概览`->`访问域名`中找到`外网访问`这一行中的`Bucket 域名`,将其复制。
* 来到`域名控制台`中解析域名，参考下方提示添加一条记录。
  ![域名解析](/images/three-party/OSSWeb8.png)
  * `记录类型`: 选择`CNAME` 将域名指向另一个域名。
  * `主机记录`: 输入你在`OSS`配置的域名前缀。
  * `记录值`: 粘贴你在`Bucket 控制台`复制的`Bucket 域名`
  * 其余保持默认。


# 完成 OSS 静态托管部署
* 配置完域名后，访问之前配置的域名，可以看到前端页面飞速加载了出来！
