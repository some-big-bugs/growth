# FeignClient 使用

为了测试方便，这里提供四个项目

1. user-server
2. user-server-api
3. spring-boot-feign
4. spring-mvc-feign


## Spring Cloud 使用


**user-server项目**

提供服务，暂时提供三个简单的查询操作

**Controller**
```java
package com.zyndev.server.user.controller;

import com.zyndev.commontool.web.BaseResponse;
import com.zyndev.server.user.entity.UserEntity;
import com.zyndev.server.user.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * @author: 张瑀楠 zyndev@gmail.com
 */
@RestController
@RequestMapping("user")
public class UserController {

    @Autowired
    private IUserService userService;

    @DeleteMapping("deleteById")
    public BaseResponse deleteByID(@RequestParam("id")Integer id) {
        try {
            Integer retVal = userService.deleteByID(id);
            if (retVal == null || retVal == 0) {
                return BaseResponse.fail("删除失败");
            }
            return BaseResponse.success(null);
        } catch (Exception e) {
            return BaseResponse.fail(e.getMessage());
        }
    }


    @GetMapping("getUserByUID")
    public BaseResponse getUserByID(@RequestParam("id")Integer id) {
        try {
            UserEntity userEntity = userService.getUserByID(id);
            if (userEntity == null) {
                return BaseResponse.fail("未找到用户信息");
            }
            return BaseResponse.success(userEntity);
        } catch (Exception e) {
            return BaseResponse.fail(e.getMessage());
        }
    }
}
```

**user-server-api**

提供一个消费方使用服务的API

```java
package com.zyndev.server.user.api;

import com.zyndev.commontool.web.BaseResponse;
import com.zyndev.server.user.hystrix.UserServiceHystrix;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * Desc: 
 * @author: 张瑀楠 zyndev@gmail.com 
 * TODO:
 */
@FeignClient(name = "user-server",url = "${user-server-api.url}", path = "user", fallback = UserServiceHystrix.class)
public interface UserServiceAPI {

    @DeleteMapping("deleteById")
    public BaseResponse deleteByID(@RequestParam("id")Integer id);

    @GetMapping("getUserByUID")
    public BaseResponse getUserByID(@RequestParam("id")Integer id);
}
```

其中通过动态配置 `url` 来实现在 `spring cloud` 外部环境调用接口

**FeignClient注解源码**

```java
package org.springframework.cloud.netflix.feign;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.core.annotation.AliasFor;

/**
 * Annotation for interfaces declaring that a REST client with that interface should be
 * created (e.g. for autowiring into another component). If ribbon is available it will be
 * used to load balance the backend requests, and the load balancer can be configured
 * using a <code>@RibbonClient</code> with the same name (i.e. value) as the feign client.
 *
 * @author Spencer Gibb
 * @author Venil Noronha
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {

	/**
	 * The name of the service with optional protocol prefix. Synonym for {@link #name()
	 * name}. A name must be specified for all clients, whether or not a url is provided.
	 * Can be specified as property key, eg: ${propertyKey}.
	 */
	@AliasFor("name")
	String value() default "";

	/**
	 * The service id with optional protocol prefix. Synonym for {@link #value() value}.
	 *
	 * @deprecated use {@link #name() name} instead
	 */
	@Deprecated
	String serviceId() default "";

	/**
	 * The service id with optional protocol prefix. Synonym for {@link #value() value}.
	 */
	@AliasFor("value")
	String name() default "";
	
	/**
	 * Sets the <code>@Qualifier</code> value for the feign client.
	 */
	String qualifier() default "";

	/**
	 * An absolute URL or resolvable hostname (the protocol is optional).
	 */
	String url() default "";

	/**
	 * Whether 404s should be decoded instead of throwing FeignExceptions
	 */
	boolean decode404() default false;

	/**
	 * A custom <code>@Configuration</code> for the feign client. Can contain override
	 * <code>@Bean</code> definition for the pieces that make up the client, for instance
	 * {@link feign.codec.Decoder}, {@link feign.codec.Encoder}, {@link feign.Contract}.
	 *
	 * @see FeignClientsConfiguration for the defaults
	 */
	Class<?>[] configuration() default {};

	/**
	 * Fallback class for the specified Feign client interface. The fallback class must
	 * implement the interface annotated by this annotation and be a valid spring bean.
	 */
	Class<?> fallback() default void.class;

	/**
	 * Define a fallback factory for the specified Feign client interface. The fallback
	 * factory must produce instances of fallback classes that implement the interface
	 * annotated by {@link FeignClient}. The fallback factory must be a valid spring
	 * bean.
	 *
	 * @see feign.hystrix.FallbackFactory for details.
	 */
	Class<?> fallbackFactory() default void.class;

	/**
	 * Path prefix to be used by all method-level mappings. Can be used with or without
	 * <code>@RibbonClient</code>.
	 */
	String path() default "";

	/**
	 * Whether to mark the feign proxy as a primary bean. Defaults to true.
	 */
	boolean primary() default true;

}

```

