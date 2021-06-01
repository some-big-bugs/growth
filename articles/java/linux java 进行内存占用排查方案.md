# java 进程内存占用排查方案

---

**Table of Contents**

[TOC]

---



# 1. 使用 jps 查找指定的 Java 进程

```shell
[www@dn-a18m30p121-skyrim-rs skyrim]$ jps -mlvV
15771 com.br.skyrim.engine.Starter -m sync -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8902 -Dcom.sun.management.jmxremote.local.only=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcsp.sentinel.dashboard.server=172.18.31.54:8080 -Dproject.name=skyrim-blaze-engine -Dcsp.sentinel.log.dir=./sentinel/ -Dcsp.sentinel.log.use.pid=true -DprojectName=blaze_skyrim -Dspring.profiles.active=publish -Xmx10g -Xms10g -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:MaxGCPauseMillis=50 -verbose:gc -Xloggc:logs/gc/gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+DisableExplicitGC -XX:NativeMemoryTracking=detail
```

这里主要看 sync 启动的，进程为 11410 ，可以看到启动 jvm 参数

- -Xmx10g      堆内存最大可用内存为10G
- -Xms10g      堆内存的初始大小 10G
- -XX:+UseG1GC 
- -XX:G1HeapRegionSize=16m                 region 大小 取值 1-32M
- -XX:MaxGCPauseMillis=50                      gc 停顿时间毫秒
- -verbose:gc 
- -Xloggc:logs/gc/gc_%p.log                       gc 日志打印位置
- -XX:+PrintGCDetails 
- -XX:+PrintGCDateStamps 
- -XX:+DisableExplicitGC 
- -XX:NativeMemoryTracking=detail



# 2. 使用 PS 命令查看内存占用

```shell
[www@dn-a18m30p121-skyrim-rs skyrim]$ ps -aux | grep sync
www      15771 23.8  8.1 18191148 1668836 pts/4 Sl  14:33   2:39 java -cp ./skyrim.engine.jar com.br.skyrim.engine.Starter -m sync

[www@dn-a18m30p121-skyrim-rs skyrim]$ top -p 15771
top - 14:45:02 up 249 days, 22:25,  5 users,  load average: 0.12, 0.26, 0.27
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
Cpu(s):  6.9%us,  1.5%sy,  0.0%ni, 91.4%id,  0.1%wa,  0.0%hi,  0.1%si,  0.1%st
Mem:  20528948k total, 12035976k used,  8492972k free,   130700k buffers
Swap: 10239992k total,   416164k used,  9823828k free,  3431944k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
15771 www       20   0 17.3g 1.6g  24m S  2.0  8.1   2:43.09 java
```

从 top 命令和 ps 命令都可以看出内存占用为 1.6G




# 3. 使用 jmap 命令查看内存占用



**1. jmap -heap pid：输出堆内存设置和使用情况（这个命令可能会卡住进程）**

```
jmap -heap 15771
Attaching to process ID 15771, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13

using thread-local object allocation.
Garbage-First (G1) GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40                                当前当空闲堆/当前总的堆内存 < 40,触发堆空间扩容
   MaxHeapFreeRatio         = 70                                当前当空闲堆/当前总的堆内存 > 70,触发堆空间缩容
   MaxHeapSize              = 10737418240 (10240.0MB)           最大内存 10 G
   NewSize                  = 1363144 (1.2999954223632812MB)    新生代空间默认值大小 1.3 
   MaxNewSize               = 6442450944 (6144.0MB)             最大新生代大小 6144 M
   OldSize                  = 5452592 (5.1999969482421875MB)    JVM 老年代堆空间的默认值
   NewRatio                 = 2        Old/New的比值，默认值为2，表示Old代与Young代的比值为2比1   
   SurvivorRatio            = 8        Eden/Survivor的比例，设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)  						压缩对象头空间大小
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 16777216 (16.0MB)                 每个 Reigon 的大小
```



