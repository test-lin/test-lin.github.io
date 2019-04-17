---
title: 快速排序法理解
date: 2017-02-12 11:50:13
tags:
- php
- algorithm
categories:
- 算法
comments: true
---

**核心技术:** 递归, 数组合并, 引用

**说明:** 取排序中的一个数字. 对剩下的数字进行排序. 得到比这个数字 (中间值) 大的和小的两个数组. 再对这两个数组进行递归处理. 

**注意点:** 因为是递归所以需要递归点. 不然会出现死循环的情况. 这个递归点是: 对比数组长度小于等于 1的时候就直接返回

```php
<?php
$arr = array(5,2,7,9,10,3);

function kp(&$arr) { // 这里的引用很重要, 因为递归中会有没有返回值的情况.所以需要引用
    $count = count($arr);
    if ($count > 1) { // 递归点
        $median = $arr[0]; // 中间对比值 
        $min = array();
        $max = array();

        // 对比
        for ($i = 1; $i < $count; $i++) { // 注: 这里的是从数组 1 开始. 因为数组 0 被拿去当中间对比值了
            if ($arr[$i] <= $median) { // 这里使用小于等于是为了防止相等的值会出现问题
                $min[] =& $arr[$i]; // 这里使用[引用赋值]节省内存空间
            } else {
                $max[] =& $arr[$i];
            }
        }

        $min = kp($min); // 只有递归到 return $arr 这一步才会得这一段的结果
        $max = kp($max);

        // 返回排序结果
        return array_merge($min, array($median), $max);
    } else {
        return $arr;
    } 
}

$arr = kp($arr);
print_r($arr);

```