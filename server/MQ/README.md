# 什么是消息队列 (Message Queue)?
![](./assets/MQ/MQ.png)

`MQ` 可以理解为一种通信方式。把数据放入消息队列的一方被称为 生产者(`Producer`); 从消息队列中取出数据的一方被称为 消费者(`Consumer`)。

> 消息队列可以被用来实现进程、线程间的通信。

# 为什么需要消息队列？
+ 解耦: `Producer` 只需要将数据放入 `MQ` ，剩下的一概不关心
+ 异步: 一般 `Producer` 都是负责主任务的，在并发量巨大的情况下，过多的附属任务会影响到主任务会导致主任务迟迟不能返回。这个时候可以利用 `MQ` 实现对附属任务的异步调用，以此提高吞吐量。
+ 限流: 在请求数目超出服务器能力的时候，可以将请求写入 `MQ`，服务器端自发去读取 `MQ` 进行处理，能够实现限流防止服务器崩溃。

# `MQ` 面临什么问题？
+ 高可用: 为保证系统的稳定系、健壮性， `MQ` 肯定都是分布式/集群的，那 `MQ` 就必须要做到能够提供现成的支持，无需适用方手动实现
+ 数据丢失问题: 如果 `Producer` 写入数据到 `MQ` ， `Consumer` 来没来得及读取 `MQ` ，`MQ` 就挂掉 那其中的数据就会丢失。那 `MQ` 中的数据就不能只是存储在内存中，disk？redis？DB？分布式文件系统？sync or async？
+ `Consumer` 如何消费数据: `MQ` 主动推送数据(push)？ 消费方轮训(pull)？
+ 其他:
  + 数据重复问题？
  + 要保证数据的顺序怎么办？

# 总结
@TODO