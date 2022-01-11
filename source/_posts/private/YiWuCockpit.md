---
title: 义乌市展馆数字驾驶舱
description: 义乌市展馆数字驾驶舱开发说明
keywords: 浙政钉, 义乌驾驶舱
top_img: /images/private/YiWuCover1.png
cover: /images/private/YiWuCover1.png
tags:
  - 浙政钉
categories:
  - Private
date: 2021-12-16 10:54:48
updated: 2021-12-16 10:54:48
---

# 单点登录
> 单点登录实现方式参考已完成的项目，参考代码的获取请在群里沟通。

# 展馆页开发说明

![展馆UI图示](/images/private/YiWuExhibitionHall.png)

## 左
### 展馆信息/综合配置
> 此数据直接写在页面上，不对接接口，不对数据进行修改或调整。

| 栏目 | 参数 | 栏目 | 参数 |
| --- | --- | --- | --- |
| 总建筑面积 | 29.6万 | 总会议室面积 | 750 |
| 展厅数量 | 10个 | 可提供展位数 | 5300 个 |
| 会议室总数 | 20个 | 楼层 地上2层地下2层 | 4层 |

> 酒店信息请在以下四个之间滚动播放

| 酒店名称 | 星级 | 建筑面积 | 可提供套房/客房数 | 酒店会议室数量 |
| --- | --- | --- | --- | --- |
| 幸福湖国际会议中心 | 五星级 | 8万 | 376 | 13 |
| 义乌海洋酒店 | 四星级 | 3.05万 | 285 | 6 |
| 义乌皇冠假日酒店 | 五星级 | 5.5万 | 337 | 10 |
| 义乌银都酒店 | 四星级 | 3.2万 | 217 | 3 |

### 展馆出租率
> 调用接口：[展馆出租率](http://docs.cloudvhall.com/sw/index.html?src=/api/gdte_halladmin_api.yaml#/Echart/get_echarts_exhibition_hall_occupancy_rate)
* 传参说明
  * `division_code` 无需传值
  * `exhibition_hall_id` 固定为 `1`，代表查看`义乌国际博览中心`的数据
  * `date_type`传递的值为要查看的年份，比如要查看2021年的出租率，`date_type: 2021`即可

### 展会排期
> 关于展会排期我们有一个已经开发完成的`template`名为`展会日历`。请在群里获取代码并在此基础上调整表现效果以符合`UI`的要求。请勿修改此`template`的接口及逻辑

## 中
### 平面图
* 此处单纯一张图片，无互动无动画，图片从UI中自取。
### 展会信息
> 调用接口：[展会列表](http://docs.cloudvhall.com/sw/index.html?src=/api/gdte_halladmin_api.yaml#/Echart/get_echarts_exhibition)
* 传参说明
  * `exhibition_hall_id` 固定为 `1`，代表查看`义乌国际博览中心`的数据
  * `date_type`传递的值固定为`2`
  * `field`排序字段，因为针对的功能不同，所以传值要求也不同。在此项目中请传递`start_time`
  * `order`排序方式，请传递`DESC`
  * `in_progress` 是否需要正在举办中的展会，不传此参数
  * 新增固定参数`extras`，值为`1`，若不传递此参数，返回信息可能不完整。

## 右
### 平安展馆
> 由于平安展馆内的两项数据均无法对接真实数据，这里直接生成一个随机数即可
> 随机数需要根据当前时间符合以下条件

* 随机数每5秒变动一次
* 变动幅度不能过大，需要在合理的范围内少量递增或递减或波动
* 每天早晨五点半起，数值缓慢波动且递增，下午两点达到峰值左右，随后缓慢递减，至晚上十一点达到谷值，随后维持在谷值附近波动直到第二天早晨五点半。
* 当前馆内人数峰值为`3500`，谷值为`5`, 在早晨五点半至晚上十一点的曲线参考函数
  ```javascript
  let numberOfPeople = 0.218928*x*x*x*x-9.80358*x*x*x+90.573778*x*x+652.04052*x-4890.34164
  ```
* 近一小时新增客流峰值为`2150`， 谷值为`0`, 在早晨五点半至晚上十一点的曲线参考函数
  ```javascript
  let passengerFlow = -0.348976*x*x*x*x+20.93919*x*x*x-471.069259*x*x+4628.6198*x-14371.98745
  ```

### 绿色展馆
> 调用接口：[获取绿色展位率](http://docs.cloudvhall.com/sw/index.html?src=/api/gdte_halladmin_api.yaml#/Echart/get_echarts_green_exhibition_booths)
* 此接口为专用接口，传参及返回均以接口文档为准。

### 展会赋能
> 月度调用接口：[获取展会赋能数据](http://docs.cloudvhall.com/sw/index.html?src=/api/gdte_halladmin_api.yaml#/Echart/get_echarts_energize_for_exhibitions)
> 年度调用接口：[获取展会赋能数据](http://docs.cloudvhall.com/sw/index.html?src=/api/gdte_halladmin_api.yaml#/Echart/get_echarts_energize_for_exhibitions_year)
* 此接口为专用接口，传参及返回均以接口文档为准。

# 展会页开发说明

![展会UI图示](/images/private/YiWuExhibition.png)

> 展会页中所有展示的数据均来自同一个接口：[获取展会详情](http://docs.cloudvhall.com/sw/index.html?src=/api/gdte_halladmin_api.yaml#/Echart/get_echarts_exhibitions__exhibition_id_)

| 条目 | 键名 | 条目 | 键名 |
| ---- | ---- | ---- | ---- |
| 展览面积 | area_size | 展位数 | number_of_booths |
| 展会成交 | volume_of_business | 主办方 | responsible_organization |
| 累积参展人数 | number_of_visitors | 展场联动数 | number_of_linkages |
| 展会连续性 | continuous_times | 采购商参展商比值 | purchaser_exhibitor_rate |
| 参展驻留时间 | dwell_time | 展馆人流统计 | people_counting |
| 参展商数量 | number_of_exhibitors | 采购商数量 | number_of_purchasers |
| 市外参展商 | number_of_external_exhibitors | 市内参展商 | number_of_inside_purchasers |
| 特装展位比例 | special_exhibition_booth_rate | 绿色展装比例 | green_exhibition_booth_rate |
