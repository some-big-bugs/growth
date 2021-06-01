# Java 各个版本特性


## Java 5

1. 引入泛型功能（伪泛型）
2. 增强循环，可以使用迭代循环，Iterator
3. 支持自动装箱和自动拆箱
4. 支持类型安全的枚举
5. 支持可变参数
6. 支持静态引入
7. 支持元数据（注解）功能
8. 引入Java Instrumentation

## Java 6 

1. 支持脚本语言
2. 引入JDBC 4.0 API
3. 引入Java Compiler API
4. 支持可插拔注解

## Java 7

1. 支持了switch语句以String作为条件，至此已支持4中基本类型(byte, char , short ,int)，2种对象类型(枚举，String)
2. 优化增强了泛型对象类型推断
3. 支持了在一个语句块中捕获多种异常，既一个catch，可以|多个异常
4. 支持try-with-resources语法，自动为资源类进行关闭，而无需finally进行
5. JSR203 引入Java NIO2开发包，支持了AIO
6. 支持了数值的二进制表示方式，比如0b1010000, 以前只能8,10,16进制的表示
7. 钻石型语法，省略类型参数的声明
8. JSR292 支持了动态语言 InvokeDynamic
9. 数字字面量的改进 / 数值可加下划
10. Path接口、DirectoryStream、Files、WatchService（重要接口更新）
11. fork/join framework

## Java 8

1. 支持lambda表达式
2. 支持集合的Stream流处理
3. 支持Functional函数式接口
4. 对各个类支持了很多对应的lambda增强方法
5. 增强接口，支持了接口的默认和静态的具体方法
6. 支持LocalDate等时间包，以改进原有Date
7. 支持Optional来改进Null值的处理
8. 优化了HashMap和ConcurrentHashMap

## Java 9

1. 支持Java平台级模块系统，既模块化
2. 支持了JShell，既类似node.js, python一样的命令行工具，对待简单的东西，可以直接命令测试
3. 改进Javadoc, 使得Javadoc可以搜索
4. 支持了List.of(), Set.of(), Map.of()的方式初始化不可变集合，省略了大量代码，语法糖
5. 改进的Stream API，比如ofNullable(),dropWhile(),takeWhile()等
6. 增强接口，支持了私有接口具体方法，补充了Java8默认和静态方法的支持
7. 把G1垃圾收集器作为默认的垃圾收集器，并标注CMS为过时收集器
8. 引入了响应式流Reactive Streams API, 支持了响应式编程
9. 支持了HTTP/2客户端

## Java 10

1. 支持了局部变量的类型推导，支持了局部变量的var声明
2. 优化改进了G1垃圾回收器，采用并行化mark-sweep-compact算法

## Java 11
1. 简化了启动单个源代码文件的方法，使得小白命令式编译运行Java文件变成更加简单，java Helloworld.java 即可
2. 增加支持了一个叫Epsilon的低开销垃圾收集器
3. 增加支持了一个叫ZGC(Z Garbage Collector)的可伸缩低延迟垃圾收集器, 相比G1，停顿时间有了很大的改进，稳定在10ms以下
4. 重写了HttpClient,提供了新的标准化HttpClient API, 以后不再需要引入apache包的HttpClient或是okhttp就能支持高性能的网络编程
6. 增加了对TLS 1.3的支持
7. 允许了飞行记录器进行商用下的使用
8. 更好的支持了动态语言，在JVM层面增加了invokedynamic指令
9. 支持了在lambda内部使用var声明局部变量

## Java 12 

1. 引入了一个叫Shenandoah 低停顿的实验性垃圾收集器
2. 改进了G1垃圾收集器
3. 引入了实验性的预览版Switch语句优化，可以省略break语句，合并多个case
4. 引入了JVM的常量API, 有助于一些分析的实现

## Java 13

1. 增强优化了ZGC垃圾收集器, 归还未使用的堆内存给操作系统
2. 将Java12引入的预览版Switch语法提为正式版，使得Switch的使用更加的顺滑和流畅
3. 支持了"""字符串块功能

## Java 14

305: instanceof的模式匹配 (预览)
343: 打包工具 (Incubator)
345: G1的NUMA内存分配优化
349: JFR事件流
352: 非原子性的字节缓冲区映射
358: 友好的空指针异常
359: Records (预览)
361: Switch表达式 (标准)
362: 弃用Solaris和SPARC端口
363: 移除CMS（Concurrent Mark Sweep）垃圾收集器
364: macOS系统上的ZGC
365: Windows系统上的ZGC
366: 弃用ParallelScavenge + SerialOld GC组合
367: 移除Pack200 Tools和API
368: 文本块 (第二个预览版)
370: 外部存储器API (Incubator)

## Java 15

http://openjdk.java.net/projects/jdk/15/

对应中文特性：(JEP：JDK Enhancement Proposals，JDK 增强建议，也就是 JDK 的特性新增和改进提案。) 

JEP 339：EdDSA 数字签名算法 
JEP 360：密封类（预览） 
JEP 371：隐藏类 
JEP 372：移除 Nashorn JavaScript 引擎 
JEP 373：重新实现 Legacy DatagramSocket API 
JEP 374：禁用偏向锁定 
JEP 375：instanceof 模式匹配（第二次预览） 
JEP 377：ZGC：一个可扩展的低延迟垃圾收集器 
JEP 378：文本块 
JEP 379：Shenandoah:低暂停时间垃圾收集器 
JEP 381：移除 Solaris 和 SPARC 端口 
JEP 383：外部存储器访问 API（第二次孵化版） 
JEP 384：Records（第二次预览） 
JEP 385：废弃 RMI 激活机制




# 参考



参考信息

IBM Developer Java9 www.ibm.com/developerwo…

Guide to Java10 www.baeldung.com/java-10-ove…

Java 10 新特性介绍www.ibm.com/developerwo…

IBM Devloper Java11 www.ibm.com/developerwo…

Java 11 – Features and Comparison： www.geeksforgeeks.org/java-11-fea…

Oracle Java12 ReleaseNote www.oracle.com/technetwork…

Oracle Java13 ReleaseNote www.oracle.com/technetwork…

New Java13 Features www.baeldung.com/java-13-new…

Java13 新特性概述 www.ibm.com/developerwo…

Oracle Java14 record docs.oracle.com/en/java/jav…

java14-features www.techgeeknext.com/java/java14…
