---
title: 1.3 xdebug性能分析
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

在上两期中我已经对 xdebug 最核心的操作已经进行了讲解。相信你可以摆脱写 var_dump($data);die; 的编写和完成调试后的清除了。这一期我们来学习xdebug的第二个特色-性能分析。这一个功能，在实操中用的不是很多。

使用场景：

1. 高并发项目的核心功能优化。通过查看运行一个方法经过的依赖耗时情况，进行代码优化。

## 环境说明

* windows
* vagrant+vbox+centos7+nginx+php
* phpstorm

## 学前准备

1. xdebug + phpstorm 调试环境已经可以正常运行

## 学习点

1. 明白 xdebug 性能分析需要做那些配置
2. 知道怎么去看性能报告

## xdebug 配置

xdebug 是通过访问指定方法。生成性能分析文件，再通过分析软件进行查看性能结果。

php.ini 添加配置

```sh

xdebug.profiler_enable=Off
xdebug.profiler_enable_trigger=On
xdebug.profiler_enable_trigger_value="create"
xdebug.profiler_output_dir="/tmp/"
xdebug.profiler_output_name="cachegrind.out.%R"

```

## phpstorm 分析性能文件

比较建议一个方法访问完后，生成分析文件，马上进行性能分析，分析完后再清除文件。为了数据准确性，我们还需要进行多次对比。找性能参数的平均值来提高准确性。

我们得知道那个依赖方法耗时最久，是什么原因。有没有优化的可能。

### phpstorm xdebug 性能分析工具详解

我们通过 tool -> Analyze Xdebug profiler Snapshot 打开性能分析文件 cachegrind.out. 打头的文件。就会进入以下界面
![Analyze Xdebug profiler Snapshot](/img/xdebug/31.png)

1. 了解各个选项的意思

* Refresh - 刷新
* Execution statistics - 执行统计数据
* Call Trees - 哪个函数调用哪个函数
* Callable - 已执行的文件
* Own Time - 函数执行自己的代码所花费的时间(不包括对其他函数的调用)
* Calls - 调用次数
* Callees - 调用哪些函数
* Callers - 从函数被调用的地方

* time 前的数字代表的是什么意思

time 列里有 数字和百分比。分别代表 执行时间和执行占用总时间百分比

单位是 server 旁边的 time 那里进行设置.默认是 ms

* callees 里的方法大部分都会出来一个折叠的图标，有什么用

这个可以让我们查看他的上一步执行了以什么操作。这个可以方便我们在了解调用到这个函数的过程。

2. 找到我们关心的数据

* 那些地方占用的执行时间最多，为什么

我们可以在 Execution statistics 标签里对 own time 进行排序取执行占用最多的内容。里面会包含文件和函数以及方法。如果我们设置了 server 关联当前的项目，统计里的方法是可以进行跳转到项目的实际代码里。

我们通过 Callees 标签查看函数里那些方法调用耗时最多

* 那些方法调用最多次，在那些地方调用的比较多

Execution statistics 标签里对 calls 进行排序取执行次数最多的函数或方法。

可通过下面的 Callers 查看那些地方对他进行了调用

## 系列文章

1. [xdebug的安装配置](/工具/xdebug-install-and-config)
2. [xdebug的实际运用](/工具/xdebug-use/)
3. xdebug性能分析 [本篇]

## FQA

### 为什么要设置 xdebug.profiler_output_name 默认的不就可以了吗？

如果你的项目不是多入口的类型，你保存的到一个文件就会出现性能分析文件不精确的情况。而我在上文中用的是 $_SERVER['REQUEST_URI'] 作为文件后缀。可以很好的区分性能分析文件。
