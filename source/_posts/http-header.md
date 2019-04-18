---
title: 网站安全配置（响应头篇）
date: 2017-04-25 19:49:00
tags:
- nginx
- php
categories:
- 安全
comments: true
---

默认的网站环境中。有很多暴露环境信息的地方。

在 HTTP 响应头中
* `Server` 参数会把 http 服务器的类型和版本信息完整的返回. apache 就更过分了. php 版本也会显示出来
* `Set-Cookie` 参数关联后台的服务器的 session 默认会使用 PHPSESSID
* `X-Powered-By` 参数会返回当前环境所使用的语言以及版本信息。

那要怎么办呢. 虽然在我看来并没有什么意义. 但是黑客大佬们可不会那么想. 多暴露一些信息就会多一些安全隐患.

话不多说. 以下是解决一些方法. 网上的很多比我写还好的. 如有雷同. 勿喷

接下来我会以下面两套环境来进行处理.
1. apache + php
2. nginx + php

处理的方式

1. 伪造成其他同类产品的默认响应头。
2. 彻底隐藏

先说下 apache 的环境吧

apache 是把 php 当成自己的一个模块运行的. 所以它才会知道 php 是什么版本.
在网上我找了很多资料, 都说想响应头中的 server 不能隐藏起来（你要知道和我说下哟）. 最少也会留个服务器类型. 下面我要做的就是把他能隐藏的信息隐藏起来

```conf
ServerTokens
ServerSignature
```

在 apache.conf 中找到了这两个配置值.

windows 里的为什么找不到阿. 那是因为他在 apache/conf/extra/httpd-default.conf 中

注意咯: 这里改了还要看下. apache/conf/httpd.conf 里有没有把这个文件引入. 可以在 httpd.conf 中搜索 httpd-default.conf 如果前面有 # 注释就去除.

你找到了这两参数了你不说怎么设置吗? 网上的文章和我配置里的配置参数不一样. 不要问我是怎么知道的. 我也是看这个参数上在写的注释里得知的.

```conf
#
# ServerTokens
# This directive configures what you return as the Server HTTP response
# Header. The default is 'Full' which sends information about the OS-Type

# and compiled in modules.
# Set to one of:  Full | OS | Minor | Minimal | Major | Prod
# where Full conveys the most information, and Prod the least.
#
```

看到用 | 隔开的了吗? 那就是参数. 如果你英文好. 那你一定知道是这些参数代表着什么. 英文不好. 赶紧翻译去.

哦, 对了. 是翻译你自己的不是翻译我的(笑)

我设置的是 Prod(产品). 想深入研究也可以把参数每个配置一遍看看

`ServerSignature` 参数也一样. 进行设置.(记得看注释哦)

配置完了. 重启 `apache`. 看看对比对比有没有变化.

## nginx

主要也是隐藏自身的版本号.

参数
server_tokens off;

这个参数只用设置到 http{} 里面就可以了

## php

php 里可以控制显示 X-Powered-By 和 Set-Cookie 里的信息.

主要配置参数都在 php.ini 中的
* `session.name` 控制 `Set-Cookie` 里的一个 key 相信你看到了熟悉的 `PHPSESSID` , `laravel` 框架默认使用 `laravel_session`
* `expose_php` 控制着 `X-Powered-By` 的输出. 我们关掉就可以了

## 其他

如果你使用的是空间.

使用php 函数 ini_set 进行设置
```php
ini_set('session.name', 'newvalue');
header_remove('x-powered-by'); // php 5.3+
header('x-powered-by:'); // 通用兼容方法,不过响应头会输出空的 x-powered-by
```

注意: 如果你网站是放在空间里运行的话. 空间默认是会帮你进行一些安全处理的. 先查看它的显示方式再进行修改. 要不然会出现一些可笑的现象.
如: 请求头中出现了两个 `X-Powered-By`

有些框架默认会使用 php 函数设置这些配置. 如 ThinkPHP(X-Powered-By), laravel(set-cookie)

## 误区

1. php ini_set() 可以设置 expose_php 配置.

在设置系统参数之前请查看清楚文档的介绍. 网上很多都写了用 ini_set() 可以设置 expose_post 的配置. 事实上只有 php.ini 可以进行设置

了解更多:
* http://php.net/manual/zh/ini.list.php
* http://php.net/manual/zh/configuration.changes.modes.php

## 总结

1. 要想让项目尽量的安全, 要注意安全方面的事情要从细节抓起.
2. 配置里有很多你没有想到的功能. 不一定要碰到了才去处理. 粗略看一遍也会对了解系统有很大的帮助
