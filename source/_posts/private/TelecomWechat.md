---
title: 电信公众号免登接入说明
description: 电信公众号免登接入说明
keywords: 电信
top_img: /images/private/TelecomWechatCover1.png
cover: /images/private/TelecomWechatCover1.png
tags:
  - 电信
categories:
  - Private
date: 2021-12-27 15:25:48
updated: 2021-12-27 15:25:48
---
### 场景说明
> 电信公众号需要完成 公众号用户进入页面时实现免登的操作


### 免登流程
* 用户进入页面时，判断本地是否存在`token`。
  * 当本地存有`token`时直接进入页面。
  * 当本地不存在`token`时，判断地址里是否携带参数`phone`与`origin`。
    * 当不存在上述参数时，跳转至登录页面。
    * 当存在`phone`与`origin`参数时，使用`phone`作为`ticket`参数，调用 [三方用户登录](http://docs.cloudvhall.com/sw/index.html?src=/api/zjtelecom_api.yaml#/User/post_login_third) 接口完成登录。
      * 若接口访问正常获取到用户信息，令用户进入页面即可。
      * 若接口返回`422`错误，说明登录失败，跳转至之前完成开发的登录页面即可。
* 示例参数
  ```javascript
  // xxxxUrl?phone=f8a7479a130e41d52bc68c54e181e9e7&origin=ZQWeChatService
  let phone = "f8a7479a130e41d52bc68c54e181e9e7";
  let origin = "ZQWeChatService";
  ```
> 完成上述免登流程后，用户应当不会看到登录页面。


### 免登接口
> [点击查看接口文档](http://docs.cloudvhall.com/sw/index.html?src=/api/zjtelecom_api.yaml#/User/post_login_third)
