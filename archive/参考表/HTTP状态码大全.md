# HTTP状态码大全

HTTP 状态码 HTTP Status Code

---

- [HTTP状态码大全](#http状态码大全)
- [1xx消息](#1xx消息)
  - [100 Continue](#100-continue)
  - [101 Switching Protocols](#101-switching-protocols)
  - [102 Processing](#102-processing)
- [2xx成功](#2xx成功)
  - [200 OK](#200-ok)
  - [201 Created](#201-created)
  - [202 Accepted](#202-accepted)
  - [203 Non-Authoritative Information](#203-non-authoritative-information)
  - [204 No Content](#204-no-content)
  - [205 Reset Content](#205-reset-content)
  - [206 Partial Content](#206-partial-content)
  - [207 Multi-Status](#207-multi-status)
- [3xx重定向](#3xx重定向)
- [4xx客户端错误](#4xx客户端错误)
- [5xx服务器错误](#5xx服务器错误)
- [参考](#参考)

---

HTTP 状态码的英文为 HTTP Status Code。 

下面是常见的HTTP状态码：

- 200 请求成功
- 301 资源（网页等）被永久转移到其它URL
- 302 资源（网页等）被临时转移到其它URL
- 401 用户没有必要的凭据
- 403 服务器已经理解请求，但是拒绝执行它
- 404 请求的资源（网页等）不存在
- 405 请求行中指定的请求方法不能被用于请求相应的资源
- 406 请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体，该请求不可接受
- 408 请求超时
- 500 内部服务器错误
- 502 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应
- 503 由于临时的服务器维护或者过载，服务器当前无法处理请求
- 504 作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器收到响应

分类 | 描述
---|---
1**	| 信息，服务器收到请求，需要请求者继续执行操作
2**	| 成功，操作被成功接收并处理
3**	| 重定向，需要进一步的操作以完成请求
4**	| 客户端错误，请求包含语法错误或无法完成请求
5**	| 服务器错误，服务器在处理请求的过程中发生了错误

# 1xx消息

这一类型的状态码，代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。由于HTTP/1.0协议中没有定义任何1xx状态码，所以除非在某些试验条件下，服务器禁止向此类客户端发送1xx响应。这些状态码代表的响应都是信息性的，标示客户应该采取的其他行动。

## 100 Continue

服务器已经接收到请求头，并且客户端应继续发送请求主体（在需要发送身体的请求的情况下：例如，POST请求），或者如果请求已经完成，忽略这个响应。服务器必须在请求完成后向客户端发送一个最终响应。要使服务器检查请求的头部，客户端必须在其初始请求中发送Expect: 100-continue作为头部，并在发送正文之前接收100 Continue状态代码。响应代码417期望失败表示请求不应继续。

## 101 Switching Protocols

服务器已经理解了客户端的请求，并将通过 `Upgrade` 消息头通知客户端采用不同的协议来完成这个请求。在发送完这个响应最后的空行后，服务器将会切换到在 `Upgrade` 消息头中定义的那些协议。

## 102 Processing

由WebDAV（RFC 2518）扩展的状态码，代表处理将被继续执行。

# 2xx成功

## 200 OK

请求已成功，请求所希望的响应头或数据体将随此响应返回。

## 201 Created

请求已经被实现，而且有一个新的资源已经依据请求的需要而创建，且其URI已经随Location头信息返回。

## 202 Accepted

服务器已接受请求，但尚未处理。

## 203 Non-Authoritative Information
## 204 No Content

服务器成功处理了请求，没有返回任何内容。

## 205 Reset Content

服务器成功处理了请求，但没有返回任何内容。与204响应不同，此响应要求请求者重置文档视图。

## 206 Partial Content

服务器已经成功处理了部分GET请求。类似于FlashGet或者迅雷这类的HTTP下载工具都是使用此类响应实现断点续传或者将一个大文档分解为多个下载段同时下载。
## 207 Multi-Status

# 3xx重定向

- 300 Multiple Choices
- 301 Moved Permanently
- 302 Move Temporarily
- 303 See Other
- 304 Not Modified
- 305 Use Proxy
- 306 Switch Proxy
- 307 Temporary Redirect

# 4xx客户端错误

- 400 Bad Request
- 401 Unauthorized
- 402 Payment Required
- 403 Forbidden
- 404 Not Found
- 405 Method Not Allowed
- 406 Not Acceptable
- 407 Proxy Authentication Required
- 408 Request Timeout
- 409 Conflict
- 410 Gone
- 411 Length Required
- 412 Precondition Failed
- 413 Request Entity Too Large
- 414 Request-URI Too Long
- 415 Unsupported Media Type
- 416 Requested Range Not Satisfiable
- 417 Expectation Failed
- 418 I'm a teapot
- 421 Misdirected Request
- 422 Unprocessable Entity
- 423 Locked
- 424 Failed Dependency
- 425 Too Early
- 426 Upgrade Required
- 449 Retry With
- 451 Unavailable For Legal Reasons

# 5xx服务器错误

- 500 Internal Server Error
- 501 Not Implemented
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout
- 505 HTTP Version Not Supported
- 506 Variant Also Negotiates
- 507 Insufficient Storage
- 509 Bandwidth Limit Exceeded
- 510 Not Extended

# 参考

- https://baike.baidu.com/item/HTTP%E7%8A%B6%E6%80%81%E7%A0%81
- https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81
