# JVM

[TOC]

## 内存结构

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvZGE3N2Q5MDE0Njc4NmMwY2IzZTE3MGI5YzkzNzZhZTQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

## 配置参数

| 参数                        | 说明                                     |
| --------------------------- | ---------------------------------------- |
| -Xms                        | 堆内存大小                               |
| -Xmx                        | 堆内存最大大小                           |
| -Xmn                        | 堆内存中新生代大小                       |
| -Xss                        | 每个线程的栈内存大小                     |
| -XX:MetaspaceSize           | 永久代（方法区）大小，1.8以后叫Metaspace |
| -XX:MaxMetaspaceSize        | 永久代最大大小                           |
|                             |                                          |

### 参数模板

java -Xms1024M -Xmx1024M -Xmn512M -Xss1M -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize512M -jar test.jar

## TLAB

>1、多个线程同时为对象申请分配内存空间，可能会出现指向同一个区域，使用同步方案影响效率，为了避免这种情况，HotSpot虚拟机使用了TLAB（Thread Local Allocation Buffer），即在堆空间的eden区内预先为每个线程分配一小块内存，当需要为对象分配空间时优先在此分配。
>
>2、分配是线程独享的，但是读取、回收等动作都是共享的。
>
>3、当对象大小超过TLAB剩余空间时，比较refill_waste值，如果大小比refill_waste大则在堆其他位置分配，若比refill_waste小则废弃当前TLAB重新创建新的。

### 相关参数

| 参数                        | 说明               |
| --------------------------- | ------------------ |
| -XX:+/-UseTLAB              | 是否开启TLAB分配   |
| -XX:TLABWasteTargetPercent  | 占eden空间百分比   |
| -XX:-ResizeTLAB             | 禁用自动调整大小   |
| -XX:TLABSize                | 指定大小           |
| -XX:TLABRefillWasteFraction | refill_waste的大小 |
| -XX:+PringTLAB              | 跟踪使用情况       |
|                             |                    |


## 回收对象

### 可达性分析

在对象的引用链上寻找是否存在一个GC Roots

### GC Roots

> 1、被类的静态变量引用
>
> 2、被方法中的局部变量引用

### 引用类型

#### 强引用

> 强引用不执行回收
>
> ```java
> UserInfo userInfo = new UserInfo();
> ```

#### 软引用

> 某个实例对象被SoftReference类对象包裹，在系统要发生内存溢出，才对该对象进行回收
>
> ```java
> SoftReference<UserInfo> userInfo = new SoftReference<UserInfo>(new UserInfo());
> ```

#### 弱引用

>WeakReference类包裹，对象只能存活到下次一次GC之前
>
>```java
>WeakReference<UserInfo> userInfo = new WeakReference<UserInfo>(new UserInfo());
>```

#### 虚引用

>PhantomReference类包裹，无法通过虚引用获取到对象的实例，作用是在被回收时会收到一个系统通知

### 重写finalize()方法

>对象被回收之前会检查一下是否重写了finalize方法，如果在方法中重新被GC Root引用，则不会被回收
>
>```java
>public classs UserInfo {
>    public static UserInfo instance;
>    @Override
>    protected void finalize() throws Throwable {
>        UserInfo.instance = this;
>    }
>}
>```

## 收集器

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvMTVmYjc1NDc2MmZmNWRmM2Y3ZjYzZTVjMjZkNGQzYWU_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

| 收集器            | 适用年代             | 说明                                                         |
| ----------------- | -------------------- | ------------------------------------------------------------ |
| Serial            | 新生代               | 复制算法，单线程执行                                         |
| ParNew            | 新生代               | 复制算法，多线程（默认CPU核数）                              |
| Parallel Scavenge | 新生代               | 吞吐量优先，多线程，单词收集时间短，频率增加，减少单次STW时间 |
| Serial Old        | Serial的老年代版本   | 标记-整理，单线程                                            |
| Parallel Old      | Parallel的老年代版本 | 标记-整理，多线程                                            |
| CMS               | 老年代               | 标记-清除                                                    |
| G1                | 通用                 |                                                              |

### 执行GC的时候还能创建对象分配空间么

> 可以，cms回收器第二步和第四步不STW

## 类加载机制

> 1、加载，通过类名读取二进制流在内存中生成class对象
>
> 2、校验class对象的数据完整性

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvMjdhYzg3ZjQzOTJmMGFiOTllNGM2NWMyM2NjNzE5NDU_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

加载

>1、获取二进制字节流
>
>2、静态存储结构转换到方法区
>
>3、内存中生成class对象，作为方法区数据的访问入口

验证

> 1、文件格式，魔数、版本号等
>
> 2、元数据，类是否有父类，是否继承了final修饰的类
>
> 3、字节码
>
> 4、符号引用
>
> ...

准备

> 为静态变量分配内存及赋初始值

解析

> 将常量池内的符号引用替换为直接引

### 初始化时机

#### 主动引用

> 1、new实例化对象、static对象方法
>
> 2、反射调用
>
> 3、类初始化，要先对其父类初始化
>
> 4、default修饰的接口

#### 被动引用

> 1、通过子类调用父类的静态变量，子类不会执行初始化
>
> 2、通过数组定义引用类
>
> 3、常量会在编译期进入常量池

### 类加载器

#### 启动类加载器（BootstrapClassLoader）

>加载lib下或-Xbootclasspath路径下的类

#### -扩展类加载器（ExtensionClassLoader）

>加载lib/ext或者被java.ext.dirs系统变量所指定的路径下的类

#### -平台类加载器（PlatformClassLoader）

>加载lib/ext或者被java.ext.dirs系统变量所指定的路径下的类

#### 应用程序类加载器（ApplicationClassLoader）

>加载用户路径上所指定的类库

#### 自定义类加载器（UserClassLoader）

>用户自定义

Java9以后

![image-20200810185228880](C:\Users\EDZ\AppData\Roaming\Typora\typora-user-images\image-20200810185228880.png)

> 当平台及应用程序加载器收到请求，先判断该类是否能归属到某一个系统模块，如果可以则优先委派给负责的加载器执行，否则交由父加载器

### metaspace中class什么时候被回收

> 1、该类的所有实例对象都已被回收
>
> 2、该类的加载器已被回收
>
> 3、该类的class对象没有任何引用

## 如何合理配置JVM参数

> 1、找到系统的压力点，预估峰值QPS
>
> 2、估算每次请求的耗时和创建对象占用的内存大小
>
> 3、计算同一时刻，会占用多少内存空间
>
> 4、假设流量放大10-20倍，根据分配的新生代大小，估算yongGC频率和FullGC频率
>
> 5、调整参数



#### MinorGC和FullGC会在什么时候触发？

>

#### 频繁FullGC的可能原因？

> 1、新生代增长快，内存分配不合理，survivor区设置太小，导致存活对象直接进入老年代
>
> 2、一次加载缓存了大量数据，占用老年代空间
>
> 3、出现内存泄漏创建大量对象一直未被回收
>
> 4、加载类过多，metaspace被占满
>
> 5、显式调用了System.gc()

#### 内存发生OOM的区域和可能原因

> 1、metaspace，默认几十m，可能原因 设置太小而需要加载的class很多 或 使用cglib动态生成class导致空间被占满
>
> 2、jvm栈，代码bug，存在死循环调用
>
> 3、堆，老年代被大量存活对象占满，可能原因 负载过高 或 内存泄漏

内存分配估算

> 1、一次请求产生多少对象，Eden区增长速度
>
> 2、新生代MinorGC频率、耗时，有多少存活对象进入老年代
>
> 3、老年代FullGC频率、耗时
>
> 4、