**2. jmap -histo pid：输出heap的直方图，包括类名，对象数量，对象占用大小**

```
jmap -histo 8192

 num     #instances         #bytes  class name
----------------------------------------------
   1:       8007512     11_4163_9816  [C
   2:       5953232     11_3662_3944  [B
   3:       1245661     10_7231_0568  [I
   4:       3525052      112801664  java.util.HashMap$Node
   5:       4496498      107915952  java.lang.String
   6:        267270      104769840  com.alibaba.druid.stat.JdbcSqlStat
   7:        582224       64785368  [Ljava.util.HashMap$Node;
   8:       1061499       47926344  [Ljava.lang.Object;
   9:       1017924       40716960  java.util.HashMap$KeyIterator
  10:        767274       38419320  [Ljava.lang.String;
  11:        318382       33111728  sun.security.ssl.SSLSessionImpl
  12:        642954       30861792  java.util.HashMap
  13:        444777       24231720  [S
  14:        361783       23154112  org.apache.thrift.protocol.TCompactProtocol
  15:       1340417       21446672  java.lang.Object
  16:        319846       21139624  [Ljava.util.Hashtable$Entry;
  17:        327716       20973824  java.util.regex.Matcher
  18:        180923       20263376  sun.nio.ch.SocketChannelImpl
  19:        827218       19853232  java.util.ArrayList
  20:        385132       18486336  java.nio.HeapByteBuffer
  21:        319968       17918208  sun.security.util.MemoryCache$SoftCacheEntry
  22:        313170       17537520  java.util.HashMap$TreeNode
  23:        220504       15876288  org.apache.log4j.spi.LoggingEvent
  24:        318914       15307872  java.util.Hashtable
  25:        328939       13157560  java.lang.ref.Finalizer
  26:        542628       13023072  org.apache.thrift.TByteArrayOutputStream
  27:        542628       13023072  org.apache.thrift.transport.TMemoryInputTransport
  28:        180854       13021488  org.apache.thrift.server.AbstractNonblockingServer$FrameBuffer
  29:        309801       12392040  java.util.LinkedHashMap$Entry
  30:        364023       11648736  java.net.InetAddress$InetAddressHolder
Total      25394873     27_9293_8856
```



**3. jmap -histo:live pid：同上，只输出存活对象信息**



# 4. jstat 查看堆内存各部分的使用量，以及加载类的数量

```s
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]
```

jstat [-命令选项] [-h标题之前的内容行数] [vmid] [间隔时间/毫秒] [查询次数]

**其中 option 可选及其示例：**

- gc 显示gc的信息，查看gc的次数，及时间。其中最后五项，分别是 young gc 的次数，young gc的时间，full gc 的次数，full gc的时间，gc的总时间

    S0C : survivor0区的总容量
    S1C : survivor1区的总容量
    S0U : survivor0区已使用的容量
    S1U : survivor1区已使用的容量
    EC : Eden区的总容量
    EU : Eden区已使用的容量
    OC : Old区的总容量
    OU : Old区已使用的容量
    
     MC     MU
    
    PC 当前perm的容量 (KB)
    PU perm的使用 (KB)
    YGC : 新生代垃圾回收次数
    YGCT : 新生代垃圾回收时间
    FGC : 老年代垃圾回收次数
    FGCT : 老年代垃圾回收时间
    GCT : 垃圾回收总消耗时间



这里打印了每隔 1s 获取一次，共获取了 10 次，

Edge 总大小 480 MB, 占用 400 MB

S0 未分配空间

S1 总大小 64 MB, 占用 64 MB

Old区的总容量: 9696MB, 占用 156.32 MB

```
[www@dn-a18m30p121-skyrim-rs ~]$ jstat -gc 15771
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
 0.0   65536.0  0.0   65536.0 491520.0 409600.0 9928704.0   160078.0  83228.0 81124.3 12108.0 11699.3     21    1.517   0      0.000    1.517
```




