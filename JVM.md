# JVM

[TOC]

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

