---
title: LoongSon大牛的讲话
date: 2020-08-17 23:26:18
tags:
- 其他
categories:
- 其他
---


<center>
龙芯总公司研究员王洪虎老师的讲话，体会有感
</center>
<!-- more -->


# Loongson总公司大牛的讲话

> 2020.08.05总公司研究员王洪虎老师讲话，获益匪浅，记录一下，虽然未来不一定要去Loongson工作，但是和这样直率诚恳的技术大咖交流的机会，是十分难得的，所以专门记录了一下。

1. 关于生态
2. 关于项目
3. 关于结果
4. 关于我们
5. 关于解决问题
6. 关于团队与个人

## 关于生态

龙芯造国产的CPU，必然需要给下游的一些应用开发厂家、中间件厂家给予一个生态平台，这是非常重要的一个方面，**生态是从软件、硬件、核心库、中间件等等一系列CS配套产业**，打造一个生态是成片的

## 关于项目

完成一个项目，项目的目的是什么，过程是什么。。

以下举例就拿我们组目前的这个项目——数据恢复软件

1. 明确方向，制定需求。
   - 需求依然依然依然依然是一个项目的重中之重，完成一个项目，你得知道你完成的这个项目最终得完成什么样的需求，完成什么样的目标。
   - 做一个数据恢复软件，我们的需求可能就是，我们要完成U盘的数据恢复呢、还是硬盘的数据恢复呢、还是光盘等等的数据恢复呢，我们要找到我们要完成的目标需求是什么。
2. 定义设计方案。进行调研
   - 项目设计的方案，进行调研，了解这个项目的背景、等等情况。
   - 做数据恢复软件，我们就得去调研windows下是怎么数据恢复的，Linux下是怎么数据恢复的，有成品的项目软件需求，我们需要先去了解这些已经成型的技术，才方便我们完成和实现这个项目
3. 核心技术
   - 了解我们这个项目的核心技术，核心难点在什么方面首先我们就得清楚操作系统是如何存储一个数据的
   - 做一个数据恢复软件，就需要去了解各个文件的存储方式是什么，文件系统是什么，操作系统是如何存储一个数据的，如果要恢复一个数据，就得去找这个数据对应的表啊等等一系列的东西
4. 风险点
   - 在完成以上工作后，我们应该就对这个项目有了比较清晰的了解了，我们这个时间就应该对这个项目需要多长时间完成，完成后我们实现的大概是一个什么样子这个过程有了很多的了解

## 关于完成项目的过程中

完成一个项目的过程就是不断的积累：

1. 日志：日志是每天的一个小总结，记录今天完成的结果，记录面对的难点，根据今天的工作做一个总结
2. 固定的例会：组内良好的沟通是非常的必要的，这个例会不限于组的形式，可以找指导老师啊，可以问技术大牛呀等等，例会的时间也没有什么要求，可能是三五分钟啊，也可能是半个小时，根据项目我们可以几天开一次会啊等等
3. 周报：日志是对每天的一个小总结，一天可能忙来忙去，最后也不知道自己到底完成了什么东西，可以每周做一个周报，来总结这一周做了什么东西，
4. 寻求指导：遇到困难是必不可免的，要及时的去找人帮忙给出一个方向

## 关于结果

完成一个项目，我们最后肯定是要得到一个结果的，不管这个结果怎么样，我们都要对这个结果进行一个分析与总结

1. 固化：项目的结果最后一定要固化下来，落到实地
2. 测试大纲：测试大纲测试项目，这要覆盖我们当初做项目的需求，一项项分析我们是不是严格的完成了项目
3. 测试报告：测试完成一定要求有测试报告来总结
4. 完成后的工作总结：项目完成了，这个时候一定要做工作总结，工作总计要包含，在这个项目中我做过什么，我完成过什么，我解决过什么问题，什么问题困扰很久还没有解决，查漏而补缺，温故而知新

## 关于CS专业我们计算机学生

1. 源码：基本的编码能力是基础，是很重要的东西，这里面包括了很多很多东西，例如debug工具、数据结构、算法、操作系统，这是解决一个问题和找到项目的原因的基础
2. 二进制：不管我们使用什么语言，最后的结果都一定是转换为了机器语言，比如说Linux软件打包运行，包有两个大类，rpm包和deb包，解决一个问题，可能最后都要找到这个层次才能成功的解决
3. 质量：完成项目一定要有质量，做完与做好是两个层次。有以下四点
   - 功能是否完备呢？
   - 性能是否能再一次优化呢？
     - 算法优化：比如排序，有各种各样的的算法，我们是不是用另一个排序算法会更快更好
     - 程序优化：程序调优，改改源码，什么地方重构一下会更好呀，等等
     - 架构优化：使用不同的软件架构，最后得到的性能一定是不同的。(先了解，目前还是太菜了)
   - 稳定性：是否完备、健壮（这也是一个大的问题）
   - 兼容性
4. 文档：记录所学的东西，文档报告。我们经常会去百度呀，谷歌呀，我们查到CSDN的东西他们的文档，都是别人记录下来的文档，我们也要这么做，搞一个博客啊，进行记录
5. 有开源社区的经验，如果我们能力够了（当然远远不够），我们就可以去各样的开源社区，我们看他们的项目，发生了bug我们提交解决了这个bug，人家接收了我们的补丁，这就是一个很牛的事情了，至少可以表现出我们的能力是够的，面试也会+分的

## 关于解决问题的办法

中国航天是非常牛的，我们要解决一个问题，可以学习中国航天解决问题（人家叫问题归零）的办法，有五步：

1. 问题复现：重新构建出这个问题出现的环境，分析这个问题是从哪儿冒出来的
2. 定位准确：对于冒出的这个问题要定位到准确的位置
3. 基理清楚：这就需要用到我们的基础的知识了，找到这个问题我们需要知道这个问题是为什么出来的，要解决他我们就要了解这其中的知识
4. 措施有效：采取有效的、方便的方式
5. 举一反三：想想这个问题是不是还会在其他地方出现呢，之类的

## 我的想法

### 关于技术与业务

很早之前有看过一个面试视频，其中讲到了技术与业务的关系，说实话，平常我也觉得技术是非常重要的东西，但是最近真的感觉到业务的重要性，完成一个业务，技术只是实现业务的一个方法，业务好的人会知道要做些什么，技术好的人可以更好的去做。这两个是不相同的，不冲突的

### 关于团队与个人

之前加入云顶被问到过一个问题，将忠诚、个人、团队进行一个排序。

当时我思索了一番，给学长这么一个答案：忠诚 > 个人 > 团队

学长问我为什么？ 我回答：忠诚肯定是第一位的，关于个人和团队，我把个人排到前面，是因为我觉得如果不提高个人技术的话，可能在团队中像是一个混子吧。

现在呢，如果要让我重新排的话，我会把团队放到前面吧，因为越牛的团队，越强的企业，是不可能依赖于一个人的个人技术的，所以团队是更加重要的东西。

em。。感觉自己理解的越来越深刻了，加油

:wq

