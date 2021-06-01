<h1> Python | Flask 解决跨域问题 </h1>

# 系列文章目录

---

**Table of Contents**

- [系列文章目录](#系列文章目录)
- [前言](#前言)
- [使用步骤](#使用步骤)
  - [1. 引入库](#1-引入库)
  - [2. 配置](#2-配置)
    - [1. 使用 `CORS函数` 配置全局路由](#1-使用-cors函数-配置全局路由)
    - [2. 使用 `@cross_origin` 来配置单行路由](#2-使用-cross_origin-来配置单行路由)
  - [配置参数说明](#配置参数说明)
- [总结](#总结)
- [参考](#参考)

---

# 前言
我靠，又跨域了

# 使用步骤
## 1. 引入库
```
pip install flask-cors
```

## 2. 配置
flask-cors 有两种用法，一种为全局使用，一种对指定的路由使用

### 1. 使用 `CORS函数` 配置全局路由

```python
from flask import Flask, request
from flask_cors import CORS

app = Flask(__name__)
CORS(app, supports_credentials=True)
```

其中 `CORS` 提供了一些参数帮助我们定制一下操作。

常用的我们可以配置 `origins`、`methods`、`allow_headers`、`supports_credentials`


**所有的配置项如下：**
```

:param resources:
    The series of regular expression and (optionally) associated CORS
    options to be applied to the given resource path.

    If the argument is a dictionary, it's keys must be regular expressions,
    and the values must be a dictionary of kwargs, identical to the kwargs
    of this function.

    If the argument is a list, it is expected to be a list of regular
    expressions, for which the app-wide configured options are applied.

    If the argument is a string, it is expected to be a regular expression
    for which the app-wide configured options are applied.

    Default : Match all and apply app-level configuration

:type resources: dict, iterable or string

:param origins:
    The origin, or list of origins to allow requests from.
    The origin(s) may be regular expressions, case-sensitive strings,
    or else an asterisk

    Default : '*'
:type origins: list, string or regex

:param methods:
    The method or list of methods which the allowed origins are allowed to
    access for non-simple requests.

    Default : [GET, HEAD, POST, OPTIONS, PUT, PATCH, DELETE]
:type methods: list or string

:param expose_headers:
    The header or list which are safe to expose to the API of a CORS API
    specification.

    Default : None
:type expose_headers: list or string

:param allow_headers:
    The header or list of header field names which can be used when this
    resource is accessed by allowed origins. The header(s) may be regular
    expressions, case-sensitive strings, or else an asterisk.

    Default : '*', allow all headers
:type allow_headers: list, string or regex

:param supports_credentials:
    Allows users to make authenticated requests. If true, injects the
    `Access-Control-Allow-Credentials` header in responses. This allows
    cookies and credentials to be submitted across domains.

    :note: This option cannot be used in conjuction with a '*' origin

    Default : False
:type supports_credentials: bool

:param max_age:
    The maximum time for which this CORS request maybe cached. This value
    is set as the `Access-Control-Max-Age` header.

    Default : None
:type max_age: timedelta, integer, string or None

:param send_wildcard: If True, and the origins parameter is `*`, a wildcard
    `Access-Control-Allow-Origin` header is sent, rather than the
    request's `Origin` header.

    Default : False
:type send_wildcard: bool

:param vary_header:
    If True, the header Vary: Origin will be returned as per the W3
    implementation guidelines.

    Setting this header when the `Access-Control-Allow-Origin` is
    dynamically generated (e.g. when there is more than one allowed
    origin, and an Origin than '*' is returned) informs CDNs and other
    caches that the CORS headers are dynamic, and cannot be cached.

    If False, the Vary header will never be injected or altered.

    Default : True
:type vary_header: bool
```


### 2. 使用 `@cross_origin` 来配置单行路由
```python
from flask import Flask, request
from flask_cors import cross_origin

app = Flask(__name__)


@app.route('/')
@cross_origin(supports_credentials=True)
def hello():
    name = request.args.get("name", "World")
    return f'Hello, {name}!'

```

其中 `cross_origin` 和 `CORS` 提供一些基本相同的参数。

常用的我们可以配置 `origins`、`methods`、`allow_headers`、`supports_credentials`


**所有的配置项如下：**
```
:param origins:
    The origin, or list of origins to allow requests from.
    The origin(s) may be regular expressions, case-sensitive strings,
    or else an asterisk

    Default : '*'
:type origins: list, string or regex

:param methods:
    The method or list of methods which the allowed origins are allowed to
    access for non-simple requests.

    Default : [GET, HEAD, POST, OPTIONS, PUT, PATCH, DELETE]
:type methods: list or string

:param expose_headers:
    The header or list which are safe to expose to the API of a CORS API
    specification.

    Default : None
:type expose_headers: list or string

:param allow_headers:
    The header or list of header field names which can be used when this
    resource is accessed by allowed origins. The header(s) may be regular
    expressions, case-sensitive strings, or else an asterisk.

    Default : '*', allow all headers
:type allow_headers: list, string or regex

:param supports_credentials:
    Allows users to make authenticated requests. If true, injects the
    `Access-Control-Allow-Credentials` header in responses. This allows
    cookies and credentials to be submitted across domains.

    :note: This option cannot be used in conjuction with a '*' origin

    Default : False
:type supports_credentials: bool

:param max_age:
    The maximum time for which this CORS request maybe cached. This value
    is set as the `Access-Control-Max-Age` header.

    Default : None
:type max_age: timedelta, integer, string or None

:param send_wildcard: If True, and the origins parameter is `*`, a wildcard
    `Access-Control-Allow-Origin` header is sent, rather than the
    request's `Origin` header.

    Default : False
:type send_wildcard: bool

:param vary_header:
    If True, the header Vary: Origin will be returned as per the W3
    implementation guidelines.

    Setting this header when the `Access-Control-Allow-Origin` is
    dynamically generated (e.g. when there is more than one allowed
    origin, and an Origin than '*' is returned) informs CDNs and other
    caches that the CORS headers are dynamic, and cannot be cached.

    If False, the Vary header will never be injected or altered.

    Default : True
:type vary_header: bool

:param automatic_options:
    Only applies to the `cross_origin` decorator. If True, Flask-CORS will
    override Flask's default OPTIONS handling to return CORS headers for
    OPTIONS requests.

    Default : True
:type automatic_options: bool

```

## 配置参数说明

参数 |	类型 |	Head	| 默认 |说明
:---|:---|:---|:---|:---
resources | 字典、迭代器或字符串 |	无	| 全部 | 配置允许跨域的路由接口
origins	| 列表、字符串或正则表达式 |	Access-Control-Allow-Origin |	 * |配置允许跨域访问的源
methods	| 列表、字符串	| Access-Control-Allow-Methods	| [GET, HEAD, POST, OPTIONS, PUT, PATCH, DELETE] | 配置跨域支持的请求方式
expose_headers | 列表、字符串 |	Access-Control-Expose-Headers | None |	自定义请求响应的Head信息
allow_headers	| 列表、字符串或正则表达式	| Access-Control-Request-Headers | * |	配置允许跨域的请求头
supports_credentials |	布尔值 |	Access-Control-Allow-Credentials | False |	是否允许请求发送cookie
max_age | timedelta、整数、字符串 |	Access-Control-Max-Age |  None |	预检请求的有效时长

# 总结
在 flask 的跨域配置中，我们可以使用 `flask-cors` 来进行配置，其中 `CORS 函数` 用来做全局的配置, `@cross_origin` 来实现特定路由的配置

# 参考

- https://flask-cors.readthedocs.io/en/latest/
