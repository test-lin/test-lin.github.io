---
title: 两数之和
date: 2019-03-28 13:47:27
updated: 2019-03-28 13:47:27
tags:
- php
- algorithm
categories:
- 算法
comments: true
---

## 题目

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 **两个** 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:
> 给定 nums = [2, 7, 11, 15], target = 9
>
> 因为 nums[0] + nums[1] = 2 + 7 = 9
> 所以返回 [0, 1]

[题目链接](https://leetcode-cn.com/problems/two-sum/)

## 标签

* 数组
* 哈希表

## 个人解答

```php
# 方法1
function twoSum($nums, $target) {
    $count = count($nums);
    for ($i=0; $i < $count; $i++) {
        for ($j=($i+1); $j < $count; $j++) { // $j 之是 $i+1 是因为不能重复利用数组中同样的元素。
            if (($nums[$i]+$nums[$j]) == $target) {
                return [$i, $j];
            }
        }
    }

    return [];
}

# 方法2：建立值键对应
function twoSum($nums, $target) {
    $count = count($nums);
    $values = [];
    for ($i=0; $i < $count; $i++) {
        $values[$nums[$i]] = $i;
    }

    for ($i=0; $i < $count; $i++) {
        $value = $target - $nums[$i];
        if (isset($values[$value])) {
            return [$i, $values[$value]];
        }
    }

    return [];
}

# 方法3: 通过值找键
function twoSum($nums, $target) {
    for ($i=0; $i < count($nums); $i++) {
        $searchValue = $target - $nums[$i];
        // array_search 是语言自带的函数，应该和方法2差不多。
        if ($key = array_search($searchValue, $nums)) {
            return [$i, $key];
        }

        unset($nums[$i]);
    }

    return [];
}

# 方法4：匹配不到的值存放到数组中，方便后面查找
function twoSum($nums, $target) {
    $values = [];
    for ($i=0; $i < count($nums); $i++) {
        $value = $target - $nums[$i];
        if (isset($values[$value])) {
            return [$i, $values[$value]];
        }

        $values[$nums[$i]] = $i;
    }

    return [];
}
```

## 做题总结

虽然两数相加得到结果很简单，但是没有实现思路逃避了许久。后面看了解答的文字，不看代码。才以最简单的方式做出来。

## 未掌握知道点

* 空间/时间复杂度计算
* 双循环掌握不好