- jstat -gccapacity:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。其他的可以根据这个类推， OC是old 内存的占用量。

```
[www@dn-a18m30p121-skyrim-rs gc]$ jstat -gccapacity 15771
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
     0.0 10485760.0 557056.0    0.0 49152.0 507904.0        0.0 10485760.0  9928704.0  9928704.0      0.0 1122304.0  84380.0      0.0 1048576.0  12236.0     95     0
```

- gcnew new对象的信息

```
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
12288.0 12288.0    0.0    0.0  4  15 12288.0 206848.0 119412.2      7    0.081
```

- gcnewcapacity new对象的信息及其占用量

```
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
   87040.0  1397760.0   363008.0 465920.0  12288.0 465920.0  12288.0  1396736.0   206848.0     7     2
```
- gcold old对象的信息

```
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
 35416.0  32942.8   4736.0   4310.2    140800.0     13896.3      7     2    0.080    0.161
```
- gcoldcapacity old对象的信息及其占用量

```
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
   175104.0   2796544.0    140800.0    140800.0     7     2    0.080    0.161
```

- gcutil 统计gc信息统计

Summary of garbage collection statistics.

`S0`: 生存空间0利用率占该空间当前容量的百分比.

`S1`: 幸存者空间的利用率占该空间当前容量的百分比.

`E`: 伊甸园空间利用率，以该空间当前容量的百分比表示.

`O`: 老空间利用率占空间当前容量的百分比.

`M`: 元空间利用率占空间当前容量的百分比.

`CCS`: 压缩的类空间利用率，以百分比表示

`YGC`: 年轻代GC事件的数量.

`YGCT`: 年轻代垃圾收集时间.

`FGC`:  Full GC 事件的数量.

`FGCT`: Full GC 的垃圾收集时间.

`GCT`: 总的垃圾收集时间.

```
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     0    0    0.081
```



# 5. 查看 GC 日志

在刚启动的时候，内存在 1.6 G 左右，之后升到 3G  4.8G  6G  7.6G，因为 gc 日志落地了，在 logs/gc 下面，可以根据进程号查看，这里先看一下内存的变化情况， 使用 `grep 'Eden' gc_pidxxxxxx`, 查看日志：

 [Eden: 272.0M(512.0M)->0.0B(496.0M) Survivors: 0.0B->16.0M Heap: 264.0M(10.0G)->14.3M(10.0G)]
 [Eden: 448.0M(448.0M)->0.0B(448.0M) Survivors: 64.0M->64.0M Heap: 656.3M(10.0G)->212.1M(10.0G)]

这里还有很多日志，因为和上一条变化不大直接省略了

[Eden: 480.0M(480.0M)->0.0B(480.0M) Survivors: 32.0M->32.0M Heap: 709.0M(10.0G)->232.5M(10.0G)]
[Eden: 480.0M(480.0M)->0.0B(3648.0M) Survivors: 32.0M->32.0M Heap: 712.5M(10.0G)->233.8M(10.0G)]
[Eden: 3648.0M(3648.0M)->0.0B(384.0M) Survivors: 32.0M->128.0M Heap: 3881.8M(10.0G)->321.2M(10.0G)]

[Eden: 384.0M(384.0M)->0.0B(464.0M) Survivors: 128.0M->48.0M Heap: 705.2M(10.0G)->252.9M(10.0G)]

...

[Eden: 464.0M(464.0M)->0.0B(448.0M) Survivors: 48.0M->64.0M Heap: 716.9M(10.0G)->259.2M(10.0G)]

...

[Eden: 480.0M(480.0M)->0.0B(1776.0M) Survivors: 32.0M->48.0M Heap: 718.4M(10.0G)->249.4M(10.0G)]
[Eden: 1776.0M(1776.0M)->0.0B(432.0M) Survivors: 48.0M->80.0M Heap: 2025.4M(10.0G)->281.9M(10.0G)]
[Eden: 432.0M(432.0M)->0.0B(464.0M) Survivors: 80.0M->48.0M Heap: 713.9M(10.0G)->261.8M(10.0G)]

