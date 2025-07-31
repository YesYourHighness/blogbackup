---
title: Flink SQL
date: 2024-02-24 19:59:43
tags: 
- Flink
categories: 
- Flink
---

 <center>
引言：常用于实时数据流处理，但其实流批一体。
</center>

<!--more-->

# Flink SQL

详细使用方式请参考：[Flink 官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/table/sql/create/#columns)

>Flink是常用的用来解决实时数据流处理的工具，但其本身对流式数据、批式数据都可以处理（流批一体，Flink1.11）

本文只聚焦于Flink处理流式数据。

常见使用：实时数仓、ETL、CEP（复杂数据流处理）、双流 Join（两个不同的数据流按照某种条件进行关联操作，以产生一个包含两个流中元素关联信息的新流）、维表 Join等。

Flink的使用可以归为三步：Source->Deal->Sink

1. 规定原表Source（假设数据为水流，这就是一个水龙头）
   - 数据的源头可能是：MQ、HIVE、MYSQL等
2. 进行处理操作、聚合操作：对Source流来的数据流进行处理，
3. 写入下游表Sink
   - 写入的下游可能是：MQ、HTTP（每条数据触发一次服务调用）

# Flink时间属性
Flink支持三个时间属性：
- **事件时间(Event Time)**：代表数据在产生时的时间，格式是TIMESTAMP、单位是ms（13位数字）
- 摄取时间(ingestion time)： Flink 读取事件时记录的时间（一般不关心这个时间）
- **处理时间(processing time)**： Flink pipeline 中具体算子处理事件的时间
事件、处理时间想要正常工作，必须得在source标明，如下SQL：
```sql
CREATE TABLE test_stream( 
  a INT, 
  b BIGINT, 
  `TIMESTAMP` BIGINT, -- 数据源的时间戳字段
  ts AS LONG_TO_TIMESTAMP(`TIMESTAMP`), -- 此处就标明了事件时间
  WATERMARK FOR ts AS ts - INTERVAL '5' SECOND 
) WITH ( 
  type = 'kafka', 
  ... 
);
```

# WATERMARK

上述的SQL，除了事件事件外，还有一个WATERMARK

WATERMARK是一个时间戳，含义是：**当一个Event Time小于WATERMARK值的时候将不再出现**

注意：

- 只有基于 event time 的处理需要用到 watermark。（只要使用了事件时间，就要加watermark）
- 基于 process time 的处理不存在乱的问题
> 为什么要有WARTERMAKR？

理想情况：每一个packet随时间依次到达，这种情况下，我们随便切一下，都是当前时间最早的一个packet。
实际情况：包是乱序到达的，这意味着，我们不能确定当前的包是不是最早的，**因此我们只能在一段时间内，拿一个主意，认定某一个是最早的数据**。（watermark的作用即定义了何时停止等待较早的时间）

```sql
-- 数据乱序到达：
 → 23 19 22 24 21 14 17 13 12 15 9 11 7 2 4 →
```
比如，当"4"数据包到来的时候，4就是最早的数据吗？我们就要从4开始之后的计算处理吗？

从上帝视角看，2才是，但是2真的是最早吗？如果之后1出现了怎么办？

Flink是实时计算，我们必须在一定的时间内拿出一个主意进行计算，因此，watermark规定多长时间停止等待最早的数据出现。

# 滑动窗口
滑动窗口：一个算法概念，只透出窗口内的值，窗口在数据流上滑动。
![滑动窗口](http://img.yesmylord.cn//image-20240224200445641.png)
滑动窗口有两个变量：

- 步长（window slide）：每次窗口移动的距离
- 窗口大小（window size）：窗口大小
```sql
-- HOP(时间属性列, 滑动步长, 窗口大小)
hop(ts, INTERVAL '5' MINUTE, INTERVAL '5' MINUTE); --申明滑动窗口
```
# 滚动窗口
类似于翻书，**滚动窗口相当于`滑动窗口步长=窗口大小`的特殊情况**
![滚动窗口](http://img.yesmylord.cn//image-20240224200604589.png)

```sql
-- TUMBLE(时间属性字段, 窗口大小)
TUMBLE(ts, INTERVAL '10' MINUTE) --申明滚动窗口
```
> 滑动窗口和滚动窗口在工程上如何使用？

现在我们要实时统计24h内的某个商品的数量，并且当数量大于1w时进行一些通知操作：

我们可以结合滚动窗口和滑动窗口，使用**滚动窗口减少计算量，滑动窗口完成状态判断**

- 滚动窗口：我们可以定义一个5min的滚动窗口，每5min聚合一次计算这期间的数量。
- 滑动窗口：滑动窗口统计所有滑动窗口的聚合值，判断是否超过1w，进行对应操作

注意：滚动窗口或是滑动窗口必须是最后一步处理操作，之后必须写入sink，因此如果想结合使用，可以将滚动窗口的值写入sink，比如：

1. source(源数据的mq)->滚动窗口->sink1
2. source(sink的mq)->滑动窗口->sink2

sink1和sink2可以使用一个字段进行区别，这样我们就可以复用同一个mq来存储两个数据的sink