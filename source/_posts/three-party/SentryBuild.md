---
title: Sentry 部署
description: 简单来讲，Sentry 可以监控线上项目，将每一个报错上报，整理，主动推送给你的邮箱。如果你恰巧需要这样一个功能，那选 Sentry 准没错。
keywords: Sentry, 部署, 接入, Docker, Python,
top_img: /images/three-party/ThreePartyCover2.png
cover: /images/three-party/ThreePartyCover2.png
tags:
  - ThreeParty
  - 入门
categories:
  - ThreeParty
date: 2021-08-13 14:46:17
updated: 2021-08-13 14:46:17
---
# Sentry 部署

## 什么是 Sentry? 我为什么要使用它？
简单来讲，`Sentry`可以监控线上项目，将每一个报错上报，整理，主动推送给你的邮箱。

如果你恰巧需要这样一个功能，那选`Sentry`准没错。至于`Sentry`其他更高级的功能，可以等入门之后慢慢了解。

## Sentry 使用途径
`Sentry`可以自建服务，也可以采用官方提供的服务。官方提供的服务有免费的有收费的。以下是各个途径的功能对比。

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

### 官方服务
官方服务到 [官网(https://sentry.io/signup)](https://sentry.io/signup) 注册一个账号即可使用。
### 自建服务
部署 Sentry 有两种方式。`Docker` 与 `Python`。
* `Docker` 部署复杂程度低，但对硬件要求高。最新版最低需要`4核8G`，历史版本最低内存也需要`2400M`。
* `Python` 部署复杂程度较高，对硬件要求低。

可以视情况选择合适的自建方式。

## 使用 Docker 部署 Sentry

{% note warning flat %}
自建 Sentry 服务吃力不讨好，Docker 需要太高的配置，Python 又存在学习门槛。如果仅仅为了使用 Sentry，我建议使用官方服务。
如果官方的免费版无法满足使用需求，可以考虑使用我的另一个开源产品： GGT Sentry White ，结合钉钉机器人，提供报错信息的分发和报错信息日志功能。
{% endnote %}

### Docker 一键式部署
Sentry 官方提供一键式部署 Sentry，只需要克隆 github 上的脚本即可。
```bash
git clone https://github.com/getsentry/onpremise
```
若需要部署 2400M 大小内存限制的 Sentry，可以克隆历史版本。
```bash
git clone https://github.com/getsentry/onpremise/tree/20.12.1
```
克隆完成后进入目录，运行脚本 install.sh 即可使用。

## 使用 Python 部署 Sentry

### 必备环境
* Python3
* 关系型数据库（PostgreSQL）
* 内存型数据库（Redis）
* 进程守护（supervisor）

### 更新 apt 源

* 先复制备份原文件
* 然后修改为国内的阿里云镜像源
* 删除开头的 # 注释
* 更新 apt 源
```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
sed -i "s/archive.ubuntu.com/mirrors.aliyun.com/g" /etc/apt/sources.list
sed -i '/^#/d' /etc/apt/sources.list
apt update
```


### 安装 PostgreSQL
#### 安装
```bash
apt-get install postgresql
```
#### 配置
```bash
vim /etc/postgresql/12/main/pg_hba.conf
```
将配置文件翻到最后，这里保存着各自连接方式，授权和鉴权。将
```
host    all   all   127.0.0.1/32    md5
```
一行最后的`md5`改为`trust`，意为`信任localhost所有指令`

#### 创建数据库
* 切换到 postgres 用户，以使用 PostgreSQL
  ```bash
  sudo -i -u postgres
  ```
* 创建数据
  ```bash
  createdb sentry
  ```
* 进入数据库

{% note warning %}
在 Sentry 部署中，我们并不需要进入 sentry 数据库，之后查看数据也有其他数据库查看工具。
{% endnote %}

  ```bash
  psql sentry
  ```

### 安装 Redis

```bash
apt-get install redis
```

### 安装 virtualenv
`virtualenv` 是 `python` 用来部署独立环境的工具，类似于虚拟机，在`virtualenv`中部署的项目将独立于主服务。
#### 安装
```bash
pip3 install virtualenv
```
#### 使用
* 创建文件夹
  ```bash
  mkdir /var/www
  ````
* 创建独立环境
  ```bash
  cd /var/www
  virtualenv sentryEnv
  ```
* 进入环境
  ```bash
  cd sentryEnv
  source bin/activate
  ```
* 更新
  ```bash
  bin/python -m pip install --upgrade pip
  ```
* 退出环境

{% note warning %}
Python 部署 Sentry 的接下来全部指令均在 virtualenv 中运行，如非必要，不用退出。
{% endnote %}

  ```bash
  deactivate
  ```

### 安装 Sentry 前的基础环境

#### libxml2-dev
* 更新索引
  ```bash
  apt-get update
  ```
* 安装
  ```bash
  apt-get install libxml2-dev
  ```

#### libxmlsec1-dev
* ubuntu 20.04 libxmlsec1-dev 暂时无法通过 apt-get 安装，需要先安装一个安装程序 `aptitude`
  ```bash
  apt-get install aptitude
  ```
* 安装

{% note warning flat %}
`aptitude` 提供多种子依赖的安装模式，但是模式1并不能将 `libxmlsec1-dev` 的全部依赖安装完毕。所以在执行下面命令时一定要记得：第一次选择 Y/n/... 时要选择 n，第二次和第三次选择 Y
{% endnote %}

  ```bash
  aptitude install libxmlsec1-dev
  ```

#### libxmlsec1-openssl
```bash
apt-get install libxmlsec1-openssl
```  

#### pkg-config
```bash
apt-get install pkg-config
```

### 千呼万唤始出来，安装 Sentry
#### 安装
```bash
pip install -U sentry
```

#### 修改BUG

* 在 Python 部署 21.6.3 版本的 Sentry 时存在一个类型错误需要修改。
```bash
vim /var/www/sentryEnv/lib/python3.8/site-packages/dataclasses.py
```
* 查找错误
```vim
/typing._ClassVar
```
* 按`i`，修改为`typing.ClassVar`

#### 初始化
```bash
sentry init
sentry upgrade
```

### 安装 supervisor
#### 安装
```bash
pip install supervisor
```
#### 配置
```bash
vim /etc/supervisor/conf.d/sentry.conf
```
写入如下内容
```vim
[program:sentry-web]
directory=/usr/local/lib/python3.8/dist-packages/sentry/
environment=SENTRY_CONF="~/.sentry"
command=sentry run web
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/sentry/sentry-web.log
stderr_logfile=/var/log/sentry/sentry-web.log

[program:sentry-worker]
directory=/usr/local/lib/python3.8/dist-packages/sentry/
environment=SENTRY_CONF="~/.sentry"
command=sentry run worker
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/sentry/sentry-worker.log
stderr_logfile=/var/log/sentry/sentry-worker.log

[program:sentry-cron]
directory=/usr/local/lib/python3.8/dist-packages/sentry/
environment=SENTRY_CONF="~/.sentry"
command=sentry run cron
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/sentry/sentry-cron.log
stderr_logfile=/var/log/sentry/sentry-cron.log
```

#### 使用
```bash
# 查看所有进程的状态
supervisorctl status
# 停止all
supervisorctl stop all
# 启动all
supervisorctl start all
# 重启
supervisorctl restart
# 配置文件修改后使用该命令加载新的配置
supervisorctl update
# 重新启动配置中的所有程序
supervisorctl reload
```