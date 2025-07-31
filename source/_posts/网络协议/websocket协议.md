---
title: WebSocket协议
date: 2023-09-25 21:23:39
tags:
- 计算机网络
- WebSocket
categories:
- 计算机网络
---

<center>
引言：WebSocket——基于TCP的全双工实时通讯
</center>

<!-- more -->

# WebSocket

## WebSocket的应用场景

> WebSocket：**一种在单个TCP连接上进行全双工通信的协议**。
>
> 它最初被设计用于在Web浏览器和Web服务器之间提供实时、双向通信的能力，但现在已经被广泛用于各种应用程序和领域，包括在线游戏、聊天应用、金融交易系统和实时协作工具等

常用在以下场景：

- **在线实时聊天室**
- **多人在线游戏 (MMOGs)**：魔兽世界、星际争霸2、Apex、我的世界等等。
- **协作工具**：在线协作工具，如实时白板、远程桌面共享和协同编辑，需要实时同步多个用户之间的操作。
- **实时监控和仪表板**：WebSocket可用于实时监控系统状态、服务器性能、传感器数据等，并将这些数据实时传送给操作人员。

## 如何做到实时通信？

在没有websocket协议之前，有这么几种**伪实时通信**的方式，以扫描二维码登录这件事为例：

1. **浏览器轮询**：浏览器定期请求服务器，获得响应

微信公众平台的扫码登录就是使用这种方式，每隔1~2s就发送一次http请求，等待响应

![微信公众平台二维码登录](http://img.yesmylord.cn//image-20230925220948254.png)

这样的登录方式简单粗暴，但是用户在扫码后，会有等待1~2s的体验，而且频繁的http请求，对后台也是不小的负担。

2. **服务器轮询**：将用户请求的超时时间设置长一点，比如30s，后端一直轮询，如果客户扫描二维码（扫描后会发送请求），就返回响应

这种做法，减少了HTTP请求次数，但是后台一直轮询，对服务器的压力也比较大

## WebSocket协议

> WebSocket协议基于运输层TCP协议，WebSocket的握手阶段基于HTTP协议，数据传输是WebSocket自己的部分。

### 握手部分

在TCP连接建立后，如果想要使用WebSocket协议，需要使用**HTTP协议进行握手**

1、**HTTP握手请求头**：

```http
GET ws://localhost/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

- **Connection:  Upgrade**：表明浏览器想升级协议
- **Upgrade:  websocket** ：表明升级链接为websocket

- **Sec-WebSocket-Key**：这是一个Base64编码的随机密钥，用于安全验证WebSocket连接请求的合法性
- **Sec-WebSocket-Version**: 表示客户端支持的WebSocket协议版本，通常是13

2、**HTTP握手响应头**：

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

- **HTTP/1.1 101 Switching Protocols**：状态码 "101 Switching Protocols" 表示切换协议成功。
- **Upgrade: websocket** 和 **Connection: Upgrade**: 这两个头部字段表示服务器同意升级连接到WebSocket协议。
- **Sec-WebSocket-Accept**: 这是服务器生成的Base64编码的密钥，用于验证WebSocket连接请求的合法性。服务器使用客户端发送的Sec-WebSocket-Key来计算这个值。
- **Sec-WebSocket-Protocol**: (可选) 如果客户端请求了特定的子协议，服务器可以在这里指定所选择的子协议。

### 数据交互部分

（本节图源来自：[小白debug](https://www.bilibili.com/video/BV1684y1k7VP/?spm_id_from=333.337.search-card.all.click&vd_source=c8709f8826bf296abab8aeee72b0e338)）

在HTTP握手建立后，就可以进行websocket通信了，websocket中的数据包称为**帧**

![通信流程](http://img.yesmylord.cn//image-20230926152726611.png)

数据帧的格式如图：只需要关注几个字段

![数据帧格式](http://img.yesmylord.cn//image-20230926153237484.png)

- **4位的opcode**：表示数据的类型，1是string帧；2表示二进制帧（即bytes帧）；8表示关闭连接帧。
- **四个payload字段**：它的意思是，一开始只读最开始的7bit，如果是0-125，那么表示数据的长度7bit就可以表示完成；如果是126（0X7E）就继续向后读，之后的几个段同理。

## 相关问题

> 1、为什么有了HTTP协议，还要有Websocket协议？

HTTP是半双工协议，是为了提供浏览服务的，设计之初没打算支持交互。

WebSocket协议是专门用来提供交互的。

> 2、WebSocket和HTTP的关系是什么？

WebSocket握手需要使用HTTP协议，升级完成后，就与HTTP毫无关系了。











