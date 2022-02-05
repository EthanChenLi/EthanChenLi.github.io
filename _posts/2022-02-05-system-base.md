---
layout: post
title:  "分布式系统专题第三章-BASE理论"
date:   2022-02-05 16:30:00 +0800
categories: 分布式系统
tags: system
comments: 1
---
# BASE理论
> BASE理论是对CAP中AP的一个扩展，通过牺牲强一致性来获得可用性，当出现故障允许部分不可用但要保证核心功能可用，允许数据在一段时间内是不一致的，但最终一致性。满足BASE理论的事务，我们称之为“柔性事务”。


BASE是指：
* 基本可用（Basically Available）
基本可用是指分布式系统在出现故障的时候，允许损失部分可用性（例如响应时间、功能上的可用性），允许损失部分可用性。需要注意的是，基本可用绝不等价于系统不可用。
响应时间上的损失：正常情况下搜索引擎需要在0．5秒之内返回给用户相应的查询结果，但由于出现故障（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到了1～2秒。
功能上的损失：购物网站在购物高峰（如双十一）时，为了保护系统的稳定性，部分消费者可能会被引导到一个降级页面。
* 软状态（Soft state）
软状态是指允许系统存在中间状态，而该中间状态不会影响系统整体可用性。分布式存储中一般一份数据会有多个副本，允许不同副本同步的延时就是软状态的体现。mysql replication的异步复制也是一种体现。
* 最终一致性（Eventual Consistency）
最终一致性是指系统中的所有数据副本经过一定时间后，最终能够达到一致的状态。弱一致性和强一致性相反，最终一致性是弱一致性的一种特殊情况。（这也是分布式事务的想法）

从客户端角度，多进程并发访同时，更新过的数据在不同程如何获的不同策珞，决定了不同的一致性。
* 对于关系型要求更新过据能后续的访同都能看到，这是强一致性。
* 如果能容忍后经部分过者全部访问不到，则是弱一致性
* 如果经过一段时间后要求能访问到更新后的数据，则是最终一致性