<h1> Java 入门学习路线 </h1>

**Table of Contents**

- [前言](#%e5%89%8d%e8%a8%80)
- [学习路线以及方法推荐](#%e5%ad%a6%e4%b9%a0%e8%b7%af%e7%ba%bf%e4%bb%a5%e5%8f%8a%e6%96%b9%e6%b3%95%e6%8e%a8%e8%8d%90)
  - [基础篇](#%e5%9f%ba%e7%a1%80%e7%af%87)
    - [1. Java 基础](#1-java-%e5%9f%ba%e7%a1%80)
    - [2. 使用 Git](#2-%e4%bd%bf%e7%94%a8-git)
    - [3. 文件IO的简单使用](#3-%e6%96%87%e4%bb%b6io%e7%9a%84%e7%ae%80%e5%8d%95%e4%bd%bf%e7%94%a8)
    - [4. XML 文件操作](#4-xml-%e6%96%87%e4%bb%b6%e6%93%8d%e4%bd%9c)
- [正确提问](#%e6%ad%a3%e7%a1%ae%e6%8f%90%e9%97%ae)
- [总结](#%e6%80%bb%e7%bb%93)
- [一些好的文档项目推荐](#%e4%b8%80%e4%ba%9b%e5%a5%bd%e7%9a%84%e6%96%87%e6%a1%a3%e9%a1%b9%e7%9b%ae%e6%8e%a8%e8%8d%90)
- [公众号](#%e5%85%ac%e4%bc%97%e5%8f%b7)

# 前言

大一的时候，我开始接触 `C` 语言，对 C 语言的掌握程度仅仅停留在指针这层面。

到了大二我才接触到 `HTML、CSS、JS、Java、Linux` 这些名词，只所以学`Java`,恐怕是因为导员天天喊的原因吧。

很多人在学完 Java 基础之后，不知道后面该如何进行下一步地进行学习，或者不知道如何去学习。我一直在做web服务端和一些简单的前端开发，希望这篇文章对学习 Java 的朋友能有一点作用。

由于我个人能力有限，下面的学习路线只是我个人见解，一定还有很多欠缺的地方。同时不适合大佬学习。

> 我在上学的时候，慕课网是有个 `Java攻城狮路线` 的学习路线，现在再登录，发现没有了，不知道为什么. 下面的一部分路线是根据回忆找的一部分

# 学习路线以及方法推荐

**建议:** 在开始学习的时候，建议使用 **Intellij IDEA** 进行编码，如果已经使用了 **Eclipse**, 还是建议你切换一下.

不要因为上面的建议而强制使用 **Intellij IDEA**，如果你用了 2 天，还是不能开始编码学习，那还是使用你顺手的IDE. 

## 基础篇

### 1. Java 基础

[Java入门第一季](https://www.imooc.com/learn/85), 看完这个视频后，你大概会学会 `Java` 的一些基础语法和基本类型， 这对你以后看书和学习更深入的语法有所帮助. 认真完成每个练习题. 完成后可以考虑看下书.

**《Java 核心技术卷 1/2》** 和 **《Head First Java》** 这两本书都很适合入门，**《Head First Java》** 可能更适合刚刚起步的你，这个时候不建议去读 **《Java 编程思想》**
，你也可以两本书都一起看看，找到适合你的那本

书本先看完前 150 页, 然后对照书本完成下面的视频

[Java入门第二季](https://www.imooc.com/learn/124)

[Java入门第三季](https://www.imooc.com/learn/110)

学完之后，你应该可以做一个简单命令行里面运行的`计算器` 或者 `学生信息管理` 的小软件了

到这里之后，你已经学习了基本的语法知识和面向对象的语法，当然视频是没法满足所有的知识点的，所有还需要多读书，多看官方文档。

一般人到这里就开始迷茫了，我接下来应该学习什么，应该做什么. 我在这个阶段的时候，也不知道自己应该去做什么.

**记得多总结！打好基础！把自己重要的东西都记录下来。** API 文档放在自己可以看到的地方，以备自己可以随时查阅。

### 2. 使用 Git

这时候你也一定写了好多代码，学习一下用`git`来管理你的代码

**step 1: 看看视频，轻松一下**

[git 教程](https://www.imooc.com/learn/1052)

**step 2: 看看教程，系统学习一下**

[git 教程](https://www.liaoxuefeng.com/wiki/896043488029600)

**step 3: 遇到问题，翻翻官网文档**
[git 官网](https://git-scm.com/docs)

### 3. 文件IO的简单使用

[文件传输基础——Java IO流](https://www.imooc.com/learn/123)

检测一下自己的掌握情况，多试试复制、遍历、读取、删除各种文件

### 4. XML 文件操作

通过上面的学习，你已经学会了`Java`简单的语法和文件操作, 我们经常遇到一些特殊的文件，比如`excel、word、ppt、txt、xml` 等格式的文件，这里学一下 `xml` 文件读写. 实际的开发过程中也会经常和这种文件打交道

[Java眼中的XML---文件读取](https://www.imooc.com/learn/171)

[Java眼中的XML 文件写入](https://www.imooc.com/learn/251)

# 正确提问

我们平时任何时候都离不开提问特别是初学的时候，但是真正知道如何正确的提问的人很少。问别人问题前不要来一句“在吗”，你说你问了在吗我是回复好还是不回复好呢 ？

更多关于如何提问的内容，详见 github 上开源版『提问的智慧』 <https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/master/README-zh_CN.md>，抽时间看一下，我想看完之后应该会有很多收获。

**注意：回答是恩情，不回答是本分**

> 很多初学者在提问题的时候，很希望得到回复，但是不是每个人都有空，也可能对方刚好这个问题不会。记住回答你问题是人家在花费宝贵的时间在帮助你，不回答你是别人的本分，并不欠你什么。所以不要恶语相向。别人回答你，即使没有解决你的问题也要谢谢人家。

# 总结

上面只是介绍了一些你以后必须会的，而不是最全的，你在学习的过程中会遇到很多问题，或粗心造成，或半懂装懂造成，遇到问题一定要学会自己解决和总结

首先百度/Google，通过搜索引擎解决不了的话就找身边的朋友或者认识的一些人。

你可以通过以下途径获取你想要的资源和教程，互联网带来的便捷，帮你降低获取知识的门槛。


1. **官网（大概率是英文，多看看就会熟悉了）**。
2. **官网翻译（有很多人在帮你打怪，大部分是中国人翻译，感谢他们）**
3. **书籍（知识更加系统完全，推荐）**。
4. **视频（比较容易理解，推荐，特别是初学的时候。慕课网、极客时间、百度云盘和b站上面有挺多学习视频可以看，只直接在上面搜索关键词就可以了）**。
5. **网上博客（解决某一知识点的问题的时候可以看看）**。

**如果你的老师有相关 Java 后台项目的话，你也可以主动申请参与进来。如果没有，可以网上找一些小需求做一下**

# 一些好的文档项目推荐

> **学习的过程中有一个可以参考的文档很重要，非常有助于自己的学习**。

**注意:** 不要好高骛远，要找自己会的那部分一点一点看，一点一点的学

名称 | 描述
:---|:---
 [free-books](https://github.com/ruanyf/free-books) | 免费书籍
 [awesome](https://github.com/topics/awesome) | awesome 合集
 [javaok](https://github.com/xjjdog/javaok) | 必看！java后端，亮剑诛仙。java发展路线技术要点。
 [JavaGuide](https://github.com/Snailclimb/JavaGuide) |【Java学习+面试指南】 一份涵盖大部分Java程序员所需要掌握的核心知识。
 [advanced-java](https://github.com/doocs/advanced-java) | 互联网 Java 工程师进阶知识完全扫盲：涵盖高并发、分布式、高可用、微服务、海量数据处理等领域知识，后端同学必看，前端同学也可学习
 [CS-Notes](https://github.com/CyC2018/CS-Notes) | 📚 技术面试必备基础知识、Leetcode、计算机操作系统、计算机网络、系统设计、Java、Python、C++
 [interviews](https://github.com/kdn251/interviews) | Everything you need to know to get the job.
 [toBeTopJavaer](https://github.com/hollischuang/toBeTopJavaer) | To Be Top Javaer - Java工程师成神之路
 [fullstack-tutorial](https://github.com/frank-lam/fullstack-tutorial) | 🚀 fullstack tutorial 2019，后台技术栈/架构师之路/全栈开发社区，春招/秋招/校招/面试
 [android-interview-questions](https://github.com/MindorksOpenSource/android-interview-questions) | Your Cheat Sheet For Android Interview - Android Interview Questions
 [AndroidNote](https://github.com/GcsSloop/AndroidNote) | 安卓学习笔记
 [spring-analysis](https://github.com/seaswalker/spring-analysis) | Spring源码阅读
 [SpringCloudLearning](https://github.com/forezp/SpringCloudLearning) |《史上最简单的Spring Cloud教程源码》
 [mit-deep-learning-book-pdf](https://github.com/janishar/mit-deep-learning-book-pdf) | MIT Deep Learning Book in PDF format (complete and parts) by Ian Goodfellow, Yoshua Bengio and Aaron Courville
 [miaosha](https://github.com/qiurunze123/miaosha) | ⭐⭐⭐⭐秒杀系统设计与实现.互联网工程师进阶与分析🙋🐓
 [spring-boot-demo](https://github.com/xkcoding/spring-boot-demo) | spring boot demo 是一个用来深度学习并实战 spring boot 的项目，目前总共包含 63 个集成demo

# 公众号

如果大家想要实时关注我更新的文章，可以关注我的公众号。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020120185644.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p5bmRldg==,size_16,color_FFFFFF,t_70)