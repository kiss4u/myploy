[TOC]

### RPC调用过程

> 1、client stub将接口名称、方法、参数组装成网络传输消息体
>
> 2、client stub找到server端地址，将序列化后的消息发送到服务端
>
> 3、server stub接收到消息，进行反序列化，调用本地服务
>
> 4、本地服务执行后将结果返回给server stub
>
> 5、server stub结果序列化打包发送给client stub

### dubbo和feign对比

|          | Dubbo                                                        | feign                                                   |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| 传输协议 | 1、支持多种传输协议，比如说dubbo、http、rmi等<br />2、默认Dubbo协议，netty框架，TCP传输，适合数据量小、高并发、服务提供者远远小于消费者场景 | Http短连接，不适合高并发访问                            |
| 负载均衡 | 1、轮询、权重随机、一致性hash、活跃度<br />2、支持到接口级   | 1、轮询、随机、response time加权<br />2、支持到客户端级 |
| 容错机制 | 1、集群容错机制<br />failover - 失败自动切换，设置retries重试n次<br />failfast - 失败报错；适用于幂等操作，比如说新增记录<br />failback - 失败自动恢复，记录失败请求，定时重发；适用于消息通知<br />failsafe - 出现异常时，直接忽略<br />forking - 并行调用多个服务器，只要一个成功即返回；适用于实时性高的读操作，设置forks限定并行数<br />broadcast - 广播所有服务提供者，逐个调用，任意一个报错则报错；适用于缓存或本地资源更新 | 熔断                                                    |

## Dubbo

### 流程图

![image-20210529104906978](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210529104906978.png)

### 节点角色

| 节点     | 说明                           |
| -------- | ------------------------------ |
| provider | 暴露接口的服务提供方           |
| consumer | 调用远程服务接口的消费方       |
| register | 提供服务发现负载均衡的注册中心 |
| monitor  | 调用情况的监控中心             |

### 工作原理

> 1、服务启动时，provider和consumer根据配置信息连接到注册中心，注册或订阅
>
> 2、consumer把provider信息缓存到本地，如果provider发生变化进行修改
>
> 3、consumer通过代理对象调用，负载均衡选择一台provider
>
> 4、provider收到请求后进行反序列化后调用

### provider暴露过程

> 1. 在容器启动的时候，通过ServiceConfig解析标签，创建dubbo标签解析器来解析dubbo的标签，容器创建完成之后，触发ContextRefreshEvent事件回调开始暴露服务
> 2. 通过ProxyFactory获取到invoker，invoker包含了需要执行的方法的对象信息和具体的URL地址
> 3. 再通过DubboProtocol的实现把包装后的invoker转换成exporter，然后启动服务器server，监听端口
> 4. 最后RegistryProtocol保存URL地址和invoker的映射关系，同时注册到服务中心

![img](https://img-blog.csdnimg.cn/img_convert/73fc309f0d59b0c6a86a56c9d5c72443.png)

### consumer引用过程

>1. 首先客户端根据配置文件信息从注册中心订阅服务
>2. 之后DubboProtocol根据订阅的得到provider地址和接口信息连接到服务端server，开启客户端client，然后创建invoker
>3. invoker创建完成之后，通过invoker为服务接口生成代理对象，这个代理对象用于远程调用provider，服务的引用就完成了

![img](https://img-blog.csdnimg.cn/img_convert/ddf4c73f74512613ea7c98251aa9620a.png)

### 注册中心

> 默认zookeeper，可以替换nacos等

### 配置方式

> xml、API代码、注解式、properties文件

### 核心配置

> dubbo.registry.address 注册中心地址
>
> dubbo.scan.base-package 暴露接口扫描路径
>
> dubbo.protocol.name 传输协议
>
> dubbo.protocol.port 通信端口

### 支持的序列化协议

>dubbo、hession、json等

### 集群容错机制

>

### SPI机制

> service provider interface，服务发现机制
>
> 接口实现类全名配置在文件中，通过服务读取配置进行加载类，可以实时动态的替换实现类



## 问题

### 为什么要通过代理对象通信？

> 实现接口的透明代理，封装调用细节，可以像本地操作一样调用远程方法
>
> 通过代理实现负载均衡、容错机制、监控统计等

### Dubbo启动时如果依赖的服务不可用会怎样？

> 启动失败，可以针对consumer、provider、register、reference设置check false关闭检查

### 注册了多个同一样的服务，如果测试指定的某一个服务呢？

> 可以针对接口配置机器ip，绕过注册中心，点对点直连

### Dubbo支持服务多协议吗？

> 可以针对接口或分组配置协议类型（同一接口可以配置多种协议，比如说）

### 当一个服务接口有多种实现时怎么做？

> 可以对接口进行分组，消费方和提供方在同组访问

### 服务上线怎么兼容旧版本

>利用版本号过渡，多个版本发布到注册中心，消费方指定调用的版本号

### 服务读写推荐的容错策略是怎样的？

> 读推荐failover，失败自动切换，默认重试2次
>
> 写推荐failfast，失败返回报错

### Fegin支持长链接吗

> 1、默认情况下使用的是HttpURLConnection连接类，使用1.8提供的HttpClient完成连接，
>
> 只有URL相同的情况下，才能进行连接复用
>
> 2、在URL变化较大的情况下，JDK默认的HttpClient会保持一段时间才释放连接，占用系统资源
>
> 建议使用ApacheHttpClient连接池

### 为什么Feign调用会默认在请求头中加上Connection:keep-alive?

> 配置keepAliveProp属性值为true时，才放入长连接缓冲池 KeepAliveCache
>
> 一定程度维持连接，实现复用