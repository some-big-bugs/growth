第三天：Redis核心数据结构与底层设计原理详解
1、Redis五大核心数据存储结构精讲
2、Redis6.0多线程工作模式解析
3、如何实现亿级用户日活统计
4、Redis阻塞队列实现原理
5、如何实现一个高性能的延迟队列
6、Redis ZSet底层跳表实现原理及时间复杂度分析

第四天：亿级流量新浪微博与微信Redis架构实战
1、Redis核心数据结构精讲

18. Redis

19.  redis 有哪些功能？

20.  redis 为什么是单线程的？
21.  什么是缓存穿透？怎么解决？
22.  redis 支持的数据类型有哪些？
23.  redis 支持的 java 客户端都有哪些？
24.  jedis 和 redisson 有哪些区别？
25.  怎么保证缓存和数据库数据的一致性？
26.  redis 持久化有几种方式？
27.  redis 怎么实现分布式锁？
28.  redis 分布式锁有什么缺陷？
29.  redis 如何做内存优化？
30.  redis 淘汰策略有哪些？
31.  redis 常见的性能问题有哪些？该如何解决？


一、在修改配置配置文件之前，可手动执行 bgwriteaof 指令，重写aof文件。文件生成后，再关闭服务，修改配置文件。
二、使用config set appendonly yes 指令进行动态加载，但是这个指令不会修改配置文件。 redis可支持动态加载，但是版本高低支持的指令不同。可先使用 config get * 指令查看当前版本redis可支持动态加载的指令