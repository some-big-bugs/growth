<h1> Guava EventBus 使用 </h1>

# 系列文章目录

---

**Table of Contents**
- [系列文章目录](#系列文章目录)
- [前言](#前言)
- [一、为什么要用 Observer模式以及 EventBus 优点 ？](#一为什么要用-observer模式以及-eventbus-优点-)
- [二、EventBus 使用步骤](#二eventbus-使用步骤)
  - [1. 引入库](#1-引入库)
  - [2. 同步使用](#2-同步使用)
  - [3. 异步使用](#3-异步使用)
  - [4. 异常处理](#4-异常处理)
- [总结](#总结)
- [参考](#参考)

---

# 前言

EventBus 是 Guava 的事件处理机制，是观察者模式（生产/消费模型）的一种实现。

观察者模式在我们日常开发中使用非常广泛，例如在订单系统中，订单状态或者物流信息的变更会向用户发送APP推送、短信、通知卖家、买家等等；审批系统中，审批单的流程流转会通知发起审批用户、审批的领导等等。

Observer模式也是 JDK 中自带就支持的，其在 1.0 版本就已经存在 Observer，不过随着 Java 版本的飞速升级，其使用方式一直没有变化，许多程序库提供了更加简单的实现，例如 Guava EventBus、RxJava、EventBus 等


# 一、为什么要用 Observer模式以及 EventBus 优点 ？


**EventBus 优点**

- 相比 Observer 编程简单方便
- 通过自定义参数可实现同步、异步操作以及异常处理
- 单进程使用，无网络影响

**缺点**

- 只能单进程使用
- 项目异常重启或者退出不保证消息持久化

如果需要分布式使用还是需要使用 `MQ` 
 
# 二、EventBus 使用步骤
## 1. 引入库

**Gradle**
```groovy
compile group: 'com.google.guava', name: 'guava', version: '29.0-jre'
```
**Maven**

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>29.0-jre</version>
</dependency>

```

引入依赖后，这里我们主要使用 `com.google.common.eventbus.EventBus` 类进行操作，其提供了 `register`、`unregister`、`post` 来进行注册订阅、取消订阅和发布消息

```java
public void register(Object object);

public void unregister(Object object);

public void post(Object event);
```

## 2. 同步使用

**1. 首先创建一个 EventBus**

```java
EventBus eventBus = new EventBus();
```

**2. 创建一个订阅者**

在 Guava EventBus 中，是根据参数类型进行订阅，每个订阅的方法只能由一个参数，同时需要使用 `@Subscribe` 标识

```java
class EventListener {

  /**
   * 监听 Integer 类型的消息
   */
  @Subscribe
  public void listenInteger(Integer param) {
    System.out.println("EventListener#listenInteger ->" + param);
  }

  /**
   * 监听 String 类型的消息
   */
  @Subscribe
  public void listenString(String param) {
    System.out.println("EventListener#listenString ->" + param);
  }
}
```
**3. 注册到 EventBus 上并发布消息**

```java
EventBus eventBus = new EventBus();

eventBus.register(new EventListener());

eventBus.post(1);
eventBus.post(2);
eventBus.post("3");
```

运行结果为

```
EventListener#listenInteger ->1
EventListener#listenInteger ->2
EventListener#listenString ->3
```

根据需要我们可以创建多个订阅者完成订阅信息，同时如果一个类型存在多个订阅者，则所有订阅方法都会执行

**为什么说这么做是同步的呢？**



Guava Event 实际上是使用线程池来处理订阅消息的，通过源码可以看出，当我们使用默认的构造方法创建 `EventBus` 的时候，其中 `executor` 为 `MoreExecutors.directExecutor()`，其具体实现中直接调用的 `Runnable#run` 方法，使其仍然在同一个线程中执行，所以默认操作仍然是同步的，这种处理方法也有适用的地方，这样既可以解耦也可以让方法在同一个线程中执行获取同线程中的便利，比如事务的处理


**EventBus 部分源码**

```java
public class EventBus {
  private static final Logger logger = Logger.getLogger(EventBus.class.getName());
  private final String identifier;
  private final Executor executor;
  private final SubscriberExceptionHandler exceptionHandler;
  private final SubscriberRegistry subscribers;
  private final Dispatcher dispatcher;

  public EventBus() {
    this("default");
  }

  public EventBus(String identifier) {
    this(identifier, MoreExecutors.directExecutor(), Dispatcher.perThreadDispatchQueue(), EventBus.LoggingHandler.INSTANCE);
  }

  public EventBus(SubscriberExceptionHandler exceptionHandler) {
    this("default", MoreExecutors.directExecutor(), Dispatcher.perThreadDispatchQueue(), exceptionHandler);
  }

  EventBus(String identifier, Executor executor, Dispatcher dispatcher, SubscriberExceptionHandler exceptionHandler) {
    this.subscribers = new SubscriberRegistry(this);
    this.identifier = (String)Preconditions.checkNotNull(identifier);
    this.executor = (Executor)Preconditions.checkNotNull(executor);
    this.dispatcher = (Dispatcher)Preconditions.checkNotNull(dispatcher);
    this.exceptionHandler = (SubscriberExceptionHandler)Preconditions.checkNotNull(exceptionHandler);
  }
}
```

**DirectExecutor  部分源码**

```java
enum DirectExecutor implements Executor {
  INSTANCE;

  private DirectExecutor() {
  }

  public void execute(Runnable command) {
    command.run();
  }

  public String toString() {
    return "MoreExecutors.directExecutor()";
  }
}
```



## 3. 异步使用
通过上面的源码，可以看出只要将构造方法中的 executor 换成一个线程池实现即可, 同时 Guava EventBus 为了简化操作，提供了一个简化的方案即 `AsyncEventBus`

```java
EventBus eventBus = new AsyncEventBus(Executors.newCachedThreadPool());
```

这样即可实现异步使用

**AsyncEventBus 源码**

```java
public class AsyncEventBus extends EventBus {
  public AsyncEventBus(String identifier, Executor executor) {
    super(identifier, executor, Dispatcher.legacyAsync(), LoggingHandler.INSTANCE);
  }

  public AsyncEventBus(Executor executor, SubscriberExceptionHandler subscriberExceptionHandler) {
    super("default", executor, Dispatcher.legacyAsync(), subscriberExceptionHandler);
  }

  public AsyncEventBus(Executor executor) {
    super("default", executor, Dispatcher.legacyAsync(), LoggingHandler.INSTANCE);
  }
}
```

## 4. 异常处理
如果处理时发生异常应该如何处理? 在看源码中，无论是 `EventBus ` 还是 `AsyncEventBus` 都可传入自定义的 `SubscriberExceptionHandler` 该 handler 当出现异常时会被调用，我可可以从参数 `exception` 获取异常信息，从 `context` 中获取消息信息进行特定的处理

其接口声明为 

```java
public interface SubscriberExceptionHandler {
  /** Handles exceptions thrown by subscribers. */
  void handleException(Throwable exception, SubscriberExceptionContext context);
}
```

# 总结
在上面的基础上，我们可以定义一些消息类型来实现不同消息的监听和处理，通过实现 `SubscriberExceptionHandler` 来处理异常的情况，无论时同步还是异步都能游刃有余 

# 参考
- https://github.com/google/guava
- https://github.com/greenrobot/EventBus
- https://github.com/ReactiveX/RxJava
