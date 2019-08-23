---
title: "The Internet changes: HTTP/3 will not use TCP anymore(因特网变更：HTTP/3 将不再使用 TCP)"
date: 2019-04-14 11:02:24
tags:
- http
categories:
- 翻译
comments: true
---

## QUIC is the protocol underlying the next version of HTTP

## QUIC 是下一个版本 HTTP 的基础协议

the Internet Engineering Task Force (**IETF**) has revealed that the third official version of HyperText Transfer Protocol (HTTP) **will not use TCP anymore**. Instead, it will run over the QUIC protocol first developed by Google back in 2012.

因特网工程工作组（Internet Engineering Task Force 简称：**IETF**）透露，第三个官方版本的超文本传输协议（HyperText Transfer Protocol 简称：HTTP）**将不再使用 TCP**。反而，它将运行 2012 年由谷歌首次开发的 QUIC 协议。

## WHAT IS QUIC?

## 什么是 QUIC？

**Quick UDP Internet Connections (QUIC)** is, as its name states, a transport layer protocol based on multiplexed UDP connections. In fact, QUIC uses a combination of TCP + TLS + SPDY over UDP with several enhancements with respect to the current HTTP/2 over TCP implementation.

**快速 UDP 因特网连接（Quick UDP Internet Connections 简称：QUIC）** 是一种基于多路复用的 UDP 连接的传输层协议。事实上，QUIC 使用了 TCP + TLS + SPDY over UDP 与当前 HTTP/2 over TCP 实现的几个增强功能的组合。

The **IETF has been working on a standardized version of Google’s QUIC** since 2016, and it was recently that they announced their intention to include it for the new HTTP/3 version. However, the IETF QUIC version already diverged significantly from the original QUIC design.

IETF 自 2016 年以来一直致力于开发标准化的 Google QUIC 版本，最近他们宣布打算将其用于新的 HTTP/3 版本。然而，IETF 的 QUIC 版本已经与原来的 QUIC 设计有了很大的不同。

![](/img/review/1_E90CoPNTa24ekQ85LEyrgg.png)
Position of HTTP/3 and QUIC in the protocol stack
HTTP/3 和 QUIC 在协议栈中的位置

