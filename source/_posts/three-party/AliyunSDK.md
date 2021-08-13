---
title: 阿里云冷门 API/SDK 对接心得
description: 当我们需要使用那种网上缺少讨论，出现问题难以搜索到解决方案，难以获得讨论的 SDK ，如何依靠有限的经验解决当前问题。
keywords: 阿里云, SDK, API, 媒体转码服务
top_img: /images/three-party/ThreePartyCover1.jpg
cover: /images/three-party/ThreePartyCover1.jpg
tags:
  - ThreeParty
  - 探讨
categories:
  - ThreeParty
date: 2021-08-13 13:51:17
updated: 2021-08-13 13:51:17
---
# 阿里云冷门 API/SDK 对接心得
<p align="right">以『媒体转码服务』为例</p>

> 这里提到的`冷门`指的是网上缺少讨论，出现问题难以搜索到解决方案，难以获得讨论的`SDK`，如何依靠有限的经验解决当前问题。

首先要树立一个信心：
* `阿里爸爸`的产品这么香，肯定不是某个不负责任的小编用两张网图编出来了个`api`接口。
* 就算是产品再不人性化，也是经过了严格的评定和测试流程才挂到`阿里云产品`中 ~~ㅤ骗钱ㅤ~~ 造福大众的的。
* 获取制作这个产品的时候阿里云投入的产品力量不足，或许开发者由于某些思维惯性，没有在文档中写入一些细节，或者我们的知识、经验积累还不够，没办法第一时间理解某些参数的含义。
* 但是只要挖掘到阿里云在编写文档时的盲点，理解了调用方式，最终总能完成对接，给自己的技能库添砖加瓦。

## 与官方文档的第一步交锋
* 当我们尝试从官方途径寻找文档的时候，第一步总是能找到阿里云的[『帮助文档』](https://help.aliyun.com/)，对于很多热门产品，尤其是如`OSS`般的当红炸子鸡，赚钱小能手，这类产品的帮助文档已经被打磨的足够完善。
* 可惜今天的例子[『媒体转码服务』](https://help.aliyun.com/document_detail/29226.htm?spm=a2c4g.11186623.2.21.708d5ee7dUeKfs#reference-ksb-vdt-x2b)可能还不在此列。当你点击入我提供的链接时，聪明的你可能很快就会发现一个问题：接口地址是什么？
* 当然我们不能冤枉阿里云帮助文档，他们在媒体服务中还是提供了接口地址的，接口地址位于 >媒体处理 >API参考 >调用方式 >请求结构 >服务地域。如果要我们每个人都像这样去寻找每一个信息，那就要求我们每个人都要把整篇文档大致读一遍。这样就增加了很多的学习成本。
* 回到[『媒体转码服务』](https://help.aliyun.com/document_detail/29226.htm?spm=a2c4g.11186623.2.21.708d5ee7dUeKfs#reference-ksb-vdt-x2b)的帮助文档继续往下读，你会发现接口还需要我们提供一个`管道ID`和`模板ID`。读到这里我们已经意识到了问题的严重性：不要逃课了，先把整篇文档通读一遍吧，求求你了。
* 很抗拒，通读太累了。那就挑一些关键的信息看一看。比如`API概述`，`调用方式`之类的。这时候我们再顺手翻一翻控制台，能得到阿里云提供的一个固定的`管道ID`，使用控制台手动转码，能从他们的调用方式里获得公共的默认的`模板ID`。有点儿像是玩解密RPG？
* OK，信息够了，但也没完全够，这些接口还要`验签`。看到验签我终于坚持不下去了，继续逃课，选择使用他们封装好的`SDK`做开发。

## SDK：逃课好帮手
* 文档中的`SDK 参考`要我们看[『安装』](https://help.aliyun.com/document_detail/55742.htm?spm=a2c4g.11186623.2.4.17e257b2tQX5uH#concept-k5b-322-z2b)，安装文档给了我们一个`github 仓库`,和一行`require_once`代码。
* 感谢`composer`为我们养成了懒惰的引用习惯，看到`require_once`我不自觉要寻找`composer`版本的引用。但网上查到的都是抱怨阿里云自己没做`composer`，和个人自己做的`composer`版本。
* 这还是在`2021年4月18日`的搜索结果，在4天后的22号，情况已经不同了。
* 跳过自己改`composer`的步骤，后来我在查询资料的时候发现阿里云悄悄推出了新的`SDK平台`[『阿里云OpenApi』](https://next.api.aliyun.com/home)。这两天好像正在排查旧SDK的引用，准备全面引入新的平台上。
* 但是不要高兴的太早，等真正细读下来才发现这个平台对我们的`冷门SDK`还是没有足够的支持。只告诉你要用哪个方法调用这一步，参数要怎么填，是什么格式，什么含义，依旧一头雾水。

## SDK不完善，只能靠API文档来补课。
* 某些SDK的相关示例中，有标识为社区提供，但明显不是社区写的输入参数示意，提到了参数格式，参数含义，参数详情。当我们点击[『参数详情』](https://help.aliyun.com/document_detail/29253.html?spm=api-workbench.CodeSample%20Detail%20Page.0.0.11b91e0fMEAi59)时，API 文档：没想到吧，我又回来啦！
* SDK 初始化中要填写一个参数 `$config->endpoint`。一开始我在这里填了`oss`的`endpoint`，`SDK`直接报错404，内容大概是`it is not a map`，翻阅多层源代码才知道，这个`endpoint`就是接口调用地址，也就是之前我们在`API文档`中找到的`服务地域`。
* 我充分利用以上多方面数据来源，终于将`SDK`跑了起来。这一步常常会遇到数据格式不正确的报错。这时候要结合文档，示例，将参数传递方式在以下多种格式中选择合适的格式使用。

```php
[
    // 数组
    "input" => [
        'Bucket'    => config('aliyun.oss.bucket'),
        'Location'    => config('aliyun.mts.location'),
        'Object'    => urlencode($url),
    ],
    // json 对象
    "input" => json_encode([
        'Bucket'    => config('aliyun.oss.bucket'),
        'Location'    => config('aliyun.mts.location'),
        'Object'    => urlencode($url),
    ]),
    // json 数组
    "input" => json_encode([
        [
            'Bucket'    => config('aliyun.oss.bucket'),
            'Location'    => config('aliyun.mts.location'),
            'Object'    => urlencode($url),
        ],
        [
            'Bucket'    => config('aliyun.oss.bucket'),
            'Location'    => config('aliyun.mts.location'),
            'Object'    => urlencode($url),
        ],
    ]),
]

```