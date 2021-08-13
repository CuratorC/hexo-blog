---
title: Sentry 接入
description: Vue/Laravel 接入 Sentry
keywords: Vue, Laravel, Sentry
top_img: /images/three-party/ThreePartyCover2.png
cover: /images/three-party/ThreePartyCover2.png
tags:
  - ThreeParty
  - 进阶
categories:
  - ThreeParty
date: 2021-08-13 14:38:38
updated: 2021-08-13 14:38:38
---

# Sentry 接入

## 使用方式对比

Sentry 提供网页版应用。监控服务直接部署在他们的服务器上。提供免费使用和两种升级。我们使用 Sentry 有四种方案。

* 方案一：自建服务器部署 Sentry
* 方案二：Sentry 提供的免费版本，各自管理各自的项目
* 方案三：Sentry 团队版，基本满足团队开发所需
* 方案四：Sentry 商业版，提供更高级的图表和跨项目联动

因为Sentry提供14天的最高版本试用，我把几种方案都试了试。下面是功能对比

<table style="text-align:center;">
<thead>
<tr>
<td style="width:300px">功能列表</td><td style="width:100px">自建服务</td><td style="width:100px">免费版</td><td style="width:100px">团队版</td><td style="width:100px">商业版</td>
</tr>
</thead>
<tbody>
<tr>
<td>多个项目管理</td><td style="color:green">√</td><td style="color:green">√</td><td style="color:green">√</td><td style="color:green">√</td>
</tr>
<tr>
<td>监控报错</td><td style="color:green">√</td><td style="color:green">√</td><td style="color:green">√</td><td style="color:green">√</td>
</tr>
<tr>
<td>邮件通知</td><td style="color:green">√</td><td style="color:green">√</td><td style="color:green">√</td><td style="color:green">√</td>
</tr>
<tr>
<td>开通子账户</td><td style="color:green">√</td><td style="color:red">×</td><td style="color:green">√</td><td style="color:green">√</td>
</tr>
<tr>
<td>一个月以上数据保存</td><td style="color:green">√</td><td style="color:red">×</td><td style="color:green">√</td><td style="color:green">√</td>
</tr>
<tr>
<td>作为插件接入其他管理平台</td><td style="color:red">×</td><td style="color:red">×</td><td style="color:green">√</td><td style="color:green">√</td>
</tr>
<tr>
<td>更多花里胡哨的统计</td><td style="color:red">×</td><td style="color:red">×</td><td style="color:red">×</td><td style="color:green">√</td>
</tr>
</tbody>
</table>

{% note info flat %}
最终我们选择了团队版。不过有条件的也可以选择自建服务。登录地址 [https://sentry.io/signup](https://sentry.io/signup)
{% endnote %}

## Laravel 项目接入步骤

### 1.使用 composer 安装插件

```bash
composer require sentry/sentry-laravel
```

### 2.重写 report 方法

在文件`App/Exceptions/Handler.php`中加入以下方法

```php
public function report(Throwable $e)
{
    if (app()->bound('sentry') && $this->shouldReport($e)) {
        $_SERVER['SENTRY_ENVIRONMENT'] = config('app.env');
        app('sentry')->captureException($e);
    }

    parent::report($e);
}
```

### 3.发布配置文件

{% note warning flat %}
注意替换下方的 {DSN_ADDRESS}，发布过程中需要输入两次`yes`
{% endnote %}

```bash
php artisan sentry:publish --dsn={DSN_ADDRESS}
```

### 4.配置环境变量

在不同环境的`.env`文件中加入以下代码，初始的`.env`文件末尾已经在发布配置文件的时候写入了DSN地址，如有需要，注意替换。

{% note warning flat %}
注意替换下方的 {DSN_ADDRESS}，正式服的采样率可以设置为0.2，即每5个人中只有一个人进行报错监测
{% endnote %}

```
# Sentry DSN 地址
SENTRY_LARAVEL_DSN={DSN_ADDRESS}
# 采样率
SENTRY_TRACES_SAMPLE_RATE=1
```

## Vue 项目接入方法

### 1. 使用 yarn 或者 npm 安装插件

* yarn
  ```bash
  yarn add @sentry/vue @sentry/tracing
  ```
* npm
  ```bash
  npm install --save @sentry/vue @sentry/tracing
  ```

### 2. 配置环境变量

在不同环境的`.env.js`文件中加入以下代码。

{% note warning flat %}
注意替换下方的 {DSN_ADDRESS}，正式服的采样率可以设置为0.2，即每5个人中只有一个人进行报错监测
{% endnote %}

```javascript
// Sentry DSN 地址
SENTRY_LARAVEL_DSN = {DSN_ADDRESS}
// 采样率
SENTRY_TRACES_SAMPLE_RATE = 1
```

### 3. 在入口文件中初始化 Sentry

* 引入

  ```javascript
  import Vue from "vue";
  import * as Sentry from "@sentry/vue";
  import {Integrations} from "@sentry/tracing";
  ```

* 初始化

  <!-- tabs:start -->

#### **JavaScript**

  ```javascript
  Sentry.init({
    Vue,
    dsn: process.env.SENTRY_LARAVEL_DSN,
    integrations: [new Integrations.BrowserTracing()],

    // Set tracesSampleRate to 1.0 to capture 100%
    // of transactions for performance monitoring.
    // We recommend adjusting this value in production
    tracesSampleRate: parseInt(process.env.SENTRY_TRACES_SAMPLE_RATE ?? '0.2'),
    environment: process.env.ENV,
  });
  ```

#### **TypeScript**

{% note warning flat %}
 `TypeScript`存在类型报错，但不影响`Sentry`的正常工作。使用注释`// @ts-ignore`让`ts`忽略此问题即可
{% endnote %}

  ```typescript
    // @ts-ignore
Sentry.init({
    Vue,
    dsn: process.env.SENTRY_LARAVEL_DSN,
    integrations: [new Integrations.BrowserTracing()],

    // Set tracesSampleRate to 1.0 to capture 100%
    // of transactions for performance monitoring.
    // We recommend adjusting this value in production
    tracesSampleRate: parseInt(process.env.SENTRY_TRACES_SAMPLE_RATE ?? '0.2', 10),
    environment: process.env.ENV,
})
  ```

  <!-- tabs:end -->


