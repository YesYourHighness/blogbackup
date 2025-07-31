---
title: RocketMq
date: 2024-12-29 00:19:39
tags:
- 消息队列
categories: 
- 消息队列
---

<center>
引言：应该是24年最后一篇了，学一下RocketMq
</center>

<!-- more -->

# RocketMq4.x

Rocket的4.x版本，与5版本的区别还是比较大的，4.x的使用更广泛一些，本文介绍4.x的基本使用方式。

- 本文代码使用：https://github.com/yudaocode/SpringBoot-Labs/tree/master/lab-31
- 文章内容参考官网4.0版本以及[此篇文章](https://www.iocoder.cn/Spring-Boot/RocketMQ/?github)

## 初期准备

### mq集群

需要搭建一个mq环境，基本的结构要有nameserver和一个broker，我直接使用docker-compose快速搭建：[官方文档](https://rocketmq.apache.org/zh/docs/4.x/quickstart/03quickstartWithDockercompose)

额外创建了一个broker.conf做卷

```yaml
# broker 对外提供服务的ip，如果是公网，则公网ip，如果本地测试，则本地机ip
brokerIP1 = 127.0.0.1
# Broker 的名称1
brokerName = broker-a
# 在集群中对 Broker 的唯一标识。值 0 通常表示这是一个主 Broker
brokerId = 0
# 指定什么时候删除旧的提交日志。值 04 表示在每天的凌晨 4 点删除旧日志
deleteWhen = 04
# 指定提交日志文件的保留时间（以小时为单位）。这里的 48 意味着提交日志会保留 48 小时，然后才会被删除
fileReservedTime = 48
# 定义 Broker 在集群中的角色。ASYNC_MASTER 表示该 Broker 作为主 Broker，并会将消息异步复制到从 Broker
brokerRole = ASYNC_MASTER
# 定义提交日志的刷新模式。ASYNC_FLUSH 表示数据会异步刷新到磁盘，以提高性能。
flushDiskType = ASYNC_FLUSH
# 如果设置为 true，则启用消息属性过滤。这允许你除了通过主题和标签过滤消息外，还可以通过消息的属性进行过滤
enablePropertyFilter=true
# 开启自动创建主题
autoCreateTopicEnable=true
```

用到的compose文件：

```yaml
version: '3.8'

services:
  namesrv:
    image: apache/rocketmq:4.9.6
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    networks:
      - rocketmq
    command: sh mqnamesrv

  broker:
    image: apache/rocketmq:4.9.6
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    environment:
      - NAMESRV_ADDR=rmqnamesrv:9876
    volumes:
      - ./broker.conf:/home/rocketmq/rocketmq-4.9.6/conf/broker.conf
    depends_on:
      - namesrv
    networks:
      - rocketmq
    command: sh mqbroker -c /home/rocketmq/rocketmq-4.9.6/conf/broker.conf

networks:
  rocketmq:
    driver: bridge
```



### rocket-console

[github地址](https://github.com/apache/rocketmq-dashboard)，一个mq的看板，这里原本我想也跑到docker上，但是我改掉nameserver的ip也无法访问，因此就拉下来代码跑在宿主机上面了。

![看板](https://img.yesmylord.cn//image-20241228160223573.png)

## 工作流程

![工作流程](https://img.yesmylord.cn//02.png)

1. 启动NameServer，NameServer监听端口（默认为9876），等待连接
2. 启动Broker后，Broker会与所有的NameServer建立长连接，30s心跳一次
3. 建立Topic，指定该Topic的存储在哪些Broker节点（可选，可以动态指定）
   1. Producer/Consumer(Client)发送消息前，需要先与NameServer建立连接，获取路由信息（即Topic的queue与broker地址的映射关系，因为消息要发送到一个Topic下的某一个queue上，而queue是存放在某一个broker上的），路由信息会缓存在本地，30s更新一次
4. Producer生产消息、Consumer消费消息

## 普通消息

>  [官网原文](https://rocketmq.apache.org/zh/docs/4.x/producer/02message1)

消息有三种发送方式，**同步、异步和单向传输**，前两种消息类型都有返回，单向传输只发送请求不等待应答

```java
@Autowired
private RocketMQTemplate rocketMQTemplate;

public SendResult syncSend(Integer id) {
    Demo01Message message = new Demo01Message();// 一个消息类内部定义了TOPIC、ID，无需关注
    message.setId(id);
    // 同步发送消息，具有返回值SendResult
    return rocketMQTemplate.syncSend(Demo01Message.TOPIC, message);
}

public void asyncSend(Integer id, SendCallback callback) {
    Demo01Message message = new Demo01Message();
    message.setId(id);
    // 异步发送消息，返回值通过回调函数获取SendCallback
    rocketMQTemplate.asyncSend(Demo01Message.TOPIC, message, callback);
}

public void onewaySend(Integer id) {
    Demo01Message message = new Demo01Message();
    message.setId(id);
    // oneway 发送消息，没有返回值
    rocketMQTemplate.sendOneWay(Demo01Message.TOPIC, message);
}
```

- `RocketMQTemplate`是官方提供的模版类，可以直接使用它发送消息。本质会创建一个 `DefaultMQProducer` 生产者 `producer` ，所以`RocketMQTemplate`后续的各种发送消息的方法，都是使用它。

- 异步方式的回调函数有两个接口，分别处理成功和失败两种情况：

  - ```java
    new SendCallback() {
        @Override
        public void onSuccess(SendResult result) {
            logger.info("[testASyncSend][发送编号：[{}] 发送成功，结果为：[{}]]", id, result);
        }
        @Override
        public void onException(Throwable e) {
            logger.info("[testASyncSend][发送编号：[{}] 发送异常]]", id, e);
        }
    }
    ```



消费一般会使用监听器的方式，而且通常要确保**一个消费者分组，仅消费一个 Topic**

```java
@Component
@RocketMQMessageListener(
        topic = Demo01Message.TOPIC,
        consumerGroup = "demo01-consumer-group-" + Demo01Message.TOPIC
)
public class Demo01Consumer implements RocketMQListener<Demo01Message> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void onMessage(Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```

- `@RocketMQMessageListener`注解用来标记该实例属于哪一个消费组、可以消费哪一个topic
- 实现`RocketMQListener`接口，重写`onMessage`方法来对消息进行操作

> 为什么要保证**一个消费者分组仅消费一个 Topic**？

1. 单一职责要求，消费者分组职责单一便于维护与理解
2. **每个消费者分组独占一个线程池**，因此可以让多个Topic隔离在不同的线程池

假设我们让同一个消费者分组消费A、B两个Topic，假设A消费的很慢，执行时间就会长，就导致影响了B的消费（因为同一个线程池去消费A、B）

## 批量消息

`DefaultMQProducer`支持批量的生产发送，`RocketMQTemplate`也实现了批量发送

```java
public <T extends Message> SendResult syncSend(String destination, Collection<T> messages, long timeout) {
    // ... 省略具体代码实现
}
```

批量发送要注意：

- 一批消息的Topic需要相同
- **批量消息的大小不能超过 1MiB**（否则需要自行分割）

聚成一批以后进行发送，可以增加吞吐率，并减少API和网络调用次数。

批量消息与普通消息的消费是一样的。

## 定时消息

### 定时消息（延迟消息）

定时消息，指的不是生产定时定点的发往broker，而是定时定点的被消费。

只有到达要求的时间后，该消息才可以被消费。

（rocketmq不支持任意时间精度的定时消息，固化了 18 个延迟级别，支持1s到两小时之间的大概延迟时间）

如果要求高精度的定时消息，可以借助mysql+job，或者是使用DDMQ（单条消息设置精确到秒级的延迟时间）

| 延迟级别 | 时间 | 延迟级别 | 时间 | 延迟级别 | 时间 |
| :------- | :--- | :------- | :--- | :------- | :--- |
| 1        | 1s   | 7        | 3m   | 13       | 9m   |
| 2        | 5s   | 8        | 4m   | 14       | 10m  |
| 3        | 10s  | 9        | 5m   | 15       | 20m  |
| 4        | 30s  | 10       | 6m   | 16       | 30m  |
| 5        | 1m   | 11       | 7m   | 17       | 1h   |
| 6        | 2m   | 12       | 8m   | 18       | 2h   |

生产上的特点就是需要指定延迟级别。

```java
// 同步
rocketMQTemplate.syncSend(Demo03Message.TOPIC, message, 30 * 1000, delayLevel);
// 异步
rocketMQTemplate.asyncSend(Demo03Message.TOPIC, message, callback, 30 * 1000, delayLevel);
```

消费时会在规定的级别时间后才可以被消费，比如指定了延迟级别为5，那么这条消息只有1分钟后才会被消费

### 消费重试

消息消费失败时，Rocketmq会有消费重试机制，重新投递消息给consumer。

默认情况下达到16次重试次数仍然失败的话，该消息会进入**死信队列**（Dead-Letter Message Queue）

> 为什么重试16次？

消费重试是基于定时消息来实现，第一次重试消费按照延迟级别为3（10s）。所以，3~18共有16次。

注意：只有**集群消费模式才有消息重试**。

## 消息模型

### 集群消费

- 一个消费组下的消费者会平均消费一个Topic下的所有消息（这就是集群消费，有点像负载均衡）

（比如1个消费组有2个消费者，那么假设同一个Topic下有10条消息，那么2两个消费者各自消费5个）

- 订阅同一个Topic的不同消费组会得到所有消息

（比如一个Topic有10个消息，有两个消费组订阅了，那么两个消费组都会得到10条消息）

比如再创建一个`demo01-A`，两个消费组都会获得到 `Demo01Message.TOPIC`的所有消息

```java
@Component
@RocketMQMessageListener(
        topic = Demo01Message.TOPIC,
    // 这里的消费组与上一个不同
        consumerGroup = "demo01-A-consumer-group-" + Demo01Message.TOPIC
)
public class Demo01AConsumer implements RocketMQListener<MessageExt> {

    private Logger logger = LoggerFactory.getLogger(getClass());
    @Override
    public void onMessage(MessageExt message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```

这种特性可以让我们对同一个结果展开多个业务，比如一个游戏升级后，A消费组可以专注于增加血量、B消费组可以专注于增加下一次升级所需的阈值等等。

### 广播消费

同一个消费组group下的每一个实例都会接收到全部的消息

比如：在应用中，缓存了数据字典等配置表在内存中，可以通过广播消费，实现每个应用节点都消费消息，刷新本地内存的缓存

```java
@Component
@RocketMQMessageListener(
        topic = Demo05Message.TOPIC,
        consumerGroup = "demo05-consumer-group-" + Demo05Message.TOPIC,
        messageModel = MessageModel.BROADCASTING // 设置为广播消费
)
public class Demo05Consumer implements RocketMQListener<Demo05Message> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void onMessage(Demo05Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```

这里贴一点`@RocketMQMessageListener`注解的常用属性：

```java
// Consumer 所属消费者分组
String consumerGroup();

// 消费的 Topic
String topic();

// 选择器类型。默认基于 Message 的 Tag 选择。
SelectorType selectorType() default SelectorType.TAG;

// 选择器的表达式。
String selectorExpression() default "*";

// 消费模式。可选择并发消费，还是顺序消费。 concurrently or orderly.
ConsumeMode consumeMode() default ConsumeMode.CONCURRENTLY;

// 消息模型。可选择是集群消费，还是广播消费。
MessageModel messageModel() default MessageModel.CLUSTERING;

// 消费的线程池的最大线程数
int consumeThreadMax() default 64;

// 消费单条消息的超时时间
long consumeTimeout() default 30000L;
```

RocketMQ 提供了两种顺序级别：

- 普通顺序消息 ：Producer 将相关联的消息发送到相同的消息队列。
- 完全严格顺序 ：在【普通顺序消息】的基础上，Consumer 严格顺序消费。

## 顺序消息

对于一个指定的Topic，消息严格按照先进先出（FIFO）的原则进行消息发布和消费，即**先发布的消息先消费，后发布的消息后消费**

但是Rocketmq的实现上分为了**生产顺序性和消费顺序性**。只有同时满足了生产顺序性和消费顺序性才能达到上述的FIFO效果。

- 生产顺序性：**单个**生产者**串行发送**的消息，如果设置了**相同的分区键**就会存储在**同一个队列**中，也就保证了生产顺序性。
  - 注意要点，要求的是单个生产者，且生产者使用了一个线程发送消息，否则即使设置相同分区键也不能保证顺序。
- 消费顺序性：同一个分区键的消息会被分配到同一个消息队列，消费者会按序消费

![官网图片](https://img.yesmylord.cn//image-20241228171722163.png)

如图所示，消息1-7依次被生产，有各自的分区键（相同分区键表面属于同一个业务），在分配后，每个消息队列可以保证相同分区键的消息是有序的。

此外还有其他的称呼方式：

- 普通顺序性：即生产顺序性
- 严格顺序性：生产顺序性+消费顺序性

> 目前已知的应用只有数据库 binlog 同步强依赖严格顺序消息，其他应用绝大部分都可以容忍短暂乱序

代码方面可以使用DefaultMQProducer或者rocketMQTemplate：

1. DefaultMQProducer使用时要传入一个**选择器**，规定了相同key消息的分区方式。

MessageQueueSelector的接口如下：

```java
public interface MessageQueueSelector {
    MessageQueue select(final List<MessageQueue> mqs, final Message msg, final Object arg);
    //  mqs 是可以发送的队列，msg是消息，arg是上述send接口中传入的Object对象，也就是key
}
```

**生产环境中建议选择最细粒度的分区键进行拆分**，例如，将**订单ID、用户ID**作为分区键关键字，可实现同一终端用户的消息按照顺序处理，不同用户的消息无需保证顺序。

2. 使用rocketMQTemplate比较简单，只需要额外传入一个分区键参数

```java
// 同步
rocketMQTemplate.syncSendOrderly(TOPIC, message, 分区键);
// 异步
rocketMQTemplate.asyncSendOrderly(TOPIC, message, 分区键, callback);
// 单向
rocketMQTemplate.sendOneWayOrderly(TOPIC, message, 分区键);
```

> 如果一个Broker掉线，那么此时队列总数是否会发化？

- 如果发生变化，那么同一个 ShardingKey 的消息就会发送到不同的队列上，造成乱序。
- 如果不发生变化，那消息将会发送到掉线Broker的队列上，必然是失败的。

RocketMQ 提供了两种模式，如果要保证严格顺序而不是可用性，创建 Topic 是要指定 `-o` 参数（--order）为true，表示顺序消息

## 事务消息

### 事务消息

> 事务消息：在普通消息基础上，**支持二阶段提交**能力

如何实现的？通过**半事务消息**，如果可以执行完成，就提交，否则就回滚（二阶段思想）

两个阶段：

![官网原图](https://img.yesmylord.cn//image-20241228213401667.png)

- 第一阶段：发送一个半事务消息到broker，同时本地开始执行事务
  - 半事务消息是指：生产者已经发送到broker上，但是**还不能被消费**的消息，是否能被消费看后续是提交还是回滚
- 第二阶段：判断两件事情，如果全部成功就标记commit，否则就回滚
  1. 半事务消息是否发送成功
  2. 事务是否执行成功

详细流程如图所示：

![官网原图](https://img.yesmylord.cn//image-20241228213922031.png)

1. 生产者将半事务消息发送至 `Broker`。
2. `Broker` 将消息持久化成功之后，向生产者返回 Ack 确认消息
3. 生产者开始执行本地事务
4. 生产者根据本地事务执行结果向服务端提交二次确认结果（Commit或是Rollback），服务端收到确认结果后处理逻辑如下：
   - 二次确认结果为Commit：服务端将半事务消息标记为可投递，并投递给消费者。
   - 二次确认结果为Rollback：服务端将回滚事务，不会将半事务消息投递给消费者。

5. 在断网或者是生产者应用重启的特殊情况下，若服务端**未收到**发送者提交的二次确认结果，或服务端收到的二次确认结果为**Unknown未知状态**，经过固定时间后，服务端将对消息生产者即生产者集群中任一生产者实例发起**消息回查**
6. 注意：服务端仅仅会**按照参数尝试指定次数，超过次数后事务会强制回滚**，因此未决事务的回查时效性非常关键，需要按照业务的实际风险来设置

事务消息**回查**步骤如下： 

7. 生产者收到消息回查后，需要检查事务执行的最终结果
8. 生产者根据最终状态再次提交二次确认，服务端仍按照步骤4对半事务消息进行处理

### 实现

`rocketMQTemplate`实现了该方法

```java
rocketMQTemplate.sendMessageInTransaction(TX_PRODUCER_GROUP, TOPIC, message, arg);
// 四个参数分别是 生产者组name、topic、msg、arg（执行本地事务需要的参数）
```

### 事务回查机制

> **事务回查机制**：RocketMq支持如果应用超过一定时长未 commit 或 rollback 这条事务消息，RocketMQ 会主动回查应用，询问这条事务消息是 commit 还是 rollback ，从而实现事务消息的状态最终能够被 commit 或是 rollback ，达到最终事务的一致性。

通过实现监听器`RocketMQLocalTransactionListener`：

- 注解`@RocketMQTransactionListener`，指定一个生产者组，回查时Broker端如果发现原始生产者已经崩溃，则会**联系同一生产者组的其他实例**回查

- `executeLocalTransaction` 是半事务消息发送成功后，执行本地事务的方法，具体执行完本地事务后，可以在该方法中返回以下三种状态：
  - `LocalTransactionState.COMMIT_MESSAGE`：提交事务，允许消费者消费该消息
  - `LocalTransactionState.ROLLBACK_MESSAGE`：回滚事务，消息将被丢弃不允许消费。
  - `LocalTransactionState.UNKNOW`：暂时无法判断状态，等待固定时间以后Broker端根据回查规则向生产者进行消息回查。
- `checkLocalTransaction`是由于二次确认消息没有收到，Broker端回查事务状态的方法。
  - 回查规则：本地事务执行完成后，若Broker端收到的本地事务返回状态为`LocalTransactionState.UNKNOW`，或生产者应用退出导致本地事务未提交任何状态。则Broker端会向消息生产者发起事务回查，第一次回查后仍未获取到事务状态，则之后每隔一段时间会再次回查。

```java
@RocketMQTransactionListener(txProducerGroup = TX_PRODUCER_GROUP)// 指定一个生产者组
public class TransactionListenerImpl implements RocketMQLocalTransactionListener {
    private Logger logger = LoggerFactory.getLogger(getClass());
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
        logger.info("[executeLocalTransaction][执行本地事务，消息：{} arg：{}]", msg, arg);
        return RocketMQLocalTransactionState.UNKNOWN;
    }

    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
        logger.info("[checkLocalTransaction][回查消息：{}]", msg);
        return RocketMQLocalTransactionState.COMMIT;
    }

}
```

> RocketMq与其他Mq事务消息的区别

RabbitMQ 和 Kafka 也有事务消息，支持发送事务消息的发送，以及后续的事务消息的 commit提交或 rollbackc 回滚。

但是要考虑一个极端的情况，在**本地数据库事务已经提交**的时时候，如果因为网络原因，又或者崩溃等等意外，导致事务消息没有被 commit ，最终导致这条事务消息丢失，分布式事务出现问题。

RocketMq提供了事务回查机制，可以保证**最终一致性**

> 事务回查的一点trick

对于实际的业务，我们可以：

- 第一步，在 `#executeLocalTransaction(...)` 方法中，先存储一条 `id` 为 `msg` 的事务编号，状态为 `UNKNOWN` 的记录。
- 第二步，调用带有**事务的**业务 Service 的方法。
  - 在该 Service 方法中，在逻辑都执行成功的情况下，更新 `id` 为 `msg` 的事务编号，状态变更为 `COMMIT` 。这样，我们就可以伴随这个事务的提交，更新 `id` 为 `msg` 的事务编号的记录的状为 `COMMIT` 
- 第三步，要以 `try-catch` 的方式，调用业务 Service 的方法。如此，如果发生异常，回滚事务的时候，可以在 `catch` 中，更新 `id` 为 `msg` 的事务编号的记录的状态为 `RocketMQLocalTransactionState.ROLLBACK` 。😭 极端情况下，可能更新失败，则打印 error 日志，告警知道，人工介入。
- 如此三步之后，我们在 `#executeLocalTransaction(...)` 方法中，就可以通过查找数据库，`id` 为 `msg` 的事务编号的记录的状态，然后返回。

## 总结

RocketMq的总结：

- **发送方式**：同步、异步、单向传输
- **消息类型**：普通消息、批量消息、定时消息、顺序消息
  - 普通消息：最常用的消息类型
  - 批量消息：将同一个topic下的多个消息打包起来一同发送，注意如果超过1MB需要自行分割
  - 定时消息：只有指定的时间到后，消息才可以被消费，Rocketmq支持18个级别的消费，不支持高精度消费
  - 顺序消息：消息可以保证生产顺序性和消费顺序性，一般情况下只保证生产顺序性即可。
  - 事务消息：普通消息+二阶段提交，RocketMq额外提供事务回查机制
- **消息模型**：集群消费和广播消费
  - 集群消费：
    - 特点：同一个消费组下的实例之间负载均衡。
    - 应用：适用于普遍情况，比如一个游戏升级后，A消费组可以专注于增加血量、B消费组可以专注于增加下一次升级所需的阈值等等。
  - 广播消费：
    - 特点：同一个消费组下的每一个实例获取所有消息
    - 应用：缓存了数据字典等配置表在内存中，可以通过广播消费，实现每个应用节点都消费消息，刷新本地内存的缓存
- **消费模式**：并发消费和顺序消费
- **消费重试**：消费失败会被重试16次，超过16次进入死信队列





