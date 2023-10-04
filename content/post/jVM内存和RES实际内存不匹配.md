---

title: "JVM内存占用和RES不匹配"
date: 2023-09-12T20:37:56+08:00
lastmod: 2023-09-10T20:37:56+08:00
draft: false
tags: ["jvm", "内存分析"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

# 环境

1. SpringBooot项目

2. Jar运行，参数：-Xms2g -Xmx2g 

3. Linux虚拟机运行内存8g

4. 启动占用18%左右

5. 接收文件分片，内存进行合并写磁盘

6. top -p pid 内存占用飙升到30%左右，RES占用2.2g左右

# 工具

1. jmap -histo pid 打印每个class的实例数目

2. jmap -heap pid 查看概要信息

3. jmap -dump:format=b,file=heap_pid.hprof  PID 

4. Jprofile  可远程可视化分析

5. Elipse memory Analyzer 可视化分析

# jvm内存分配

1. 操作系统给用户进程分配内存空间是虚拟内存，不是物理内存；

2. 进程在申请内存时，并不是直接分配物理内存的，而是分配一块虚拟空间，到真正堆这块虚拟空间写入数据时才会通过缺页异常（Page Fault）处理机制分配物理内存，也就是我们看到的进程 Res 指标。

3. 哪怕配置了Xms2G，启动后也不会直接占用 2G 内存，只是 JVM 在启动后会malloc 2G 而已，但实际占用的内存取决于你有没有往这 2G 内存区域中写数据的。

# jvm内存管理

JVM 的自动内存管理，其实只是先向操作系统申请了一大块内存，然后自己在这块已申请的内存区域中进行“自动内存管理”。JAVA 中的对象在创建前，会先从这块申请的一大块内存中划分出一部分来给这个对象使用，在 GC 时也只是这个对象所处的内存区域数据清空，标记为空闲而已。

# 为什么不把内存归还给操作系统？

JVM 还是会归还内存给操作系统的，只是因为这个代价比较大，所以不会轻易进行。而且不同垃圾回收器 的内存分配算法不同，归还内存的代价也不同。

比如在清除算法（sweep）中，是通过空闲链表（free-list）算法来分配内存的。简单的说就是将已申请的大块内存区域分为 N 个小区域，将这些区域同链表的结构组织起来。

每个 data 区域可以容纳 N 个对象，那么当一次 GC 后，某些对象会被回收，可是此时这个 data 区域中还有其他存活的对象，如果想将整个 data 区域释放那是肯定不行的。

所以这个归还内存给操作系统的操作并没有那么简单，执行起来代价过高，JVM 自然不会在每次 GC 后都进行内存的归还。

# 怎么归还给操作系统？

虽然代价高，但 JVM 还是提供了这个归还内存的功能。JVM 提供了-XX:MinHeapFreeRatio和-XX:MaxHeapFreeRatio 两个参数，用于配置这个归还策略。

* MinHeapFreeRatio 代表当空闲区域大小下降到该值时，会进行扩容，扩容的上限为 Xmx

* MaxHeapFreeRatio 代表当空闲区域超过该值时，会进行“缩容”，缩容的下限为Xms
  不过虽然有这个归还的功能，不过因为这个代价比较昂贵，所以 JVM 在归还的时候，是线性递增归还的，并不是一次全部归还。

| JAVA 版本 | 垃圾回收器                                   | VM Options                                                                       | 是否可以“归还” |
| ------- | --------------------------------------- | -------------------------------------------------------------------------------- | -------- |
| JAVA 8  | UseParallelGC(ParallerGC + ParallerOld) | -Xms100M -Xmx2G -XX:MaxHeapFreeRatio=40                                          | 否        |
| JAVA 8  | CMS+ParNew                              | -Xms100M -Xmx2G -XX:MaxHeapFreeRatio=40 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC | 是        |
| JAVA 8  | UseG1GC(G1)                             | -Xms100M -Xmx2G -XX:MaxHeapFreeRatio=40 -XX:+UseG1GC                             | 是        |
| JAVA 11 | UseG1GC(G1)                             | -Xms100M -Xmx2G -XX:MaxHeapFreeRatio=40                                          | 是        |
| JAVA 16 | UseZGC(ZGC)                             | -Xms100M -Xmx2G -XX:MaxHeapFreeRatio=40 -XX:+UseZGC                              | 否        |

# 综上所述

通过上面排查和分析，堆内存不会释放是无关代码导致内存溢出，是jvm垃圾回收机制。
