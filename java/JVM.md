# JVM

[TOC]

## 内存结构

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvZGE3N2Q5MDE0Njc4NmMwY2IzZTE3MGI5YzkzNzZhZTQ_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

## TLAB

>1、多个线程同时为对象申请分配内存空间，可能会出现指向同一个区域，使用同步方案影响效率，为了避免这种情况，HotSpot虚拟机使用了TLAB（Thread Local Allocation Buffer），即在堆空间的eden区内预先为每个线程分配一小块内存，当需要为对象分配空间时优先在此分配。
>
>2、分配是线程独享的，但是读取、回收等动作都是共享的。
>
>3、当对象大小超过TLAB剩余空间时，比较refill_waste值，如果大小比refill_waste大则在堆其他位置分配，若比refill_waste小则废弃当前TLAB重新创建新的。

### 相关参数

|                             |                    |
| --------------------------- | ------------------ |
| -XX:+/-UseTLAB              | 是否开启TLAB分配   |
| -XX:TLABWasteTargetPercent  | 占eden空间百分比   |
| -XX:-ResizeTLAB             | 禁用自动条调整大小 |
| -XX:TLABSize                | 指定大小           |
| -XX:TLABRefillWasteFraction | refill_waste的大小 |
| -XX:+PringTLAB              | 跟踪使用情况       |

## 可回收对象

从GC Roots作为出发点，路径上引用链不可达的对象进行回收

#### GC Roots

> 1、
>
> 2、

#### 引用类型

#### 强引用

>强引用不执行回收

#### 软引用

> SoftReference类，在系统要发生内存溢出之前，对这些对象进行回收

#### 弱引用

>WeakReference类，对象只能存活到下次一次GC之前

#### 虚引用

>PhantomReference类，无法通过虚引用获取到对象的实例，作用是在被回收时会收到一个系统通知

## 收集器

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxNy85LzQvMTVmYjc1NDc2MmZmNWRmM2Y3ZjYzZTVjMjZkNGQzYWU_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ)

| 收集器            | 年代                 | 说明                               |
| ----------------- | -------------------- | ---------------------------------- |
| Serial            | 用在新生代           | 复制算法，单线程收集               |
| ParNew            | 用在新生代           | 复制算法，多线程收集               |
| Parallel Scavenge | 用在新生代           | 吞吐量优先，复制算法，并行的多线程 |
| Serial Old        | Serial的老年代版本   | 标记整理，单线程                   |
| Parallel Old      | Parallel的老年代版本 | 标记整理，多线程                   |
| CMS               |                      | 标记清除                           |
| G1                |                      |                                    |



### 类加载机制

> 1、加载，通过类名读取二进制流在内存中生成class对象
>
> 2、校验class对象的数据完整性
>
> 

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

#### 初始化时机

##### 主动引用

> 1、new实例化对象、static对象方法
>
> 2、反射调用
>
> 3、类初始化，要先对其父类初始化
>
> 4、default修饰的接口

##### 被动引用

> 1、通过子类调用父类的静态变量，子类不会执行初始化
>
> 2、通过数组定义引用类
>
> 3、常量会在编译期进入常量池



#### 类加载器

##### 启动类加载器（BootstrapClassLoader）

>加载lib下或-Xbootclasspath路径下的类

##### -扩展类加载器（ExtensionClassLoader）

>加载lib/ext或者被java.ext.dirs系统变量所指定的路径下的类

##### -平台类加载器（PlatFormClassLoader）

>加载lib/ext或者被java.ext.dirs系统变量所指定的路径下的类

##### 应用程序类加载器（ApplicationClassLoader）

>加载用户路径上所指定的类库

##### 自定义类加载器（UserClassLoader）

>用户自定义

Java9以后

![image-20200810185228880](C:\Users\EDZ\AppData\Roaming\Typora\typora-user-images\image-20200810185228880.png)

> 当平台及应用程序加载器收到请求，先判断该类是否能归属到某一个系统模块，如果可以则优先委派给负责的加载器执行，否则交由父加载器
