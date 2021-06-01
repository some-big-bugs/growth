<h1> JVM 命令参考 </h1>

---

**Table of Contents**

- [jps 显示出所有的 JAVA 进程以及 PID](#jps-显示出所有的-java-进程以及-pid)
- [jstat 查看堆内存各部分的使用量，以及加载类的数量](#jstat-查看堆内存各部分的使用量以及加载类的数量)
- [jstack – 用来查看堆栈信息](#jstack--用来查看堆栈信息)
- [参考文档](#参考文档)
---

# jps 显示出所有的 JAVA 进程以及 PID

显示出所有的 JAVA 进程以及 PID.
当未指定 hostid 时，默认查看本机jvm 进程，否者查看指定的hostid机器上的jvm进程，此时hostid所指机器必须开启jstatd服务。 jps 可以列出jvm进程lvmid，主类类名，main函数参数, jvm参数，jar名称等信息。

```s
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
```

**参数**

- -q 仅仅显示VM 标示，不显示jar,class, main参数等信息.
- -l 输出应用程序主类完整 package 名称或 jar 完整名称 
- -v 显示出 jvm 参数
- -m 显示出传递给main函数的参数
- -V 输出通过 .hotsportrc 或 -XX:Flags=<filename> 指定的 jvm 参数
- hostid 指定特定主机，可以是ip地址和域名, 也可以指定具体协议，端口。

**示例**

```
# jps
93793
95554 Jps
93860 Application
95439 Application
527
95438 Launcher

# jps -q
93793
93860
95628
95439
527
95438

# jps -l
93793
93860 /Users/zhangyunan/Library/Application
95439 com.example.aries.Application
95678 sun.tools.jps.Jps
527
95438 org.jetbrains.jps.cmdline.Launcher

jps -lvmV
95793 sun.tools.jps.Jps -lvmV -Dapplication.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home -Xms8m
93793  -Xms128m -Xmx2000m -XX:ReservedCodeCacheSize=240m -XX:+UseConcMarkSweepGC -XX:SoftRefLRUPolicyMSPerMB=50 -ea -XX:CICompilerCount=2 -Dsun.io.useCanonPrefixCache=false -Djdk.http.auth.tunneling.disabledSchemes="" -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Djdk.attach.allowAttachSelf=true -Dkotlinx.coroutines.debug=off -Djdk.module.illegalAccess.silent=true -XX:+UseCompressedOops -Dfile.encoding=UTF-8 -XX:ErrorFile=/Users/zhangyunan/java_error_in_idea_%p.log -XX:HeapDumpPath=/Users/zhangyunan/java_error_in_idea.hprof -Dide.no.platform.update=true -Didea.plugins.path=/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app.plugins -Djb.vmOptionsFile=/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app.vmoptions -Didea.paths.selector=IdeaIC2020.3 -Didea.executable=idea -Didea.platform.prefix=Idea -Didea.vendor.name=JetBrains -Didea.home.path=/Users/zhangyunan/Library/Applicat
93860 /Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app.plugins/idea-spring-tools/lib/server/language-server.jar
95439 com.example.aries.Application -javaagent:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/lib/idea_rt.jar=64466:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/bin -Dfile.encoding=UTF-8
527  -Djava.library.path=/Applications/JetBrains Toolbox.app/Contents/MacOS/Contents -Duser.dir=/Applications/JetBrains Toolbox.app/Contents/MacOS -Xmx256m -Xms8m -Xss256k -XX:+UseStringDeduplication -XX:+UseCompressedOops -XX:+UseSerialGC -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true vfprintf exit abort -DTOOLBOX_VERSION=1.19.7784
95438 org.jetbrains.jps.cmdline.Launcher /Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/lib/jna-platform.jar:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/lib/jna.jar:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/lib/asm-all-9.0.jar:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/lib/maven-resolver-impl-1.3.3.jar:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/lib/guava-29.0-jre.jar:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents/plugins/java/lib/jps-javac-extension-1.jar:/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.15 -Xmx700m -Djava.awt.headless=true -Djava.endorsed.dirs="" -Djdt.compiler.useSingleThread=true -Dpreload.project.path=/Users/zhangyunan/IdeaProjects/aries-java -Dpreload.config.path=/Users/zhangyunan/Library/Application Support/JetBrains/IdeaIC2020.3/options -Dcompile.parallel=false -Drebuild.on.dependency.change=true -Dio.netty.initialSeedUniquifier=2915071559679537949 -Dfile.encoding=UTF-8 -Duser.language=zh -Duser.country=CN -Didea.paths.selector=IdeaIC2020.3 -Didea.home.path=/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app/Contents -Didea.config.path=/Users/zhangyunan/Library/Application Support/JetBrains/IdeaIC2020.3 -Didea.plugins.path=/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203.5981.155/IntelliJ IDEA CE.app.plugins -Djps.log.dir=/Users/zhangyunan/Library/Logs/JetBrains/IdeaIC2020.3/build-log -Djps.fallback.jdk.home=/Users/zhangyunan/Library/Application Support/JetBrains/Toolbox/apps/IDEA-C/ch-0/203
```

# jstat 查看堆内存各部分的使用量，以及加载类的数量

```s
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
```

jstat [-命令选项] [-h标题之前的内容行数] [vmid] [间隔时间/毫秒] [查询次数]

**其中 option 可选及其示例：**

- class 显示加载class的数量，及所占空间等信息

```
Loaded  Bytes  Unloaded  Bytes     Time
  7961 14216.6        0     0.0       3.67
```
- compiler 显示VM实时编译的数量等信息

```
Compiled Failed Invalid   Time   FailedType FailedMethod
    4156      1       0    12.33          1 org/springframework/boot/loader/jar/Handler openConnection
```

- gc 显示gc的信息，查看gc的次数，及时间。其中最后五项，分别是 young gc 的次数，young gc的时间，full gc的次数，full gc的时间，gc的总时间

```
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
12288.0 12288.0  0.0    0.0   206848.0 119412.2  140800.0   13896.3   35416.0 32942.8 4736.0 4310.2      7    0.081   2      0.080    0.161
```

- jstat -gccapacity:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。其他的可以根据这个类推， OC是old 内存的占用量。

```
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
 87040.0 1397760.0 363008.0 12288.0 12288.0 206848.0   175104.0  2796544.0   140800.0   140800.0      0.0 1079296.0  35416.0      0.0 1048576.0   4736.0      7     2
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

`S1`: 幸存者空间1的利用率占该空间当前容量的百分比.

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
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
```

- printcompilation 当前VM执行的信息。
除了以上一个参数外，还可以同时加上 两个数字，

```
Compiled  Size  Type Method
    4272   1558    1 java/util/TreeMap put
```

以上为 jstat -option vmid 示例，这种每次只会获取当时的信息，如果想多次获取，可以配合 interval 参数来获取，例如：每隔 1000 毫秒获取一次 vmid 为 95980 的 gcutil 信息，总共获取 7 次，其中每 2 次样本间打印下表头 `jstat -gcutil -h2 95980 1000 7`

```s
jstat -gcutil -h2 95980 1000 7
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  61.23   9.87  93.02  91.01      7    0.081     2    0.080    0.161
```

其中每个 jdk 版本中 option 有所不同，具体可见最后的参考文档

# jstack – 用来查看堆栈信息

```
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```

示例

`jstack -l 95980`

内容过长，删除了一部分

```s
2020-12-26 23:54:06
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.144-b01 mixed mode):

"DestroyJavaVM" #34 prio=5 os_prio=31 tid=0x00007f94fcdbc800 nid=0xf03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"http-nio-8080-Acceptor" #33 daemon prio=5 os_prio=31 tid=0x00007f94f77ea000 nid=0x9903 runnable [0x000070000c98b000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
	- locked <0x000000077139e090> (a java.lang.Object)
	at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:469)
	at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:71)
	at org.apache.tomcat.util.net.Acceptor.run(Acceptor.java:106)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"http-nio-8080-ClientPoller" #32 daemon prio=5 os_prio=31 tid=0x00007f94f77c2000 nid=0x6c03 runnable [0x000070000c888000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.KQueueArrayWrapper.kevent0(Native Method)
	at sun.nio.ch.KQueueArrayWrapper.poll(KQueueArrayWrapper.java:198)
	at sun.nio.ch.KQueueSelectorImpl.doSelect(KQueueSelectorImpl.java:117)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
	- locked <0x00000007715d6ca8> (a sun.nio.ch.Util$3)
	- locked <0x00000007715d6c98> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000007715d6b78> (a sun.nio.ch.KQueueSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
	at org.apache.tomcat.util.net.NioEndpoint$Poller.run(NioEndpoint.java:711)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"http-nio-8080-exec-10" #31 daemon prio=5 os_prio=31 tid=0x00007f94f77e9800 nid=0x6b03 waiting on condition [0x000070000c785000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x000000077155fc40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
	at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
	at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"http-nio-8080-exec-1" #22 daemon prio=5 os_prio=31 tid=0x00007f94fe812000 nid=0x6203 waiting on condition [0x000070000be6a000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x000000077155fc40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
	at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:108)
	at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:33)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"http-nio-8080-BlockPoller" #21 daemon prio=5 os_prio=31 tid=0x00007f95004ef000 nid=0x6003 runnable [0x000070000bd67000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.KQueueArrayWrapper.kevent0(Native Method)
	at sun.nio.ch.KQueueArrayWrapper.poll(KQueueArrayWrapper.java:198)
	at sun.nio.ch.KQueueSelectorImpl.doSelect(KQueueSelectorImpl.java:117)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
	- locked <0x000000077139fb70> (a sun.nio.ch.Util$3)
	- locked <0x000000077139fae8> (a java.util.Collections$UnmodifiableSet)
	- locked <0x000000077139f718> (a sun.nio.ch.KQueueSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
	at org.apache.tomcat.util.net.NioBlockingSelector$BlockPoller.run(NioBlockingSelector.java:313)

   Locked ownable synchronizers:
	- None

"HikariPool-1 housekeeper" #19 daemon prio=5 os_prio=31 tid=0x00007f94f7972800 nid=0x5c03 waiting on condition [0x000070000bb61000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x000000076e91c418> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"container-0" #17 prio=5 os_prio=31 tid=0x00007f94f9433000 nid=0x5a03 waiting on condition [0x000070000b95b000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at org.apache.catalina.core.StandardServer.await(StandardServer.java:570)
	at org.springframework.boot.web.embedded.tomcat.TomcatWebServer$1.run(TomcatWebServer.java:197)

   Locked ownable synchronizers:
	- None

"Catalina-utility-1" #15 prio=1 os_prio=31 tid=0x00007f95000df800 nid=0xa703 waiting on condition [0x000070000b755000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000006c081d398> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"Attach Listener" #14 daemon prio=9 os_prio=31 tid=0x00007f94fc0c6000 nid=0xa80b waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Service Thread" #9 daemon prio=9 os_prio=31 tid=0x00007f94f8029000 nid=0x4003 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" #5 daemon prio=9 os_prio=31 tid=0x00007f94f7014800 nid=0x3c03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007f94f9035800 nid=0x3a03 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007f94f800c800 nid=0x4d03 in Object.wait() [0x000070000ad34000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000006c001e8c0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
	- locked <0x00000006c001e8c0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
	- None

"Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007f94f901c800 nid=0x4f03 in Object.wait() [0x000070000ac31000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000006c000f050> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x00000006c000f050> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
	- None

"VM Thread" os_prio=31 tid=0x00007f94f8009800 nid=0x3003 runnable

"GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007f94f780d000 nid=0x2407 runnable

"GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007f94f780f800 nid=0x5103 runnable

"VM Periodic Task Thread" os_prio=31 tid=0x00007f94f8032000 nid=0xa903 waiting on condition

JNI global references: 1780
```

线程dump的分析工具：

IBM Thread and Monitor Dump Analyze for Java【https://www.ibm.com/support/pages/ibm-thread-and-monitor-dump-analyzer-java-tmda】 一个小巧的Jar包，能方便的按状态，线程名称，线程停留的函数排序，快速浏览。
http://spotify.github.io/threaddump-analyzer Spotify提供的Web版在线分析工具，可以将锁或条件相关联的线程聚合到一起。

# 参考文档

- Java7 https://docs.oracle.com/javase/7/docs/technotes/tools/
- Java8 https://docs.oracle.com/javase/8/docs/technotes/tools/index.html
- Java9 https://docs.oracle.com/javase/9/tools/tools-and-command-reference.htm#JSWOR596
- Java10 https://docs.oracle.com/javase/10/tools/tools-and-command-reference.htm#JSWOR596
- Java11 https://docs.oracle.com/en/java/javase/11/tools/tools-and-command-reference.html
- Java12 https://docs.oracle.com/en/java/javase/12/tools/tools-and-command-reference.html
- Java13 https://docs.oracle.com/en/java/javase/13/docs/specs/man/index.html
- Java14 https://docs.oracle.com/en/java/javase/14/docs/specs/man/index.html
- Java15 https://docs.oracle.com/en/java/javase/15/docs/specs/man/index.html
- JavaAll https://docs.oracle.com/en/java/javase/index.html