在源码中可以看到比较有用的四个注解 `name` ,`url`, `fallback` ,`path`

- name 指定微服务的实例名称，唯一，必填，通过实例名称可以得到实例对应的访问地址
- fallback 配置熔断
- url 配置一个绝对的地址访问，默认为空字符串，当其不空时，则使用该地址访问
- path 配置一个所有方法级别的mappings 相当于在类上加 requestMapping, 例如上面的`UserServiceAPI` 所有访问地址为 /user/xxx 

注意：`FeignClient` 请求路径和 `包名` 无关, 
```java
/user/xxx1
/user/xxx2
/user/xxx3
```

如果想放着以上地址，api 有三种实现方式

1. 在所有的方法中 写明全路径 例如 @RequestMapping("/user/xxx1")
1. 在类上写 @RequestMapping("user") 在对应方法写 @RequestMapping("xxx1")
1. 使用 FeignClient 的 path 标注

如果在**spring cloud**项目内部使用，可以直接引入 `user-server-api`

```xml
<dependency>
	<groupId>com.zyndev</groupId>
	<artifactId>user-server-api</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</dependency>
```

需要注意的是，因为在 **feignclient** 中使用`占位符`,所以你需要在配置文件中添加

> user-server-api.url=

否则会报出如下异常信息

```java
org.springframework.beans.factory.BeanDefinitionStoreException: Invalid bean definition with name 'com.zyndev.server.user.ap
i.UserServiceAPI' defined in null: Could not resolve placeholder 'user-server-api.url' in value "http://${user-server-api.url}"; nested exception is java.lang.IllegalArgumentException: Could not resolve placeholder 'user-server-api.url' in value "http://${user-server-api.url}"
	at org.springframework.beans.factory.config.PlaceholderConfigurerSupport.doProcessProperties(PlaceholderConfigurerSupport.java:223) ~[spring-beans-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	at org.springframework.context.support.PropertySourcesPlaceholderConfigurer.processProperties(PropertySourcesPlaceholderConfigurer.java:180) ~[spring-context-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	at org.springframework.context.support.PropertySourcesPlaceholderConfigurer.postProcessBeanFactory(PropertySourcesPlaceholderConfigurer.java:152) ~[spring-context-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:281) ~[spring-context-4.3.11.RELEASE.jar:4.3.11.RELEASE]
	at org.springf
```


## Spring boot 使用

这里的spring boot项目值的是不需要注册到微服务中，单独的项目

1. 首先引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>com.zyndev</groupId>
    <artifactId>user-server-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>

```

2. 在配置文件添加如下配置
其中后面的地址为网关访问地址
```
user-server-api.url=192.168.0.101:8089/api/user-server/
```

3. 在启动类中添加注解

```java

@EnableFeignClients(basePackages={"com.zyndev.server.user.api"})
@SpringBootApplication
public class SpringBootFeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootFeignApplication.class, args);
	}
}

