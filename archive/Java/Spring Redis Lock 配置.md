<h1> Spring Redis Lock 配置 </h1>

**Table of Contents**

- [前言](#前言)
- [使用步骤](#使用步骤)
  - [1. 引入库](#1-引入库)
  - [2. 配置 redis](#2-配置-redis)
  - [3. 增加配置](#3-增加配置)
  - [4. 使用](#4-使用)

# 前言

在我们项目经常遇到并发问题，在单个项目中，使用自带的锁即可完成并发控制，在多个项目中，就需要使用分布式锁来解决。这里讲一下使用 `Redis` 来做分布式锁实现方案

# 使用步骤

## 1. 引入库

在 Spring Boot 项目会根据 Spring Boot 依赖管理自动配置版本号

**Maven**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 2. 配置 redis

在 `application-xxx.yml` 中配置

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    timeout: 2500
    password: xxxxx
```

## 3. 增加配置

**RedisLockConfig.java**

```java
import java.util.concurrent.TimeUnit;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.integration.redis.util.RedisLockRegistry;

@Configuration
public class RedisLockConfig {

  @Bean
  public RedisLockRegistry redisLockRegistry(RedisConnectionFactory redisConnectionFactory) {
    return new RedisLockRegistry(redisConnectionFactory, "redis-lock",
        TimeUnit.MINUTES.toMillis(10));
  }
}
```

## 4. 使用

```java

@Autowired
private RedisLockRegistry lockRegistry;

Lock lock = lockRegistry.obtain(key);
boolean locked = false;
try {
  locked = lock.tryLock();
  if (!locked) {
    // 没有获取到锁的逻辑    
  }

  // 获取锁的逻辑
} finally {
  // 一定要解锁
  if (locked) {
    lock.unlock();
  }
}

```



