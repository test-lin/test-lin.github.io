---
title: Explain SSL Pinning with simple codes(用简单代码解释 SSL 固定)
date: 2019-03-31 10:47:21
tags:
- ssl
categories:
- 翻译
comments: true
---

## HTTPS and SSL Pinning

## HTTPS 和 SSL 固定

There are two major factors in an HTTPS connection, a valid certificate that server presents during handshaking, and a cipher suite to be used for data encryption during transmission. The certificate is the essential component and serves as a proof of identity of the server. The client will only trust the server if the server can provide a valid certificate that is signed by one of the trusted Certificate Authorities that come pre-installed in the client, otherwise, the connection will be aborted.

HTTPS 连接有两个主要因素，握手时服务器提供一个有效证书，以及在传输过程中用于数据加密的密码套件。证书是基本组件，用作服务器身份的证明。只有当服务器能够提供由预装在客户机中的受信任证书颁发机构之一签名的有效证书时，客户机才会信任服务器，否则连接将被中止。

An attacker can abuse this mechanism by either install a malicious root CA certificate to user devices so the client will trust all certificates that are signed by the attacker, or even worse, compromised a CA completely. Therefore relying on the certificates received from servers alone cannot guarantee the authenticity of the server, and it is vulnerable to potential man-in-the-middle attack.

攻击者可以滥用此机制，方法是向用户设备安装恶意根CA证书，以便客户机信任由攻击者签名的所有证书，或者更糟，完全破坏CA。因此，仅依靠从服务器接收到的证书无法保证服务器的真实性，容易受到潜在的中间人攻击。

SSL Pinning is a technique that we use in client side to avoid man-in-the-middle attack by validating the server certificates again even after SSL handshaking. The developers embed (or pin) a list of trustful certificates to the client application during development, and use them to compare against with the server certificates during runtime. If there is a mismatch between the server and the local copy of certificates, the connection will simply be disrupted, and no further user data will be even sent to that server. This enforcement ensures that the user devices are communicating only to the dedicated trustful servers.

SSL 固定是我们在客户端使用的一种技术，通过在 SSL 握手之后再次验证服务器证书来避免中间人攻击。开发人员在开发期间向客户机应用程序嵌入(或pin)一组可信证书，并在运行时使用它们与服务器证书进行比较。如果服务器和证书的本地副本不匹配，则连接将被中断，甚至不再向该服务器发送用户数据。这种强制确保用户设备只与专用的可信服务器通信。

However, developers must take extra caution in SSL Pinning, When a pinned certificate is expired and the server has updated a new certificate, since the new certificate is definitely different from the pinned one that the client currently has, the clients will not trust the updated certificate and therefore terminate the connection, in which no further communications can be established between clients and servers, so the application is basically ‘bricked’. Therefore, to avoid such situation, it is always advisable to pin the future certificates in the client applications before release.

然而,开发人员必须采取额外的谨慎在SSL固定,当固定证书过期和服务器已经更新一个新的证书,自从新证书肯定是不同的固定客户目前,客户不会信任更新后的证书,因此终止连接,没有进一步的客户端和服务器之间的通信可以建立,所以应用程序基本上是“给”。因此，为了避免这种情况，通常建议在发布之前将未来的证书固定在客户机应用程序中。

There are usually two ways we can achieve SSL Pinning in client applications. Pin either the whole certificate or its hashed public key. The hashed public key pinning is the preferred approach because the same private key can be used in signing the updated certificate, therefore we can save the trouble of pinning a new hashed public key for a new certificate, and reduce the risk of app ‘bricking’.

通常有两种方法可以在客户机应用程序中实现SSL固定。对整个证书或其散列公钥进行Pin。散列公钥固定是首选方法，因为可以使用相同的私钥对更新后的证书进行签名，因此我们可以省去为新证书固定新散列公钥的麻烦，并降低应用程序“变黑”的风险。

## Codes In Action

## 代码在行动