...

[Eden: 480.0M(480.0M)->0.0B(464.0M) Survivors: 32.0M->48.0M Heap: 724.1M(10.0G)->252.6M(10.0G)]
[Eden: 464.0M(464.0M)->0.0B(1280.0M) Survivors: 48.0M->48.0M Heap: 716.6M(10.0G)->259.4M(10.0G)]
[Eden: 1280.0M(1280.0M)->0.0B(432.0M) Survivors: 48.0M->80.0M Heap: 1539.4M(10.0G)->283.3M(10.0G)]

[Eden: 448.0M(448.0M)->0.0B(480.0M) Survivors: 64.0M->32.0M Heap: 721.2M(10.0G)->247.1M(10.0G)]
[Eden: 480.0M(480.0M)->0.0B(464.0M) Survivors: 32.0M->48.0M Heap: 727.1M(10.0G)->256.1M(10.0G)]

...

[Eden: 464.0M(464.0M)->0.0B(1504.0M) Survivors: 48.0M->48.0M Heap: 720.1M(10.0G)->256.7M(10.0G)]
[Eden: 1504.0M(1504.0M)->0.0B(416.0M) Survivors: 48.0M->96.0M Heap: 1760.7M(10.0G)->306.1M(10.0G)]
[Eden: 416.0M(416.0M)->0.0B(464.0M) Survivors: 96.0M->48.0M Heap: 722.1M(10.0G)->268.9M(10.0G)]



从内存变化情况可以看出 Survivors 变化不大，Eden 的容量在规律的 480左右、1280、1504、3648 甚至达到了 6096M 大小，此时 hep 内存到了 6468.5M (6.3G)，内存占用也随之增大，最大到了 7G 以上，此处看 sentinel 的统计，并什么出现流量增大的情况，下面是 6096.0M 时，前后几次发生的 GC 情况

| 日期                    | GC 间隔 （大约值 s） | 耗时 | 空间转移                                                     | 归入 S | 归入 O |
| ----------------------- | -------------------- | ---- | ------------------------------------------------------------ | ------ | ------ |
| 2021-04-28T11:42:08.550 |                      | 0.04 | [Eden: 464.0M(464.0M)->0.0B(464.0M) Survivors: 48.0M->48.0M Heap: 834.5M(10.0G)->371.4M(10.0G)] | 0      | 0      |
| 2021-04-28T11:42:24.117 | 16                   | 0.04 | [Eden: 464.0M(464.0M)->0.0B(464.0M) Survivors: 48.0M->48.0M Heap: 835.4M(10.0G)->371.7M(10.0G)] | 0      | 0      |
| 2021-04-28T11:42:36.812 | 12                   | 0.04 | [Eden: 464.0M(464.0M)->0.0B(464.0M) Survivors: 48.0M->48.0M Heap: 835.7M(10.0G)->371.9M(10.0G)] | 0      | 0      |
| 2021-04-28T11:42:49.416 | 13                   | 0.03 | [Eden: 464.0M(464.0M)->0.0B(6096.0M) Survivors: 48.0M->48.0M Heap: 835.9M(10.0G)->372.5M(10.0G)] | 0      | 0      |
| 2021-04-28T11:46:44.194 | 3m55s                | 0.55 | [Eden: 6096.0M(6096.0M)->0.0B(256.0M) Survivors: 48.0M->256.0M Heap: 6468.5M(10.0G)->575.3M(10.0G)] | 208    | 0      |
| 2021-04-28T11:46:54.198 | 10                   | 0.04 | [Eden: 256.0M(256.0M)->0.0B(464.0M) Survivors: 256.0M->48.0M Heap: 831.3M(10.0G)->370.2M(10.0G)] | 0      | 0      |
| 2021-04-28T11:47:17.042 | 23                   | 0.06 | [Eden: 464.0M(464.0M)->0.0B(464.0M) Survivors: 48.0M->48.0M Heap: 834.2M(10.0G)->380.9M(10.0G)] |        |        |
| 2021-04-28T11:47:45.305 | 28                   | 0.06 | [Eden: 464.0M(464.0M)->0.0B(464.0M) Survivors: 48.0M->48.0M Heap: 844.9M(10.0G)->371.6M(10.0G)] |        |        |



  从 GC 日志可以看出，内存变大主要发生在 Eden 区扩容上，在 G1 回收器上，Eden 的大小会随着算法调整，当 Eden 区扩大时，内存占用变大，从大多数 GC 日志可以看出 Eden 的内存大小保持在 480 M 是没有问题的，固暂定解决方案调整 Eden 去的最大大小，先改为一半进行测试



