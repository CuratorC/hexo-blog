---
title: 使用 GoLand 进行 Golang 开发
description: 使用 GoLand 进行 Golang 开发
keywords: GoLand, Golang
top_img: /images/golang/UseGoLand/Cover.png
cover: /images/golang/UseGoLand/Cover.png
tags:
  - 1111
categories:
  - Golang
date: 2022-01-10 14:02:48
updated: 2022-01-10 14:02:48
swiper_index:
swiper_desc:
swiper_cover:
---

# 第一次使用 GoLand

## 创建项目，`SDK`选择`download`，下载合适的版本。

* 如有需要，可自定义`SDK`下载路径。如我的地址为`C:\Go`
* 项目路径与名称可以随意选择，新建项目只是为了进入设置界面与终端界面，稍后删掉项目即可。

## 检查环境变量。
* 依路径打开设置界面：`file(文件)`->`setting(设置)`->`Go`
* `GOROOT`设置为Go下载地址，精确到版本的文件夹
![GOROOT界面](/images/golang/UseGoLand/01.png)
* `GOPATH`的全局GOPATH设置为项目列表的目录地址，而不是单个项目的根目录。
![GOPATH界面](/images/golang/UseGoLand/02.png)
* `Go Module(Go 模块)`设置为开启状态，变量数值设置如下
  ```text
  GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
  ```
![GO模块界面](/images/golang/UseGoLand/03.png)

## Air 自动重载方案

* `go`每次改动代码时都需要重新打包，使用`air`可以完成自动重载，无需重复打包与等待。
* 执行下面命令
  ```bash
  go install github.com/cosmtrek/air@latest
  ```

## gcc 问题

> `windows`使用`sqlite`等库时会遇到`gcc`问题，需要安装`MinGW-w64`来解决。
* [点击自动下载（若下载速度过慢请使用科学上网）](https://versaweb.dl.sourceforge.net/project/mingw-w64/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-posix/seh/x86_64-8.1.0-release-posix-seh-rt_v6-rev0.7z)
* 解压至应用目录，如我的地址为`C:\GoPlugins`
* 将`bin`目录添加至环境变量，如我的地址为`C:\GoPlugins\mingw64\bin`

> 完成上述所有操作后可以关闭`GoLand`令`mingw64`重新加载，之前创建的项目也可以删除。如有需要新建项目请重新打开`GoLand`继续后续操作。

# 新建项目

* 选择`Go`而不是`Go(GOPATH)`
> Go(GOPATH)是`go 1.11`版本之前使用的模式
* 位置为 `GOPATH + "/src/github.com/作者名称/" + 项目名称`
* `GOPATH`和`环境变量`保持默认即可。
