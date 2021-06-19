[TOC]

# RocketMQ

### broker机器参数调整

#### OS内核参数

> 需要开启大量线程（vm.max_map_count）
>
> 需要进行大量网络通信和磁盘IO(ulimit)
>
> 需要大量使用内存(vm.swappiness、vm.overcommit_memory)

#### JVM参数

> 使用G1回收器
>
> 根据机器内存适当调大region
>
> 等等好多

#### RocketMQ参数

> sendMessageThreadPoolNums=16  发送消息的线程池内线程数量默认16，可以根据机器CPU核数调整



### productor发信模式

#### 同步发送

> 同步阻塞发送

#### 异步发送

> 回调函数通知成功或失败

#### 单向发送

> 发送出就不管了

### consumer消费模式

#### push消费

> 本质上不停的尝试拉取，长轮询，如果没有消息会挂起15秒，15秒后再次尝试拉取，有消息到来会唤醒，看起来就像是即时推送

#### pull消费

> consumer拉取

### CommitLog

> oscache同步刷盘 防丢失