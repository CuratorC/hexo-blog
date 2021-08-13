---
title: Ubuntu 部署 LNMP 环境
description: 一篇文章，掌握 Ubuntu 部署 LNMP 环境全流程！要部署，看我就对了。
keywords: Ubuntu, LNMP, PHP, MySQL, Redis, Nginx
top_img: /images/ubuntu/UbuntuCover1.jpg
cover: /images/ubuntu/UbuntuCover1.jpg
tags:
  - Ubuntu
  - 入门
categories:
  - Ubuntu
date: 2021-08-13 15:02:13
updated: 2021-08-13 15:02:13
---
# Ubuntu 20.04 搭建 lnmp 环境

## 切换为 root 用户

我们之后要进行大量 root 权限操作，提前切换用户会方便一些。

```bash
sudo su
```

## 查看当前系统的版本信息

``` shell
lsb_release -a
```

## 更新 apt 包源

### 备份默认的源

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

### 修改为国内的阿里云镜像源

```bash
sed -i "s/archive.ubuntu.com/mirrors.aliyun.com/g" /etc/apt/sources.list
```

### 删除#开头的注释行

```bash
sed -i '/^#/d' /etc/apt/sources.list
```

### 添加 PHP 源

```bash
apt install lsb-release ca-certificates apt-transport-https -y
add-apt-repository ppa:ondrej/php
```

### 更新 apt 源

```bash
apt update
```

## MySQL 8.0

Ubuntu 20.04 已包含 mysql8.0的安装包，直接 apt 安装就好

```bash
apt-get install mysql-server
```

### MySQL服务管理

```bash
service mysql status # 查看服务状态
service mysql start # 启动服务
service mysql stop # 停止服务
service mysql restart # 重启服务
```

### 登录MySQL

“我的 MySQL 初始账号和密码是什么？”

关于这个问题每个系统上的每代 MySQL 都有自己的想法，Ubuntu20.04 也不例外。

初始账户信息使用这条查看

```bash
cat /etc/mysql/debian.cnf
```

使用默认账户登录

```bash
mysql -u debian-sys-maint -p
```

### 修改 root 密码 方便使用

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY "password";
```

{% note info flat%}
其他授权操作查看[这篇博客](/mysql/MySQLCreateUsersAndAuthorization)
{% endnote %}

## PHP

### 安装 PHP

```bash
apt install php8.0
```

查看 PHP 版本 `php -v`

### PHP 常用扩展

```bash
apt install php8.0-mbstring php8.0-sqlite3 php8.0-redis php8.0-gd php8.0-fpm php8.0-curl php8.0-xml php8.0-mysql
```

## Composer

### 进入安装包存放目录

```bash
cd /usr/src
```

### 下载安装脚本

```bash
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
```

### 执行安装过程

```bash
php composer-setup.php
```

### 删除安装脚本

```bash
php -r "unlink('composer-setup.php');"
```

### 移入运行目录

```bash
mv composer.phar /usr/local/bin/composer
```

### 查看 Composer 版本

```bash
composer --version
```

## Nginx

### 安装

Nginx 在默认的 Ubuntu 源仓库中可用。想要安装它，运行下面的命令：

```bash
apt install nginx
```

### 查看服务状态

```bash
systemctl status nginx
```

若报错`System has not been booted with systemd as init system (PID 1). Can't operate.`，可能使用的是 `SysV init` 命令。将 `systemctl status service_name` 改为 `service service_name status` 即可.

```bash
service nginx status
```

### 子站点配置

子站点配置文件统一放在 `/etc/nginx/conf.d/` 下

用自己的配置替换下面的 域名 和 public 目录

```vim
server {
        listen 80;
        listen [::]:80;

        server_name 域名;

        root public目录;
        index index.php index.html index.htm index.nginx-debian.html;


        # 允许上传大小
        client_max_body_size 100M;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
        }

        if (!-e $request_filename)
        {
                rewrite ^/(.*)$ /index.php?/$1 last;
                break;

        }

}
```
