[TOC]



### 生产者发消息流程

> 1、provider连接到broker节点，建立connection，开启一个channel
>
> 2、声明一个交换器，设置相关属性如类型，是否持久化等
>
> 3、声明一个队列，设置相关属性如是否排他，是否自动删除等
>
> 4、通过路由key将交换器和队列绑定
>
> 5、发送消息到broker，包含路由key和交换器等信息
>
> 6、broker根据路由key执行消息入队
>
> 7、关闭channel和connection

### 消费模式

#### push

>Basic.Consume方法消费，mq会不断推送消息给消费者，通过basicQos限制未ack的消息个数

#### pull

> Basic.basicGet方法获取一条消息

### 消息顺序性



### ack机制

> autoAck设置true时，mq自动移除已消费的消息；设置false时，mq会等待消费者返回ack才将消息移除；如果一直未收到ack，mq会把消息重新放回待消费队列

### confirm机制



#### 不可达消息

> mandatory参数，设置true，未找到符合条件的队列会将消息返回给生产者；设置false，消息会直接丢弃

### TTL

#### 消息ttl

> 消息在队列中超过ttl值时，会变成死信，不会再发送
>
> 1、队列属性设置，队列中所有消息都有相同的过期时间，channel.queueDeclare设置x-message-ttl参数
>
> 2、消息本身设置，channel.basicPublish设置expiration参数

#### 队列ttl

> 队列未使用状态超过ttl，自动删除
>
> channel.queueDeclare设置x-expire参数

### 死信队列

#### 消息变成死信的情况

##### 1、消息被拒绝，且设置requeue参数为false

##### 2、消息过期

##### 3、队列达到最大长度



### Exchange类型

#### Direct

> 可以不用指定routing key，在此Exchange下创建的queue默认routing key等同于队列名

![img](https://img-blog.csdn.net/20160820183903987?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### Fanout

> 会忽略routing key，发送到exchange下所有队列中

![img](https://img-blog.csdn.net/20160820184622496?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### Topic

>根据routing key 路由到响应queue

![img](https://img-blog.csdn.net/20160820184744544?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### Header

>

![img](https://img-blog.csdn.net/20160820184114896?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)