<h1> Go | Gin 解决跨域问题跨域配置 </h1>

# 系列文章目录

[Go 语言打包静态文件以及如何与Gin一起使用Go-bindata](https://blog.csdn.net/zyndev/article/details/108172504)

---

**Table of Contents**

- [系列文章目录](#系列文章目录)
- [前言](#前言)
- [一、关于跨域解决方案](#一关于跨域解决方案)
- [二、使用步骤](#二使用步骤)
	- [1. 编写一个中间件](#1-编写一个中间件)
	- [2. 使用](#2-使用)
	- [3. 注意事项](#3-注意事项)

---

# 前言
在前后端分离的项目中，经常会遇到跨域问题，遇到问题该如何解决呢？！

# 一、关于跨域解决方案
关于跨域的解决方法，大部分可以分为 2 种

1. nginx反向代理解决跨域
1. 服务端设置Response Header(响应头部)的`Access-Control-Allow-Origin`

对于后端开发来说，第 2 种的操作性更新灵活，这里也讲一下 Gin 是如何做到的

# 二、使用步骤
在 Gin 中提供了 middleware (中间件) 来做到在一个请求前后处理响应的逻辑，这里我们使用中间来做到在每次请求是添加上 `Access-Control-Allow-Origin` 头部


## 1. 编写一个中间件

可以 `middlewares` 包下创建 

```go
package middlewares

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func Cors() gin.HandlerFunc {
	return func(c *gin.Context) {
		method := c.Request.Method
    origin := c.Request.Header.Get("Origin")
		if origin != "" {
			c.Header("Access-Control-Allow-Origin", "*")  // 可将将 * 替换为指定的域名
			c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE, UPDATE")
			c.Header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept, Authorization")
			c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Cache-Control, Content-Language, Content-Type")
			c.Header("Access-Control-Allow-Credentials", "true")
		}
		if method == "OPTIONS" {
			c.AbortWithStatus(http.StatusNoContent)
		}
		c.Next()
	}
}
```

## 2. 使用
```go
r := gin.Default()

r.Use(middlewares.Cors())
```

## 3. 注意事项

需要将 `r.Use(middlewares.Cors())` 在使用路由前进行设置，否则会导致不生效

**反例**

```go
r := gin.Default()
pingGroup := r.Group("ping")
{
	pingGroup.GET("/", Ping)
}
r.Use(middlewares.Cors())
```
这样会导致跨域配置不生效