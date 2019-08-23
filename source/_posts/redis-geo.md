---
title: Redis GEO 附近的连锁店
date: 2019-04-21 16:16:28
updated: 2019-04-21 16:16:28
tags:
- php
- redis
- geo
categories:
- 功能
comments: true
---

## 说明

reids 里自带的一个坐标定位方法。

## 使用

```php
<?php

$redis = new Redis();
$redis->connect("127.0.0.1", 6379);
$redis->auth('you_password');
$redis->select(3);

// 添加坐标
$key = '7-Eleven';
list($x, $y) = [23.1194200000,113.4155300000];
$region = "广州车坡分店";
$redis->geoAdd($key, $x, $y, $region);

// 更新坐标
list($x, $y) = [25.1194200000,114.4155300000];
$redis->geoAdd();

// 删除坐标
$redis->del($key);

$redis->zRem($key, $region);

// 店铺经纬度
$redis->geoPos($key, $region);

// 店铺与店铺之间的距离
$redis->geoDist($key, $region, $region, 'km');

// 附近的店
$options = [
    'ASC'
];
$redis->geoRadius($key, -157.858, 21.306, 1, 'km', $options):
```
