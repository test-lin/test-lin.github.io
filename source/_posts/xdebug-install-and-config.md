---
title: 1.1 Xdebug的安装配置
date: 2018-09-25
tags:
- xdebug
- php
- phpstorm
categories:
- 工具
comments: true
---

## 介绍

我之前配置直接按网上的文章进行配置总是配置不成功，里面很多东西不了解。当我在 xdebug 官网看到了 xdebug 远程调试原理图时，我才知道应该怎么配置 xdebug。配置的参数也少了很多，也不需要在 IDE 里对一个个请求地址进行配置。体验比之前看到的文章设置好用多了

## 环境说明

* windows
* vagrant+vbox+centos7+nginx+php
* phpstorm

## 配置前准备

1. 检查 php 环境是否已经存在 xdebug 拓展了

在命令行中输入 php -m 可以查看 php 已加载的拓展
![](/img/xdebug/7.jpg)


2. 了解自己环境的配置
* php 版本
* php.ini 所在地址
* 服务器系统位数 32位 还是 64位

3. xdebug 和 phpstorm 交互的原理

![](/img/xdebug/dbgp-setup.gif)

* 服务器的IP和端口是 10.0.1.2:80
* IDE 的客户端IP是 10.0.1.42, 所以服务器上 xdebug.remote_host=10.0.1.42
* IDE 监听的调试端口为 9000, 所以服务器上 xdebug.remote_port=9000
* IDE 所在的客户端，对 xdebug 的服务器进行请求
* Xdebug 与 10.0.1.42:9000 的客户端 IDE 监听端口关联
* 运行调试, xdebug 所在的服务器提供 HTTP 响应

![](/img/xdebug/dbgp-setup2.gif)

* 服务器的IP和端口是 10.0.1.2:80
* IDE 的客户端IP是一个未知的IP, 所以服务器上 xdebug.remote_connect_back=1
* IDE 监听的调试端口为 9000, 所以服务器上 xdebug.remote_port=9000
* 发出 HTTP 请求后，Xdebug 将从 HTTP 请求头获取 IP 地址
* Xdebug 会和从 HTTP 请求头获取 IP 地址的客户端 IDE 监听端口关联
* 运行调试, xdebug 所在的服务器提供 HTTP 响应

## 下载缺少的扩展和软件

* [phpstorm 编辑器](http://www.jetbrains.com/phpstorm/download/)
* [xdebug 扩展](https://xdebug.org/download.php)

## 配置

### php 环境配置添加 xdebug 扩展

1. 下载 xdebug 扩展源码
```

cd /usr/local/src
wget https://xdebug.org/files/xdebug-2.7.0alpha1.tgz

解压
tar -zxvf xdebug-2.7.0alpha1.tgz

```

2. 编译安装
```

cd xdebug-2.7.0alpha1

生成安装脚本
phpize

设置安装配置参数
vim install-sh
> /configure --with-php-config=/usr/local/php/bin/php-config

运行安装配置
sh install-sh

编译扩展，使用两个 cpu 内核运行（可以快很多）
make -j 2

编译安装
make install

```

**注意**： 编译安装后，会返回扩展所在文件夹。

3. php.ini 中添加 xdebug 配置

```

[xdebug]
zend_extension="/usr/local/php/lib/php/extensions/no-debug-non-zts-20160303/xdebug.so"
xdebug.idekey="PHPSTORM"
xdebug.remote_enable = On
xdebug.remote_autostart=On
xdebug.remote_connect_back=On
xdebug.remote_port=9001

```

### phpstorm 配置

1. 打开设置 file -> settings -> Languages & Frameworks
2. 设置项目使用环境
![](/img/xdebug/1.jpg)
![](/img/xdebug/2.jpg)
3. 设置 debug 配置 Languages & Frameworks -> php -> debug
![](/img/xdebug/3.jpg)
4. 设置 xdebug dbgp 配置 Languages & Frameworks -> php -> debug -> dbgp proxy
![](/img/xdebug/4.jpg)
5. 设置当前项目所在服务器地址和域名
![](/img/xdebug/5.jpg)

## 开始调试

1. 打开编辑器调试监听
![](/img/xdebug/6.jpg)
2. 在指定控制器中添加断点
3. 请求地址，编辑器会自动进入调试模式中

## 系列文章

1. xdebug的安装配置 [本篇]
2. [xdebug的实际运用](/工具/xdebug-use/)
3. [xdebug性能分析](/工具/xdebug-property/)

## FQA

### 我的 php 运行环境在 windows 下应该要怎么设置

windows 的添加扩展会比 linux 简单很多。直接下载 dll 扩展文件就可以了。除了 zend_extension 设置的地址不一样。其他可以 xdebug 配置可以共用。

### php.ini 中配置 xdebug 为什么不用默认的 9000 端口

因为 php-fpm 是使用 cgi协议 进行运行，所以它也需要端口，而它默认的端口也是 9000。如果你像我这样使用虚拟机的方式进行访问项目，不会出问题。但是如果你使用的是本地的 php-fpm 那他就会出现端口被占用的情况。为了避免就直接用 9001 来代替默认端口

### php.ini 中可以配置的 xdebug 参数有那些，我应该在那里得到更全面的参数说明

xdebug 官网那里的手册有详细说明， [xdebug 远程连接文档链接](https://xdebug.org/docs/remote) 里的 ctrl + f 搜索 Related Settings 就可以看到连接参数了

### 我应该下载那个版本的 xdebug

如果实在不知道自己的 windos 系统的 php 环境该用那个版本的扩展
可以通过下载页提供的工具进行下载 [工具链接](https://xdebug.org/wizard.php)

![](/img/xdebug/8.jpg)

多行文本框里面是放通过 php -i 命令返回的配置内容
为了更完整的取得参数可以 php -i > D:/php-ini.txt 保存到文件中

### 为什么我的 ide 配置好后，启动调试监听没有效果

这个很有可能是你系统的防火墙的安全机制。把这个端口保存起来了，可以直接关闭防火墙进行调试