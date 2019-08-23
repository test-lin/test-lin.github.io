---
title: 登录失败次数限制
date: 2019-04-11 23:44:37
updated: 2019-04-11 23:44:37
tags:
- redis
- php
categories:
- 功能
comments: true
---

## 说明

这是一个防频繁请求登录接口导致服务器性能占用过多。也可以防止别人尝试密码

## 原理

记录用户访问次数，到指定次数后对他进行锁处理。实现方法很简单，前端使用 cookie 进行请求记录，请求成功会把登录次数相关的 cookie 清理掉。接口端也需要一起进行记录防止直接对接口进行请求。接口端使用 redis 记录他的 IP，又用别一个键来判别锁状态。

## 实现

```php
<?php

$redis = new Redis();
$redis->connect("127.0.0.1", 6379);
$redis->auth('you_password');
$redis->select(3);

$ip = get_client_ip();

if ($redis->get("login_lock.{$ip}")) {
    // 禁止访问接口
}

// 添加访问次数
$redis->incr("login_count.{$ip}");

// 上锁
$addLockCount = 3;
if ($addLockCount < $redis->get("login_count.{$ip}")) {
    // 锁定次数
    $redis->incr("login_lock_count.{$ip}");

    // 锁定次数越多，锁定时间越长
    $lockTime = 60 * $redis->get("login_lock_count.{$ip}");
    $redis->set("login_lock.{$ip}", $ip, $lockTime);

    // 清空登录记录数
    $redis->del("login_count.{$ip}");
}

// 登录成功要记得去除关联的缓存
$redis->del("login_count.{$ip}");
$redis->del("login_lock.{$ip}");
$redis->del("login_lock_count.{$ip}");

```