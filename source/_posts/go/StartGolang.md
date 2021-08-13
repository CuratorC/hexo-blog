---
title: Start Golang
description: 下载并安装 Golang
keywords: Golang, 下载, 安装
top_img: /images/golang/GolangCover.jpg
cover: /images/golang/GolangCover.jpg
tags:
  - Golang
  - 入门
categories:
  - Golang
date: 2021-08-12 16:49:48
updated: 2021-08-12 16:49:48
---
# Start Golang

## 安装 Go

### 下载 Golang
* 下载地址 [https://golang.google.cn/dl/](https://golang.google.cn/dl/)
* 当前版本 go1.16.5

### 设置 Go Module
* 查看 Go 环境变量
  ```bash
  go env
  ```
* 开启 Go Module
  ```bash
  go env -w GO111MODULE=on
  ```
* 开启GOPROXY
  ```bash
  go env -w GO111MODULE=on
  ```
* 安装 Go 文档
  ```bash
  go get golang.org/x/tools/cmd/godoc
  ```