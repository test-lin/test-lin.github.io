---
title: 表状态程序管理应用
date: 2019-04-21 8:40:58
updated: 2019-04-21 8:40:58
tags:
- php
categories:
- 功能
comments: true
---

## 说明

表的状态是我们判断业务到达什么地方的标识，它可以是英文单词，也可以是数字，更可以是枚举。在开发的过程中，有人是直接写状态值，这样很不好维护，如果值变更，他就得把程序里关联的值全部都需要修改。

## 处理

### model 常量

model 对应的是数据表来创建的。所以表中的状态字段可以使用常量来表示，使用时直接使用常量访问即可。

```php
<?php

class OrderModel extends Model
{
    const STATUS_TO_AUDIT = 2;
    const STATUS_TO_DELIVER = 3;

    public function getList($where)
    {
        $map = [];
        $map['status'] = self::STATUS_TO_AUDIT;
    }
}
```

### model 静态属性

静态属性主要的作用是搜索下拉表单（需要格式化处理）和列表状态显示时使用。

```php
<?php

class OrderModel extends Model
{
    const STATUS_TO_AUDIT = 2;
    const STATUS_TO_DELIVER = 3;

    public static $status = [
        self::STATUS_TO_AUDIT => '待审核',
        self::STATUS_TO_DELIVER => '待发货',
    ];
}
```

### 混合状态管理

混合状态就是表示一种业务状态，需要几个状态字段进行判断才可以确认业务的实际状态。这种混合状态是很不好维护的，因为你在开发中会发现你总是漏改某些字段被挨批。管理这种状态的办法就是把实际状态用数组存储起来。

```php
<?php

class OrderModel extends Model
{
    // 订单状态
    const STATUS_TO_AUDIT = 2;
    const STATUS_TO_DELIVER = 3;
    const STATUS_SUCCESS = 4;

    // 售后状态
    const RETURN_NOT = 0;
    const RETURN_TODO = 1;
    const RETURN_ACCESS = 2;
    const RETURN_FAIL = 3;

    // 无售后订单
    private $orderSuccess = [
        'status' => self::STATUS_SUCCESS,
        'return_status' => self::RETURN_NOT,
    ];

    // 售后失败订单
    private $orderReturnFail = [
        'status' => self::STATUS_SUCCESS,
        'return_status' => self::RETURN_FAIL,
    ];

    // 变更表状态,使用类覆盖
    // $data += $this->orderReturnFail;

    // 验证订单状态
    // foreach ($this->orderReturnFail as $key => $value) {
    //     if ($data[$key] != $value) {
    //         return false;
    //     }
    // }
    // return true;
}
```

## FQA

### 验证当前操作的状态值是否合法

```php
<?php

if (!in_array($status, array_keys(OrderModel::$statusMap)) {
    // 非法状态值
}
```

### 表里的状态值很多，但是显示给用户只需要其中几个

```php
<?php

class OrderController
{
    public function getOrderStatus()
    {
        return [
            OrderModel::STATUS_TO_AUDIT => OrderModel::$statusMap[OrderModel::STATUS_TO_AUDIT],
            OrderModel::STATUS_SUCCESS => OrderModel::$statusMap[OrderModel::STATUS_SUCCESS],
        ];
    }
}
```
