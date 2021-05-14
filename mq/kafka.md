

[TOC]

### 定义

#### topic

#### partition

#### leader和follower

#### ISR

### producer

#### 核心组件

##### Metadata

>从broker拉取集群分布数据，topic - patitions(leader + followers , ISR)，默认5分钟同步一次，写入的某个topic元数据不存在或有新的broker加入时，立刻拉取一次

##### Partitioner

>决定消息路由到topic的哪个分区

##### RecordAccumulator

 >消息添加到内存缓存区

##### NetworkClient

>网络通信组件，一个网络连接最多空闲多长时间（9min），每个连接最多有几个request待响应（5），socket发送缓冲区大小（128k），接收缓冲区大小（32k）

##### Sender

>负责取出缓冲区内数据发送，acks（0,1,all,默认1）


#### 核心参数

> request大小（1m）
>
> 超时时间（30s）
>
> 重试时间间隔（100ms）
>
> 缓冲区大小（32m），写入
>
> 阻塞时间（60s）



#### send流程

1、添加拦截器（自定义）

2、阻塞获取topic元数据

3、序列化key、value成byte[]

4、根据topic，partitioner计算消息的目标分区

5、检查是否超过请求大小限制，缓冲区大小限制，将数据添加到内存缓冲区

6、设置回调函数（自定义）及拦截器的回调

7、同分区的消息打包成batch，如果满了唤醒sender线程发送



### 问题

#### kafkaproducer初始化时是否拉取集群元数据？

> 不，metainitforalltopics默认false，仅初始化broker信息，转化成node存放到cluster对象实例中

#### maxBlockTimeMs，都什么情况会出现send超时

> topic的元数据不存在，首次同步拉取超时
>
> 缓冲区已满，阻塞等待超时

#### 没有指定分区key是怎么路由的?

> 1.0下通过一个AutomicInteger累加值，topic下所有分区数对其取余
>
> 2.0下，threadlocalrandom一个值，如果可用分区小于1，用topic下所有分区值对其取余；如果可用分区等于1，取get(0)；如果大于1，用topic下可用分区数取余
>
> 结果会缓存到concurrentHashMap

#### 消息发送到内存缓冲区之前的工作？

> 生成消息大小，校验是否超过request大小或缓冲区大小限制

#### 内存缓冲写入时，为什么引入double check模式

>当多个线程分别申请到bytebuffer资源，其中一个bytebuffer放入batch队列中等待发送，其余放入bufferpool中待利用

#### 内存缓冲里的batch是如何判定可以发送的

>batch满了 | batch已超过等待时间 | 内存已满 | 客户端结束 | flushinprogress

#### 如何判断目标borker可以进行数据发送

> 已建立好连接、nio的channel已建立、未超出inflightrequest限定数量

#### 和broker建立连接

> 没连过或断开状态，重试时间已超过