I am going to write a simple Android application that communicates with <www.google.com>, and use [Charles proxy](https://www.charlesproxy.com/) as a man-in-the-middle server, and finally demonstrate how an effective SSL Pining can stop man-in-the-middle attack.

我将编写一个与 <www.google.com> 通信的简单 Android 应用程序，并使用[Charles proxy](https://www.charlesproxy.com/)作为中间人服务器，最后演示一个有效的 SSL Pining 如何阻止中间人攻击。

Please note that the following codes are for educational purpose and they are not meant for use in production. Please consider using a mature solution from reputable libraries such as [okhttp](https://square.github.io/okhttp/3.x/okhttp/okhttp3/CertificatePinner.html) for your application.

请注意以下代码是用于教育目的，并不用于生产。请考虑为您的应用程序使用来自声誉良好的库的成熟解决方案，比如 [okhttp](https://square.github.io/okhttp/3.x/okhttp/okhttp/okhttp3/certificatepinner.html)。

```java
// Read the pinned certificate from local (i.e., assets folder)
// 读取本地固定证书
val inputStream = context.assets.open("google.crt")
val pinnedCertificate = CertificateFactory.getInstance("X.509")
    .generateCertificate(inputStream)

// Create a request to www.google.com
// 创建一个 www.google.com 的请求
val url = URL("https://www.google.com")
val httpsUrlConnection = url.openConnection() as HttpsURLConnection

// Establish the connection
// 建立连接
httpsUrlConnection.connect()

// Check the certificates and see if one of the server certificates
// matches the pinned certificate
// 检查证书，看看其中一个服务器证书是否与固定证书匹配
if (httpsUrlConnection.serverCertificates.contains(pinnedCertificate)) {
    // Open stream
    // 打开流
    httpsUrlConnection.inputStream
    Log.d("Pinning", "Server certificates validation successful")
} else {
    Log.d("Pinning", "Server certificates validation failed")
    throw SSLException("Server certificates validation failed for google.com")
}
```

The above codes are self-explanatory. The server certificates from `HttpsUrlConnection` are to be checked against with the local pinned certificate, the connection input stream will be open if there is a certificate match, and an `SSLException` will be thrown otherwise.

以上代码是不言自明的。将使用本地固定证书检查来自 “HttpsUrlConnection” 的服务器证书，如果证书匹配，连接输入流将打开，否则将抛出 “SSLException”。

I use openssl command to download the <www.google.com> certificate and save it as a pinned certificate in the assets folder. Just run this code and you will see message `Server certificates validation successful` in the logcat.

我使用openssl命令下载证书，并将其保存为固定在assets文件夹中的证书。只要运行这段代码，您将在logcat中看到消息“服务器证书验证成功”。

Now fire up Charles, and install Charles root certificate to the test device and configure test device proxy setting to the IP address of Charles service. With Charles root certificate being installed as trusted CA, Charles now acts as a man-in-the-middle, and all the certificate signed by Charles will be trusted by the client system by default. However, since we have pinned the legit <www.google.com> certificate in our demo application, the server certificate validation will fail in this case.

现在启动 Charles，将 Charles 根证书安装到测试设备，并将测试设备代理设置配置为 Charles 服务的 IP 地址。将 Charles 根证书安装为 trusted CA 后，Charles 现在充当中间人的角色，在缺省情况下，客户机系统将信任 Charles 签名的所有证书。但是，由于我们在演示应用程序中固定了 legit 证书，因此在这种情况下服务器证书验证将失败。

Run the test application with Charles proxy enabled, as expected, you will see `SSLException` with message `Server certificates validation failed for google.com` in the logcat.

运行启用了 Charles proxy 的测试应用程序，如预期的那样，您将在 logcat 中看到 “SSLException” 和消息 “Server certificates validation failed for google.com”。

## Further reading

## 扩展阅读

The purpose of this post is just to provide an entry-level introduction to SSL Pinning technique, you can read the [original proposal](https://www.paypal-engineering.com/2015/10/14/key-pinning-in-mobile-applications/) from Paypal engineering team, and learn more about the [bug](https://www.synopsys.com/blogs/software-security/ineffective-certificate-pinning-implementations/) that results in ineffective SSL Pinning in Java/Android.

本文的目的只是提供SSL固定技术的入门介绍，您可以阅读 Paypal 工程团队的[original proposal](https://www.paypal-engineering.com/2015/10/14/key-pinning-in-mobile-applications/) 了解更多关于导致 Java/Android 中SSL固定无效的 [bug](https://www.synopsys.com/blogs/software-security/ineffective-certificate-pinning-implementations/)。

Oct 23, 2018 （2018-10-23）

author: Zhang QiChuan

link: <https://medium.com/@zhangqichuan/explain-ssl-pinning-with-simple-codes-eaee95b70507>