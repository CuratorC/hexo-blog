---
title: ESSO 用户接入文档-前端手册
description: 前端上架浙里办，需要接入 ESSO 用户体系完成验证。这个用户体系
keywords: ESSO, 浙里办
top_img: /images/three-party/ThreePartyCover3.png
cover: /images/three-party/ThreePartyCover3.png
tags:
  - ESSO
  - 浙里办
categories:
  - ThreeParty
date: 2021-12-14 11:59:13
updated: 2021-12-14 11:59:13
---
## 登录状态判断规则
* 与`IDaaS`相同，当用户通过单点登录地址完成登录操作后，地址栏中会携带一个参数作为登录`token`。
* 此参数的参数名为`access_token`，值为`string`格式的`Bearer`类型`token`字符串
* 若存在此参数，说明用户已完成登录，调用后台接口时在`header`中携带`Authorization`字段，对应的值为`Bearer `拼接`access_token`
  ```javascript
  headers = {
    'Authorization': 'Bearer ' + access_token
  }
  ```
* 若不存在此参数，在需要用户登录的时候令用户跳转至单点登录地址即可。

## 单点登录地址生成规则
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
* 令浏览器跳转至拼接出来的单点登录地址。
  ```javascript
  location.href = esso_url;
  ```

## 测试账号
* 公共测试账号：zjfrcszh
* 公共密码：Zjfrcszh123

## 登出

> `ESSO`用户的登出需要分两步执行：首先解除我们后台服务器的登录状态，然后跳转`ESSO`地址解除用户登录操作。
> `ESSO`只提供了正式服的登出操作。若在测试服中想要模拟登出操作，可在访问后台接口后关闭浏览器来完成`登出操作`。

* 调用`数字展览浙里办API`提供的登出接口。
  * [文档地址：http://docs.cloudvhall.com/sw/index.html?src=/api/dea_ban_api.yaml#/User/post_logout](http://docs.cloudvhall.com/sw/index.html?src=/api/dea_ban_api.yaml#/User/post_logout)

* 使用页面跳转的方式解除`ESSO`用户系统的登录记录。
  >登出的跳转地址生成方式类似于单点登录的生成规则，不同的是`当前url不能进行url编码操作`
  * 正式服地址：`http://esso.zjzwfw.gov.cn/opensso/UI/Logout?goto=https://oauth.zjzwfw.gov.cn/oauth/logout.do?redirect=`
  ```javascript
  const ESSO_LOGOUT_PREFIX = 'http://esso.zjzwfw.gov.cn/opensso/UI/Logout?goto=https://oauth.zjzwfw.gov.cn/oauth/logout.do?redirect=';
  location.href = ESSO_LOGOUT_PREFIX + location.href;
  ```

## 错误处理
* 401错误：
  * 401 错误为账户登录错误，一般是`token`过期造成的。解决方案为重新跳转单点登录地址，令用户再次完成登录操作即可。
* 未找到回调地址：
  * `url_encode`没有正确拼接到单点登录地址上，请检查单点登录地址生成是否正确。
* 令牌错误:
  * 此错误一般是后台错误，出现这条提示请反馈给后台检查。
* errorcode= 13 messageParam= Error_invalid_Attrs:
  * 单点登录回调地址的授权操作，遇到此错误请反馈给后台检查。
