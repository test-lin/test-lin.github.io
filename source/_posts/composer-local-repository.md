---
title: satis composer 本地仓库搭建
date: 2018-09-26
tags:
- composer
- php
- satis
categories:
- 工具
comments: true
---

## 环境

* windows
* nginx
* php
* composer

## 安装

拉取 satis 项目包，并拉取项目依赖

```sh
composer create-project composer/satis --stability=dev

cd satis

composer install
```

## 配置

修改 satis/config.json 文件，文件内容如下

```json
{
    "name": "composer 本地仓库",
    "homepage": "http://packages.example.org", // 访问域名
    "repositories": [// 要拉取包的仓库地址
        { "type": "vcs", "url": "https://github.com/test-lin/db.git" },
        { "type": "vcs", "url": "https://github.com/test-lin/queue.git" },
        { "type": "vcs", "url": "https://github.com/test-lin/cache.git" },
        { "type": "vcs", "url": "http://192.168.6.251:3000/php/xjwSpider.git" }
    ],
    "require": { // 要拉取到本地的包文件 注：不会包含包的依赖
        "test-lin/db": "*",
        "test-lin/queue": "*",
        "test-lin/cache": "*",
        "php/xjwSpider": "*"
    },
    "archive": {
        "directory": "dist",
        "format": "tar",
        "prefix-url": "http://packages.example.org" // * 这个参数是当前项目的域名，作用是以zip压缩包的方式直接下载包文件
    }
}
```

## 拉取包到本地仓库

web/ 是本地仓库访问地址。

```sh
php bin/satis build config.json web/
```

如果需要定时更新，则需要配置定时任务去定时更新

## 设置本地仓库

nginx 设置虚拟主机

```conf
server {
    listen 80;
    server_name packages.example.org;
    root /var/www/satis/web;
    index index.php index.html;

    location ~* \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
    }
}
```

## 使用本地仓库中的包

composer.json 文件中添加以下 json 拉取，即可获取本地库了.

如果本地仓库不存在且有网络会去网络中获取。repositories 参数可以设置多个

```json
{
  "repositories": [{
    "type": "composer",
    "url": "http://packages.example.org"
  }]
}
```

## FQA

### 1. github 的包需要配置 token

```shell
Could not fetch https://api.github.com/repos/test-lin/db/git/refs/heads?per_page=100, please create a GitHub OAuth token to go over the API rate limit
Head to https://github.com/settings/tokens/new?scopes=repo&description=Composer+on+packages.example.org+2018-06-28+0310
to retrieve a token. It will be stored in "/home/vagrant/.config/composer/auth.json" for future use by Composer.
```

解决方法：

访问命令行中提示的 `https://github.com/settings/tokens/new?scopes=repo&description=Composer+on+packages.example.org+2018-06-28+0310`

复制 token description 文本框中内容

拉到页底 点击 generate token

在命令行中粘贴复制内容确认限可

### 2. 私有包，拉取不了

解决方法：

本地生成 ssh key ，配置到要拉取项目的平台即可，免密拉取了

```shell
ssh-keygen -t rsa

cat ~/.ssh/id_rsa.pub
```

以 gogs 为例

![](/img/composer/1.jpg)

### 3. composer 不支持 http

```
Your configuration does not allow connections to http://192.168.6.251:3000/php/xjwSpider.git. See https://getcomposer.org/doc/06-config.md#secure-http for details.
```

解决方法：

```sh
composer config -g secure-http false
```

### 4. 拉取的包 composer.json 配置有误

```shell
[Composer\Repository\InvalidRepositoryException]
No valid composer.json was found in any branch or tag of http://192.168.6.251:3000/php/xjwSpider.git, could not load a package from it.
```

解决方法：

1. 确保项目根部有 composer.json
2. composer.json 里需要设置 name
