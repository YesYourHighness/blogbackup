---
title: VPN
date: 2023-07-20 17:13:39
tags:
- 计算机网络
- VPN
categories:
- 计算机网络
---

<center>
引言：Virtual Private Network，也称为网络隧道
</center>
<!-- more -->


# VPN

> Virtual Private Network 虚拟私人网络，通过**加密和隧道技术**在公共网络上创建一条安全的、私密的通信通道

## VPN出现的背景

- 在1996年，互联网还处于HTTP交互时代，而HTTP是明文传输的，传输的数据容易被窃取或篡改

- 远程办公的需要，而企业或学校内部的网络无法被外网连接（站点到站点的通信）

## VPN原理

假设现在中国分公司上班的Alice需要访问在美国的总公司：

1. Alice将请求发送给**VPN服务器**（或称为VPN集线器、VPN网关）
2. VPN服务器会将Alice的源地址与目的地址封装起来，发送给在美国的VPN服务器
3. 美国的VPN服务器拆包，发送到真正要发送的目的地址

这样在外人（ISP）看来，只是两个VPN服务器在通信，就达到了隐私的目的。

VPN还需要保证数据加密（AES等）、数据完整（HASH校验）、以及身份认证（RSA、用户名密码、或是PKI）

## VPN主要的核心两大协议

VPN的最大作用就是安全，他使用两种协议来保证安全：

- SSL/TLS（Secure Socket Layer/Transport Layer Security）
- IPSec（Internet Protocol Security）

### SSL/TLS

即HTTPs中的s，SSL（Secure Socket Layer）是TLS（Transport Layer Security）的前身。**TLS是SSL的升级版**，它是在SSL的基础上发展而来的，提供更强大的加密和安全性。

SSL/TLS建立连接在TCP三次握手建立连接之后，服务器就会返回给浏览器一个数字证书，浏览器向CA机构验证证书完整性，然后混合使用对称加密与非对称加密：

- 通信建立前：采用**非对称秘钥加密**的方式交换**会话秘钥**，后续就不再使用非对称加密
- 通信过程中：全部使用**对称加密**来加密明文数据

### IPSec

TLS是位于传输层与应用层之间的协议，这意味着，在应用层的下三层，数据依旧是透明的。

IPSec是网络层的协议（与IP同一层）负责加密IP包，保证网络层到网络层的通信也是加密后的。

## 使用VPN是否万无一失

> 问题1：既然VPN保密了我们访问的目的地址，就真的万无一失了吗？

有这几种情况可能暴露我们访问的目的地址：

1. 浏览器的隐私设置：浏览器保存的浏览历史、Cookie、缓存
2. 不靠谱的VPN提供商：VPN服务商出卖了我们（免费的或是不可信的VPN经常这么做，他们出卖数据给第三方）
3. DNS解析：DNS解析请求暴露我们将要访问的网站，好的VPN服务商会使用自己的DNS服务器。

