---
title: layui 直传 OSS
description: 后端PHP使用 iidestiny/laravel-filesystem-oss  ，对 前端直传 有着良好的支持。这里不再赘述。
keywords: layui, OSS
top_img: /images/web/WebCover1.jpg
cover: /images/web/WebCover1.jpg
tags:
  - Web
  - 进阶
categories:
  - Web
date: 2021-08-13 10:44:15
updated: 2021-08-13 10:44:15
---
# layui 直传 OSS

{% note blue 'fas fa-bullhorn' simple %}
在现在的互联网背景下继续学习 layui 其实并不是一件很明智的事情。除非你真的很需要 layui 帮你快速解决问题，否则请考虑其他更专业的选择。
{% endnote %}

## 后端部分

后端PHP使用 `iidestiny/laravel-filesystem-oss` ，对`前端直传`有着良好的支持。这里不再赘述。

## 前端部分
前端使用 `layui`框架的`upload`功能。`upload`对OSS不存在任何支持，但好在在文件选择后的回调中，我们可以直接对`upload`对象进行修改，来达到我们的目的。接下来开始上代码。


```javascript
// 上传图片
let layLoading;
var uploadInst = upload.render({
    elem: '#image-upload'
    , url: '/upload/' // 随便填写一个，等下还要动态修改的。
    , choose: function (obj) { // 当文件被选择时
        layLoading = layer.load(); // load 层

        // 访问后台拿取阿里云的签名
        $.ajax({
            method: "POST",
            url: "", // 你的后台获取签名的地址
            async: false,
            done: function (signRes) { // 拿到签名的回调，修改 uploadInst 的属性。
                uploadInst.config.url = signRes.data.oss_config.host // 阿里云上传地址
                uploadInst.config.data = { // 附加参数
                    OSSAccessKeyId: signRes.data.oss_config.accessid
                    , policy: signRes.data.oss_config.policy
                    , Signature: signRes.data.oss_config.signature
                    , key: signRes.data.oss_upload_advance.name // 文件名。采取后端生成文件名的方式，来避免文件重名。
                }
            },
        });
    }
    , before: function (obj) {
        // 修改上传属性
        // 预读本地文件示例，不支持ie8
        obj.preview(function (index, file, result) {
            $('#cover-upload-img').attr('src', result).attr('style', '{display:inline}'); //图片链接（base64）
        });
    }
    , done: function (res) {
        // 关闭弹窗
        layer.closeAll();
        // 上传成功的操作

    }
    , error: function () {
        //演示失败状态，并实现重传
        var coverUploadText = $('#cover-upload-text');
        coverUploadText.html('<span style="color: #FF5722;">上传失败</span> <a class="layui-btn layui-btn-mini cover-upload-reload">重试</a>');
        coverUploadText.find('.cover-upload-reload').on('click', function () {
            uploadInst.upload();
        });
        // 关闭弹窗
        layer.closeAll();
    }
});

```

## layui 直传支持
此时点击上传会看到阿里云关于 key 的报错。因为`阿里云要求上传的文件 file 必须是在最后一位`，而`layui`的处理正好是`先把 file 放到第一位`。所以我们还需要`修改 layui 的源代码`：

找到`layui/lay/modules/upload.js`。无论你的代码是方便修改的源代码，还是构建过的源代码，都可以直接`全文搜索append`,一共会搜到四个`append`,将第三个`append`所在的逻辑块与第四个`append`所在的逻辑块对调，就能实现将`file`字段移动到最后一位。

```javascript
layui.each(l.data, function (e, t) {
    t = "function" == typeof t ? t() : t, r.append(e, t)
}),r.append(l.field, o);
```

前两个`append`是涉及`ie8,ie9的兼容模式`。这里暂不做讨论。 