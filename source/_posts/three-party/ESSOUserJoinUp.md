---
title: ESSO 用户接入文档-前端手册
description: 前端上架浙里办，需要接入 ESSO 用户体系完成验证。这个用户体系
keywords: ESSO, 浙里办
top_img: /images/php/PhpCover.png
cover: /images/php/PhpCover.png
tags:
  - ESSO
  - 浙里办
categories:
  - ThreeParty
date: 2021-12-14 11:59:13
updated: 2021-12-14 11:59:13
---
### 登录状态判断规则
* 与`IDaaS`相同，当用户通过单点登录地址完成登录操作后，地址栏中会携带一个参数作为登录`token`。
* 此参数的参数名为`access_token`，值为`string`格式的`Bearer`类型`token`字符串
* 若存在此参数，说明用户已完成登录，调用后台接口时在`header`中携带`Authorization`字段，对应的值为`Bearer `拼接`access_token`
  ```javascript
  headers = {
    'Authorization': 'Bearer ' + access_token
  }
  ```
* 若不存在此参数，在需要用户登录的时候令用户跳转至单点登录地址即可。

### 单点登录地址生成规则
* 获取当前url，并将其进行url编码
  ```javascript
  let url_encode = encodeURIComponent(location.href);
  ```
* 在url编码前拼接登录指定地址。
  * 测试服地址：`http://essotest.zjzwfw.gov.cn/opensso/spsaehandler/metaAlias/sp?spappurl=https://gdte-api.cloudvhall.com:8443/ban/api/v1/login/esso?goto=`
  * 正式服地址：`https://esso.zjzwfw.gov.cn/opensso/spsaehandler/metaAlias/sp?spappurl=https://dea-api.idtshow.com:8443/ban/api/v1/login/esso?goto=`
  ```javascript
  // 正式服地址
  // const ESSO_PREFIX = 'https://esso.zjzwfw.gov.cn/opensso/spsaehandler/metaAlias/sp?spappurl=https://dea-api.idtshow.com:8443/ban/api/v1/login/esso?goto=';
  // 测试服地址
  const ESSO_PREFIX = 'http://essotest.zjzwfw.gov.cn/opensso/spsaehandler/metaAlias/sp?spappurl=https://gdte-api.cloudvhall.com:8443/ban/api/v1/login/esso?goto=';
  let esso_url = ESSO_PREFIX + url_encode;
  ```
  * 将跳转至拼接出来的单点登录地址。
  ```javascript
  location.href = esso_url;
  ```
### 错误处理
* 401错误：
  * 
