# Java 解决跨域问题

---

**Table of Contents**

- [引言](#引言)
  - [什么是跨域（CORS）](#什么是跨域cors)
  - [什么情况会跨域](#什么情况会跨域)
- [解决方案](#解决方案)
  - [前端解决方案](#前端解决方案)
  - [后端解决方案](#后端解决方案)
- [具体方式](#具体方式)
  - [一、使用Filter方式进行设置](#一使用filter方式进行设置)
  - [二、继承 HandlerInterceptorAdapter](#二继承-handlerinterceptoradapter)
  - [三、实现 WebMvcConfigurer](#三实现-webmvcconfigurer)
  - [四、使用Nginx配置](#四使用nginx配置)
  - [五、使用 `@CrossOrgin` 注解](#五使用-crossorgin-注解)
- [Spring Cloud Gateway 跨域配置](#spring-cloud-gateway-跨域配置)

---

# 引言

我们在开发过程中经常会遇到前后端分离而导致的跨域问题，导致无法获取返回结果。跨域就像分离前端和后端的一道鸿沟，君在这边，她在那边，两两不能往来.

## 什么是跨域（CORS）
跨域（CORS）是指不同域名之间相互访问。跨域，指的是浏览器不能执行其他网站的脚本，它是由浏览器的同源策略所造成的，是浏览器对于JavaScript所定义的安全限制策略。

## 什么情况会跨域

- 同一协议， 如http或https
- 同一IP地址, 如127.0.0.1
- 同一端口, 如8080

以上三个条件中有一个条件不同就会产生跨域问题。

# 解决方案

## 前端解决方案

1. 使用JSONP方式实现跨域调用；
2. 使用NodeJS服务器做为服务代理，前端发起请求到NodeJS服务器， NodeJS服务器代理转发请求到后端服务器；

## 后端解决方案
- nginx反向代理解决跨域
- 服务端设置Response Header(响应头部)的Access-Control-Allow-Origin
- 在需要跨域访问的类和方法中设置允许跨域访问（如Spring中使用@CrossOrigin注解）；
- 继承使用Spring Web的CorsFilter（适用于Spring MVC、Spring Boot）
- 实现WebMvcConfigurer接口（适用于Spring Boot）

# 具体方式
## 一、使用Filter方式进行设置

使用Filter过滤器来过滤服务请求，向请求端设置Response Header(响应头部)的Access-Control-Allow-Origin属性声明允许跨域访问。

```java
@WebFilter
public class CorsFilter implements Filter {  

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {  
        HttpServletResponse response = (HttpServletResponse) res;  
        response.setHeader("Access-Control-Allow-Origin", "*");  
        response.setHeader("Access-Control-Allow-Methods", "*");  
        response.setHeader("Access-Control-Max-Age", "3600");  
        response.setHeader("Access-Control-Allow-Headers", "*");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        chain.doFilter(req, res);  
    }  
}
```

## 二、继承 HandlerInterceptorAdapter 
```java
@Component
public class CrossInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers", "*");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        return true;
    }
}

```

## 三、实现 WebMvcConfigurer

```java
@Configuration
@SuppressWarnings("SpringJavaAutowiredFieldsWarningInspection")
public class AppConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")  // 拦截所有的请求
                .allowedOrigins("http://www.abc.com")  // 可跨域的域名，可以为 *
                .allowCredentials(true)
                .allowedMethods("*")   // 允许跨域的方法，可以单独配置
                .allowedHeaders("*");  // 允许跨域的请求头，可以单独配置
    }
}

```

## 四、使用Nginx配置
```
location / {
   add_header Access-Control-Allow-Origin *;
   add_header Access-Control-Allow-Headers X-Requested-With;
   add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;

   if ($request_method = 'OPTIONS') {
     return 204;
   }
}
```

## 五、使用 `@CrossOrgin` 注解
如果只是想部分接口跨域，且不想使用配置来管理的话，可以使用这种方式

**在Controller使用**
```
@CrossOrigin
@RestController
@RequestMapping("/user")
public class UserController {

	@GetMapping("/{id}")
	public User get(@PathVariable Long id) {
		
	}

	@DeleteMapping("/{id}")
	public void remove(@PathVariable Long id) {

	}
}
```

**在具体接口上使用**
```
@RestController
@RequestMapping("/user")
public class UserController {

	@CrossOrigin
	@GetMapping("/{id}")
	public User get(@PathVariable Long id) {
		
	}

	@DeleteMapping("/{id}")
	public void remove(@PathVariable Long id) {

	}
}
```


# Spring Cloud Gateway 跨域配置

```yaml
spring: 
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            # 允许跨域的源(网站域名/ip)，设置*为全部
            # 允许跨域请求里的head字段，设置*为全部
            # 允许跨域的method， 默认为GET和OPTIONS，设置*为全部
            allow-credentials: true
            allowed-origins:
              - "http://xb.abc.com"
              - "http://sf.xx.com"
            allowed-headers: "*"
            allowed-methods:
              - OPTIONS
              - GET
              - POST
              - DELETE
              - PUT
              - PATCH
            max-age: 3600
```

**注意：** 通过`gateway` 转发的其他项目，不要进行配置跨域配置

有时即使配置了也不会起作用，这时你可以根据浏览器控制的错误输出来查看问题，如果提示是 `response` 中 header 出现了重复的 `Access-Control-*` 请求头，可以进行如下操作

```java
import java.util.ArrayList;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.cloud.gateway.filter.NettyWriteResponseFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpHeaders;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component("corsResponseHeaderFilter")
public class CorsResponseHeaderFilter implements GlobalFilter, Ordered {

  @Override
  public int getOrder() {
    // 指定此过滤器位于NettyWriteResponseFilter之后
    // 即待处理完响应体后接着处理响应头
    return NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER + 1;
  }

  @Override
  public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    return chain.filter(exchange).then(Mono.defer(() -> {
      exchange.getResponse().getHeaders().entrySet().stream()
          .filter(kv -> (kv.getValue() != null && kv.getValue().size() > 1))
          .filter(kv -> (
              kv.getKey().equals(HttpHeaders.ACCESS_CONTROL_ALLOW_ORIGIN)
                  || kv.getKey().equals(HttpHeaders.ACCESS_CONTROL_ALLOW_CREDENTIALS)
                  || kv.getKey().equals(HttpHeaders.ACCESS_CONTROL_ALLOW_METHODS)
                  || kv.getKey().equals(HttpHeaders.ACCESS_CONTROL_ALLOW_HEADERS)
                  || kv.getKey().equals(HttpHeaders.ACCESS_CONTROL_MAX_AGE)))
          .forEach(kv -> {
            kv.setValue(new ArrayList<String>() {{
              add(kv.getValue().get(0));
            }});
          });
      return chain.filter(exchange);
    }));
  }
}
```