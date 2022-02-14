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
SENTRY_DSN = {DSN_ADDRESS}
// 采样率
SENTRY_TRACES_SAMPLE_RATE = 1
```

### 3. 在入口文件中初始化 Sentry


  <!-- tabs:start -->

#### **Vue2**

* 引入

  ```javascript
  // sentry
  import * as Sentry from '@sentry/vue'
  import { BrowserTracing } from '@sentry/tracing'
  ```

* 初始化

  ```javascript
  Sentry.init({
    Vue,
    dsn: process.env.SENTRY_DSN,
    integrations: [new Integrations.BrowserTracing()],

    // Set tracesSampleRate to 1.0 to capture 100%
    // of transactions for performance monitoring.
    // We recommend adjusting this value in production
    tracesSampleRate: parseInt(process.env.SENTRY_TRACES_SAMPLE_RATE ?? '0.2'),
    environment: process.env.NODE_ENV,
  });
  ```

#### **Vue3**

* 引入

  ```javascript
  // sentry
  import * as Sentry from "@sentry/vue";
  import { BrowserTracing } from "@sentry/tracing";
  ```

* 初始化

  ```javascript
  Sentry.init({
    app,
    dsn: process.env.SENTRY_DSN,
    integrations: [
      new BrowserTracing({
        routingInstrumentation: Sentry.vueRouterInstrumentation(router),
        tracingOrigins: ["localhost", "my-site-url.com", /^\//],
      }),
    ],
    // Set tracesSampleRate to 1.0 to capture 100%
    // of transactions for performance monitoring.
    // We recommend adjusting this value in production
    tracesSampleRate: parseInt(process.env.SENTRY_TRACES_SAMPLE_RATE ?? '0.2'),
    environment: process.env.NODE_ENV,
  });
  ```

  <!-- tabs:end -->


