---
title: IO多路复用
date: 2024-04-26 19:16:35
tags:
- 操作系统
categories:
- 操作系统
---

<center>
    引言：select、poll、epoll模型
</center>

<!-- more -->

# IO多路复用

这篇文章再复习一下select、poll、epoll，三种最经典的IO多路复用模型，是NIO实现的基础。

他们模型共同点都是轮训文件描述符，区别在于具体的一些实现的细节。下面给出具体的区别：

## select

select模型的实现主要如下：

```c
int select(
    int maxfdp1,
    fd_set *readset,
    fd_set *writeset,
    fd_set *exceptset,
    const struct timeval *timeout)
```

主要是两个结构：

- `maxfdp1`：待轮训的文件描述符的个数。
- `fd_set`：分别表示读事件、写事件、异常事件；一个bitmap，长度只有1024位。

缺点：

- `fd`数量有限制，最多只有1024个
- 每次轮训，都需要copy 3个`fd`到内核，存在频繁的用户态和内核态的copy
- `fd_set`不能重用
- 每次遍历都得遍历所有的`fd`

## poll

```c
struct pollfd {
　　int    fd; // 要轮询的文件描述符fd
　　short  events; // 关心的fd事件：普通数据可读、优先级带数据可读等等
　　short  revents;// fd上发生的事件
};
```

poll主要改进了两点：

- 引入了`pollfd`结构体，不再局限于1024个
- `pollfd`可以重用

缺点：

- 依然需要频繁的用户态和内核态的copy
- 需要遍历所有的`pollfd`

## epoll

epoll模型由三个函数构成：

- `epoll_create`：创建`epollfd`对象
  - `epollfd`对象主要由两个结构组成
    - 红黑树：存储关心的`fd`
    - 双向链表：存储发生相关事件的`fd`（关心事件触发，就会自动填入这个双向链表）
- `epoll_ctl`：将`epollfd`copy到内核，并注册关心的事件
  - 将copy过程提前到此阶段，而且在整个轮训过程中，只需要copy这一次！
  - 关心事件的触发方式有两种
    - 水平触发LT（默认）：**事件可处理可不处理**
    - 边缘触发 ET：**事件必须处理**
- `epoll_wait`：调用系统调用，陷入内核等待



因此，前面select、poll的缺点就全被搞定了：

- 1024上限？不再有了，我们有`epollfd`对象
- 每次轮训所有？不轮训所有，只轮训事件触发的对象
- 每次都需要用户态与内核态的copy？只需要在`epoll_ctl`时copy一次即可



> 问题一：水平触发和边缘触发的区别是什么？

水平触发情况下，如果事件没有处理，那么下次`epoll_wait`依然会返回这个事件的`fd`；

边缘触发情况下，如果没有立即处理，那么下一次的返回不一定会有该事件，只能等到该事件再次触发。

因此：

- 水平触发适用于处理长时间处于可读或科协的socket
- 边缘触发更适合于要求效率更高的前提下

> 问题二：select、poll 与 epoll的区别是什么？

- select、poll：都是主动轮训
- epoll：被动轮训，所谓被动轮训指的是当数据准备好之后，就会把就绪的fd加入双向链表中（存在一个回调函数，触发就会加入到双向队列）

