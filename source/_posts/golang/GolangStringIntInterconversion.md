---
title: Golang中字符串和各种int类型之间的相互转换方式
description: Golang中字符串和各种int类型之间的相互转换方式
keywords: Golang, int, string, 格式转换
top_img: /images/golang/GolangCover3.png
cover: /images/golang/GolangCover3.png
tags:
  - Golang
  - 入门
categories:
  - Golang
date: 2021-08-12 17:35:09
updated: 2021-08-12 17:35:09
---
# Golang中字符串和各种int类型之间的相互转换方式：

## string转成int：
```go
int, err := strconv.Atoi(string)
```

## string转成int64：
```go
int64, err := strconv.ParseInt(string, 10, 64)
```

## int转成string：
```go
string := strconv.Itoa(int)
```

## int64转成string：
```go
string := strconv.FormatInt(int64,10)
```
