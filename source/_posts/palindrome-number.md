---
title: 回文数
date: 2019-04-09 18:25:15
updated: 2019-04-09 18:25:15
tags:
- php
- algorithm
categories:
- 算法
comments: true
---

## 题目

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

**示例 1:**

```
输入: 121
输出: true
```

**示例 2:**

```
输入: -121
输出: false
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```

**示例 3:**

```
输入: 10
输出: false
解释: 从右向左读, 为 01 。因此它不是一个回文数。
```

**进阶**:

你能不将整数转为字符串来解决这个问题吗？

[题目链接](https://leetcode-cn.com/problems/reverse-integer/)

注： 反转回文时需注意数值溢出

## 标签

* 数学

## 个人解答

如题可知单数，负数，0结尾的整数都不是回文数

```php
# 方法1 前后数字对比
function isPalindrome($x) {
    if ($x < 9 || ($x % 10 === 0)) return false;

    $x = (string)$x;
    $i = 0;
    $max = strlen($x) - 1;
    while ($i < $max) {
        if ($x{$i} == $x{$max}) {
            $i++;
            $max--;
        } else {
            return false;
        }
    }

    return true;
}

# 方法2 翻转数字对比
function isPalindrome($x) {
    if ($x < 9 || ($x % 10 === 0)) return false;

    $rev_num = 0;
    $val = $x;
    while ($val != 0) {
        // 注：凡是数字反转都可能会存在数字值越范围的情况
        $rev_num = bcmul($rev_num, 10) + ($val % 10);// ($rev_num * 10) + ($val % 10)
        $val = bcdiv($val, 10); // $val / 10;
    }

    return ($rev_num === $x);
}

# 方法3 数字法前后对比
function isPalindrome($x) {
    if ($x < 9 || ($x % 10 === 0)) return false;

    // 取 $x 的位数
    $bit = 1;
    while (9 < bcdiv($x, $bit)) { // 除出来的数大于 9 就要进位
        $bit = $bit * 10;
    }

    while (9 < $x) { // 除下的数如果大于 9 就还可以进行对比，如 10 的 10，个位数就不用比了
        $l = bcdiv($x, $bit); // $x / $bit
        $r = $x % 10;

        if ($l != $r) return false;

        $x = $x % $bit; // 去头
        $x = bcdiv($x, 10); // 去尾 $x / 10
        $bit = bcdiv($bit, 100); // $bit / 100 = 头+尾 = 10 * 10
    }

    return true;
}
```

## 做题总结

这到题加深了上周做的整数反转所使用的循环技巧。