# 6 G1 gc 日志内容

```
2021-04-28T15:21:59.099+0800: 2937.835: [GC pause (G1 Evacuation Pause) (young), 0.0661521 secs]
   [Parallel Time: 23.1 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 2937838.7, Avg: 2937838.8, Max: 2937838.8, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 6.2, Avg: 6.4, Max: 6.5, Diff: 0.3, Sum: 25.5]
      [Update RS (ms): Min: 7.6, Avg: 7.6, Max: 7.6, Diff: 0.0, Sum: 30.2]
         [Processed Buffers: Min: 87, Avg: 102.0, Max: 114, Diff: 27, Sum: 408]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.4, Diff: 0.4, Sum: 0.5]
      [Object Copy (ms): Min: 8.5, Avg: 8.8, Max: 9.0, Diff: 0.5, Sum: 35.3]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.2]
      [GC Worker Total (ms): Min: 22.9, Avg: 22.9, Max: 23.0, Diff: 0.1, Sum: 91.8]
      [GC Worker End (ms): Min: 2937861.7, Avg: 2937861.7, Max: 2937861.7, Diff: 0.0]
   [Code Root Fixup: 0.5 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.3 ms]
   [Other: 42.3 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 36.7 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.2 ms]
      [Humongous Register: 0.3 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.3 ms]
   [Eden: 480.0M(480.0M)->0.0B(464.0M) Survivors: 32.0M->48.0M Heap: 706.0M(10.0G)->241.6M(10.0G)]
 [Times: user=0.13 sys=0.01, real=0.07 secs]
```

- **2021-04-28T15:21:59.099+0800**  gc 发生时间

- **2937.835**  JVM进程启动之后经历的时间

- **GC pause (G1 Evacuation Pause)** ：这个是收集器把存活对象从一个区域拷贝到另一个区域的阶段

- **(Young)**：说明这是个YoungGC

- **GC Workers：4**：说明有 4 个GC线程

- **[Eden: 480.0M(480.0M)->0.0B(464.0M) Survivors: 32.0M->48.0M Heap: 706.0M(10.0G)->241.6M(10.0G)]** ：这一行说的是堆的大小变化。

  **[Eden: 480.0M(480.0M)->0.0B(464.0M)** ：回收前，Eden 区总共 480M并且全部占满，回收以后，Eden区的 480M 全部被回收，同时Eden区扩容到 464M

  **Survivors: 32.0M->48.0M** ：回收前，Survivor区是 32M，回收以后变成 48M，说明 Eden 区有 16M 没有被回收的，放到了Survivor

  **Heap: 706.0M(10.0G)->241.6M(10.0G)** ：整个堆回收前占用了 706.0 M，总大小是 10G，回收以后，堆占用 241.6M，整个堆还是 10G，其中回收了5M。(ps：经过GC，Young 区从 480 + 32 变成 0 + 48，Young 区减少 464 M，整个堆从 706 变成 241.6 减少 464.4 M，考虑下四舍五入的误差，可以证明未升级到 old 区)

- **[Times: user=0.13 sys=0.01, real=0.07 secs]**：注意下 real 时间，说明本次GC收集器花了 0.07 秒。









