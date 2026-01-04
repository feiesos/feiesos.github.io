+++
date = '2025-12-21T13:21:24+08:00'
draft = false
title = 'Kafka快速入门'
toc = true
tocBorder = true
tags = ["Kafka", "消息队列", "分布式系统", "高吞吐"]
+++

## 简单介绍Kafka
Kafka是分布式系统，通常以集群的方式部署。

也就是说会同时存在多个Kafka实例（Broker），这些实例一般部署在不同服务器上。

稍后将讲解。

Kafka最初就是为了解决大数据的实时日志流而生的，它被用来处理每天千亿规模的日志量级。

相比RocketMQ和RabbitMQ等消息队列，Kafka的核心优势就是吞吐量大，能达到17.3w/s，相比之下RocketMQ只有11.6w/s，RabbitMQ只有5.95w/s，因此Kafka十分适合大规模、高吞吐的应用场景。

RabbitMQ和RocketMQ虽然性能不如Kafka，但是功能完善还有其他方面的优点，比如RocketMQ各方面最均衡、RabbitMQ运维成本低等。

下面我们绘制一个表格简单对比这三个数据库：


## Kafka的基本概念
1. topic，主题，是消息归类的基本单元
2. partitions，消息分区，通过偏移量offset来置顶消息的位置，是实现消息的分布式管理的核心机制
3. replicas，分区副本，每个分区可以有多个副本，一个leader（读写）多个follower（拷贝）
4. broker，集群，Kafka由一个或多个broker组成集群，每一个broker就是一个Kafka实例
5. ZooKeeper，协调多个broker，存储元数据
### 主题和分区（Topic&Partitions）
在Kafka中要实现消息的发送与订阅，必须首先创建Topic。\
因为Topic是Kafka进行消息归类的基本单元。Topic接收生产者发布的消息，并将消息转发给消费者。\
比如：
```text
 -----------------                -------------------                 -----------------
| Producer(生产者)| ==发送消息==> |      [Topic]     | <==订阅消息== |Consumer(消费者) |
 -----------------                |     Kafka实例    |                -----------------
                                  -------------------
```
