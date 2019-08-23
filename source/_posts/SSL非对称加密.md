---
title: SSL 非对称加密
date: 2019-04-03 15:31:43
updated: 2019-04-03 15:31:43
tags:
- php
categories:
- 功能
comments: true
---

## 说明

SSL(Secure Sockets Layer 安全套接层),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。而它的代表软件则是 OpenSSL。php 使用 openssl 函数则可以对 OpenSSL 进行调用。

## 原理

非对称加密主要是通过 ssl 的公钥私钥都可以加密解密的特性来做的一个数据安全验证。使用到非对称加密技术的公司有支付宝、微博。大至交互流程如下：

![](/img/f0b9c2f51f2698c9f1b3163e0123e23e.png)

前置准备

1. 服务端需要开放公钥
2. 接入方需要提供公钥给服务端

请求与返回

1. 接入方请求服务端，使用自己的私钥生成加密串
2. 服务端接收请求，根据请求里的用户编号取接入方公钥
3. 根据接入方公钥对请求加密串进行校验
4. 服务端处理完后，使用服务端私钥为返回数据添加加密串
5. 接入方验证后进行指定处理

## 代码实现

```php
<?php

$data = [
    'account_id' => 1,
];
$privateKeyId = openssl_pkey_get_private(file_get_contents('id_rsa'));

openssl_sign(json_encode($data), $requestSign, $privateKeyId, OPENSSL_ALGO_SHA256);

$data['sign'] = base64_encode($requestSign);

// ===== 请求接口 ====

$request = $data;
$sign = base64_decode($data['sign']);
unset($request['sign']);

$userPublicKeyId = openssl_pkey_get_public(file_get_contents('id_rsa.pub'));

if (! openssl_verify(json_encode($request), $sign, $userPublicKeyId, OPENSSL_ALGO_SHA256)) {
    while ($error = openssl_error_string()) {
        print_r($error);
    }
}

$response = [
    'status' => 200,
    'message' => 'hello world'
];

$severPrivateKeyId = openssl_pkey_get_private(file_get_contents('private_key.pem'));

openssl_sign(json_encode($response), $responseSign, $severPrivateKeyId, OPENSSL_ALGO_SHA256);

$response['sign'] = base64_encode($responseSign);

// ===== 接口响应 ====

$ruslt = $response;
$sign = base64_decode($ruslt['sign']);
unset($ruslt['sign']);

$severPublicKeyId = openssl_pkey_get_public(file_get_contents('public_key.pem'));

if (! openssl_verify(json_encode($ruslt), $sign, $severPublicKeyId, OPENSSL_ALGO_SHA256)) {
    while ($error = openssl_error_string()) {
        print_r($error);
    }
}

// 处理业务逻辑
```

## FQA

### 为什么不是先使用服务端的公钥加密进行请求先呢？

我一开始构思时是这样想的，请求使用服务端公钥加密，接口返回使用接入方的私钥进行解密。感觉并没有什么毛病，但是细细一想，这里涉及到证书安全问题和请求验证问题。

1. 请求时使用的是服务端公钥，凭加密串上面的用户号是无法保证请求用户真实性
2. 响应时如果接入方私钥泄漏，就可以被人直接模拟支付回调了