```


## 传统Spring项目使用

这里的传统`Spring`项目指的是没有使用`spring boot`的`spring`项目，例如`ssm`

**精力有限只测试了 spring mvc 项目**


1. 配置
如果使用非 spring cloud，则应该在 api 的 FeignClient 注解上设置 url,例如例子程序



在项目配置 properties 文件，这里我使用 server.properties

下面是我测试的时候自己起的 网关地址

**server.properties**
```java
user-server-api.url=192.168.0.101:8089/api/user-server/
```

这里配置的 spring mvc 项目，配置servlet 主要为了加载 `application.xml`
**web.xml**
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <display-name>Archetype Created Web Application</display-name>

    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/application.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

**application.xml**

这里主要为了加载属性文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 包扫描，暂时扫描全部包 -->
    <context:component-scan base-package="com.renren" />

    <context:property-placeholder location="classpath:microserver/server.properties"></context:property-placeholder>

    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/>

    <mvc:default-servlet-handler/>
    <mvc:annotation-driven/>

</beans>
```

**添加配置类**
```java
package com.renren.config;

import com.google.gson.Gson;
import org.springframework.boot.autoconfigure.ImportAutoConfiguration;
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.cloud.netflix.feign.EnableFeignClients;
import org.springframework.cloud.netflix.feign.FeignAutoConfiguration;
import org.springframework.cloud.netflix.feign.ribbon.FeignRibbonClientAutoConfiguration;
import org.springframework.cloud.netflix.ribbon.RibbonAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.GsonHttpMessageConverter;

import java.util.ArrayList;
import java.util.Collection;

/**
 * @author 张瑀楠 zyndev@gmail.com
 * @version 1.0
 * time: 2017/12/13 11:40
 * TODO:
 */
@ImportAutoConfiguration({RibbonAutoConfiguration.class, FeignRibbonClientAutoConfiguration.class, FeignAutoConfiguration.class})
@EnableFeignClients(basePackages={"com.zyndev.server.user.api"})
@Configuration
public class Config {

    @Bean
    public HttpMessageConverters customConverters() {

        Collection<HttpMessageConverter<?>> messageConverters = new ArrayList<>();

        GsonHttpMessageConverter gsonHttpMessageConverter = new GsonHttpMessageConverter();
        messageConverters.add(gsonHttpMessageConverter);

        return new HttpMessageConverters(true, messageConverters);
    }

}
```

EnableFeignClients 设置对应的 api 路径，可设置多个，其中要配置一个 `HttpMessageConverters` 用来解码，可设置其他

```java
package com.renren.controller;

import com.zyndev.commontool.web.BaseResponse;
import com.zyndev.server.user.api.UserServiceAPI;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

/**
 * @author 张瑀楠 zyndev@gmail.com
 * @version 1.0
 * time: 2017/12/13 16:23
 * TODO:
 */
@RestController
public class IndexController {

    @Autowired
    private UserServiceAPI userServiceAPI;

    @RequestMapping("testGetUserById")
    public Map testApi(@RequestParam("id") Integer id) {
        Map<String, Object> result = new HashMap<>();
        result.put("name", "testApi");
        if (userServiceAPI == null) {
            System.out.println("userServiceAPI is null");
        }
        else {
            BaseResponse baseResponse = userServiceAPI.getUserByID(id);
            result.put("response", baseResponse);
        }
        return result;
    }

}
```

**userServiceAPI直接注入即可**

**对应pom文件**
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.renren</groupId>
    <artifactId>spring-feign</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>spring-feign Maven Webapp</name>
    <url>http://maven.apache.org</url>

    <properties>
        <spring.version>4.3.8.RELEASE</spring.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.32</version>
        </dependency>

        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.3.1</version>
        </dependency>

        <dependency>
            <groupId>com.zyndev</groupId>
            <artifactId>user-server-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-eureka</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <build>
        <finalName>spring-feign</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

> 具体代码见：https://github.com/zyndev/zyndev-spring-cloud