QUIC protocol aims for **simplicity and speed while maintaining security** thanks to the TLS 1.3 encryption. They developed a more efficient protocol in terms of connection establishment and data transfer. [According to Google](https://docs.google.com/document/d/1gY9-YNDNAB1eip-RTPbqphgySwSNSDHLq9D5Bty4FSU/edit), **QUIC handshakes frequently require zero roundtrips** before sending payload, as compared to 1–3 roundtrips for TCP+TLS. Actually, the first connection ever requires one roundtrip and the followings will work with zero.

由于采用了 TLS 1.3 加密，QUIC 协议旨在简化和提高速度，同时保持安全性。他们在建立连接和数据传输方面开发了一种更有效的协议。[据谷歌称](https://docs.google.com/document/d/1gY9-YNDNAB1eip-RTPbqphgySwSNSDHLq9D5Bty4FSU/edit)，与 TCP + TLS 的 1-3 次往返相比，Quic握手在发送有效负载之前通常需要**0次往返**。实际上，第一次连接需要一次往返，下面的内容将不往返。

Furthermore, it **deals better with packet loss than the current TCP**. Every retransmitted packet consumes a new sequence number, hence eliminating ambiguities and preventing losses from causing RTO. As [Jana Iyengar from IETF](https://docs.google.com/presentation/d/15e1bLKYeN56GL1oTJSF9OZiUsI-rcxisLo9dEyDkWQs/edit#slide=id.g5b4208ab1_0_272) states, QUIC is not only a redefinition of the Internet transport layer but a **reinvention to do the transport right**.

此外，它**比当前的 TCP 更能处理数据包丢失**。每一个重新传输的包都会消耗一个新的序列号，因此消除了模糊性，并防止了造成 RTO 的损失。正如[来自 IETF 的 Jana Iyengar](https://docs.google.com/presentation/d/15e1bLKYeN56GL1oTJSF9OZiUsI-rcxisLo9dEyDkWQs/edit#slide=id.g5b4208ab1_0_272) 所说，QUIC 不仅是对互联网传输层的重新定义，而且是对实现传输权的重新定义。

At the moment, only 1.2 % of the top websites support [QUIC](https://w3techs.com/technologies/details/ce-quic/all/all), but they are generally high-traffic sites: almost every Google service supports their own QUIC protocol.

目前，只有 1.2% 的顶级网站支持 QUIC，但它们通常是高流量的网站：几乎每个谷歌服务都支持自己的 QUIC 协议。

## IS QUIC SECURE?

## QUIC 是安全的？

QUIC first development included its own encryption. However, it was just a temporary implementation destined to be replaced by **TLS 1.3 as described by the IETF**.

QUIC 的第一个开发包括它自己的加密。然而，正如 IETF 所描述的，它只是一个临时的实现，注定要被 TLS 1.3 所取代。

Actually, the connection establishment strategy of QUIC is based on the combination of crypto and transport handshake.

实际上，QUIC 的连接建立策略是基于加密和传输握手的结合。

> QUIC relies on a combined cryptographic and transport handshake to minimize connection establishment latency.
>
> QUIC 依赖于密码和传输握手的组合，以最小化连接建立延迟。

![](/img/review/1_cWUBE5wABDOeBZvajVy0eQ.png)
Integration of QUIC and TLS, adapted from the IETF’s draft of “Using Transport Layer Security (TLS) to Secure QUIC”.
QUIC 和 TLS 的集成，改编自 IETF 的 “使用传输层安全性（Transport Layer Security 简称：TLS）保护 QUIC” 草案。

With QUIC, Everything will be encrypted by default. Nevertheless, there are indeed **security risks** with QUIC as with any other technology.

使用 QUIC，默认情况下将加密所有内容。然而，与其他技术一样，QUIC 确实存在**安全风险**。

The work of [Robert Lychev and Samuel Jero](https://www.cc.gatech.edu/~aboldyre/papers/quic.pdf) in 2015 reported several weaknesses of the protocol. QUIC **performance can be degraded by attacks** like the Server Config Replay Attack. However, **the confidentiality and the authenticity of the data seem to be properly secured** according to their security model and tests.

[Robert Lychev 和 Samuel Jero](https://www.cc.gatech.edu/~aboldyre/papers/quic.pdf) 在 2015 年的研究报告了该协议的几个弱点。像服务器配置重放攻击这样的攻击会**降低 QUIC 性能**。然而，根据安全模型和测试，数据的机密性和真实性似乎得到了适当的保护。

[Understanding Event-Driven Architectures (EDA): the paradigm of the future](https://medium.com/drill/understanding-event-driven-architectures-eda-the-paradigm-of-the-future-7ae632f056bb)

[理解事件驱动的architectures（EDA）：“范式”](https://medium.com/drill/understanding-event-driven-architectures-eda-the-paradigm-of-the-future-7ae632f056bb)

[BITCOIN case study: applying basic Digital Signal Processing into financial data](https://medium.com/drill/btc-case-study-applying-basic-digital-signal-processing-into-financial-data-ec34cd47c77b)

[BITCOIN 案例研究：将基本数字信号处理应用于金融数据](https://medium.com/drill/btc-case-study-applying-basic-digital-signal-processing-into-financial-data-ec34cd47c77b)

If you want to learn more about QUIC and how it will be integrated into the next version of HTTP, I strongly recommend you to check the official documentation from Google and the drafts published by IETF. You can find them in the bibliography of this article!

如果您想了解更多关于 QUIC 的信息，以及如何将其集成到下一个 HTTP 版本中，我强烈建议您查看来自 Google 的官方文档和 IETF 发布的草稿。你可以在这篇文章的参考书目中找到它们！

Innovation is always spinning forward. Just like a Drill.

创新总是向前旋转。就像一个练习。

## BIBLIOGRAPHY

## 参考文献

[1] ”QUIC, a multiplexed stream transport over UDP — The Chromium Projects”, Chromium.org. [Online]. Available: https://www.chromium.org/quic. [Accessed: 18- Nov- 2018]

[2] M. Thomson, “draft-ietf-quic-transport-16 — QUIC: A UDP-Based Multiplexed and Secure Transport”, Tools.ietf.org, 2018. [Online]. Available: https://tools.ietf.org/html/draft-ietf-quic-transport-16. [Accessed: 18- Nov- 2018]

[3] S. Turner, “draft-ietf-quic-tls-03 — Using Transport Layer Security (TLS) to Secure QUIC”, Tools.ietf.org, 2018. [Online]. Available: https://tools.ietf.org/html/draft-ietf-quic-tls-03#section-3. [Accessed: 18- Nov- 2018]

[4] E. Rescorla, “The Transport Layer Security (TLS) Protocol Version 1.3”, Tools.ietf.org, 2018. [Online]. Available: https://tools.ietf.org/id/draft-ietf-tls-tls13-23.html. [Accessed: 18- Nov- 2018]

[5] R. Lychev, S. Jero, A. Boldyreva and C. Nita-Rotaru, “How Secure and Quick is QUIC? Provable Security and Performance Analyses”, 2015. [Online]. Available: https://www.cc.gatech.edu/~aboldyre/papers/quic.pdf. [Accessed: 18- Nov- 2018]


Nov 18, 2018 （2018-11-18）

author: Telmo Subira Rodriguez

link: <https://medium.com/drill/the-internet-changes-http-3-will-not-use-tcp-anymore-427e82eeadc0>