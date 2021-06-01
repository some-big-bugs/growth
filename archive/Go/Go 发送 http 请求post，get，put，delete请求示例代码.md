<h1> Go 发送 http 请求: post，get，put，delete请求示例代码 </h1>

---

**Table of Contents**

- [HTTP 请求方法](#http-请求方法)
- [请求示例](#请求示例)
	- [GET](#get)
	- [POST](#post)
		- [kv 形式](#kv-形式)
		- [json](#json)
	- [PUT](#put)
	- [PATCH](#patch)
	- [DELETE](#delete)
	- [处理响应](#处理响应)
- [源码参考](#源码参考)
- [完成测试代码](#完成测试代码)
---

# HTTP 请求方法

根据 HTTP 标准，HTTP 请求可以使用多种请求方法。

在日常开发中大多数会用到 5 种请求方法： GET、POST、PUT、PATCH 和 DELETE

方法 | 描述
---|:---
GET | 请求指定的页面信息，并返回实体主体。
POST | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。
PUT | 从客户端向服务器传送的数据取代指定的文档的内容。
DELETE | 请求服务器删除指定的页面。
PATCH |	是对 PUT 方法的补充，用来对已知资源进行局部更新。

# 请求示例

## GET

**http.Get 直接访问**

```go
import (
	"net/http"
)

response, err := http.Get("https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1")

```

这种方法适合不需要 header 的方式

**自定义参数访问**

```go
import (
	"net/http"
)

url := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

req, _ := http.NewRequest("GET", url, nil)

req.Header.Add("Authorization", "xxxx")

response, err := http.DefaultClient.Do(req)
```

这种方法适合需要自定义一些 header 的场景

在查看 `http.Get` 方法源码时, 可以看出其是一个简便使用方式

```go
func (c *Client) Get(url string) (resp *Response, err error) {
	req, err := NewRequest("GET", url, nil)
	if err != nil {
		return nil, err
	}
	return c.Do(req)
}
```

## POST

在 POST 方式一般常用的为 2 中， 
1. 通过 kv 形式传送，例如 `form-data` 和 `x-www-form-urlencoded`
2. 通过 json 形式传送，例如 `application/json`

### kv 形式

```go
import (
	"net/http"
    "strings"
)

url := "https://blog.csdn.net/zyndev"

payload := strings.NewReader("a=111")

response, err := http.Post(url, "application/x-www-form-urlencoded", payload)
```

除了通过 http.Post 还可以使用 http.PostForm

```go

import (
	"net/http"
	"net/url"
)

targetUrl := "https://blog.csdn.net/zyndev"

payload := url.Values{"key":{"value"}, "id": {"123"}}

response, err := http.PostForm(targetUrl, payload)
```

### json

```go
targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

payload := strings.NewReader("{\"name\":\"张瑀楠\"}")

req, _ := http.NewRequest("POST", targetUrl, payload)

req.Header.Add("Content-Type", "application/json")

response, err := http.DefaultClient.Do(req)

```


## PUT

由于 `net/http` 没有提供简化的 `PUT` 请求，这里需要使用 `http.NewRequest` 来创建请求

```go
targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

payload := strings.NewReader("{\"name\":\"张瑀楠\"}")

req, _ := http.NewRequest("PUT", targetUrl, payload)

req.Header.Add("Content-Type", "application/json")

response, err := http.DefaultClient.Do(req)
```

## PATCH

由于 `net/http` 没有提供简化的 `PATCH` 请求，这里需要使用 `http.NewRequest` 来创建请求

```go
targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

payload := strings.NewReader("{\"name\":\"张瑀楠\"}")

req, _ := http.NewRequest("PATCH", targetUrl, payload)

req.Header.Add("Content-Type", "application/json")

response, err := http.DefaultClient.Do(req)
```

## DELETE

由于 `net/http` 没有提供简化的 `DELETE` 请求，这里需要使用 `http.NewRequest` 来创建请求

```go
targetUrl := "https://ddbc5ffb-c596-4f78-a99d-a6ea93bdc14f.mock.pstmn.io/user/1"

req, _ := http.NewRequest("DELETE", targetUrl, nil)

req.Header.Add("Authorization", "xxxx")

response, err := http.DefaultClient.Do(req)
```

## 处理响应

从上面的请示示例中看出，不管使用何种方式请求，最后都会得到 `response, err`, 也就是不管发起请求的方式是什么，处理的逻辑都是一样。

```go

... // 一堆请求方法构建方式

response, err := http.DefaultClient.Do(req)

if err != nil {
	// 错误逻辑处理
}

defer response.Body.Close() // 这步是必要的，防止以后的内存泄漏，切记


fmt.Println(response.StatusCode)  	// 获取状态码 
fmt.Println(response.Status)		// 获取状态码对应的文案
fmt.Println(response.Header)		// 获取响应头
body, _ := ioutil.ReadAll(response.Body) // 读取响应 body, 返回为 []byte
fmt.Println(string(body))			// 转成字符串看一下结果

```


# 源码参考


# 完成测试代码

```go
package http_demo

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
	"testing"
)

func TestHttpGet(t *testing.T) {
	response, err := http.Get("https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1")
	if err != nil {
		t.Error(err)
		panic(err)
	}
	defer response.Body.Close()
	t.Log(response)
	fmt.Println(response.StatusCode)
	fmt.Println(response.Status)
	fmt.Println(response.Header)
	body, _ := ioutil.ReadAll(response.Body)
	fmt.Println(string(body))
}

func TestHttpGetHeader(t *testing.T) {
	targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

	req, _ := http.NewRequest("GET", targetUrl, nil)

	req.Header.Add("Authorization", "xxxx")

	response, err := http.DefaultClient.Do(req)
	if err != nil {
		t.Error(err)
		panic(err)
	}
	defer response.Body.Close()
	t.Log(response)
}

func TestHttpPost(t *testing.T) {
	targetUrl := "https://blog.csdn.net/zyndev"

	payload := strings.NewReader("a=111")

	response, err := http.Post(targetUrl, "x-www-form-urlencoded", payload)

	if err != nil {
		t.Error(err)
		panic(err)
	}
	defer response.Body.Close()
	t.Log(response)
}

func TestHttpPostForm(t *testing.T) {
	targetUrl := "https://blog.csdn.net/zyndev"

	payload := url.Values{"key": {"value"}, "id": {"123"}}

	response, err := http.PostForm(targetUrl, payload)

	if err != nil {
		t.Error(err)
		panic(err)
	}
	defer response.Body.Close()
	t.Log(response)
}

func TestHttpPostJSON(t *testing.T) {
	targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

	payload := strings.NewReader("{\"name\":\"张瑀楠\"}")

	req, _ := http.NewRequest("POST", targetUrl, payload)

	req.Header.Add("Content-Type", "application/json")

	response, err := http.DefaultClient.Do(req)

	if err != nil {
		t.Error(err)
		panic(err)
	}

	defer response.Body.Close()
	t.Log(response)

}

func TestHttpPut(t *testing.T) {

	targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

	payload := strings.NewReader("{\"name\":\"张瑀楠\"}")

	req, _ := http.NewRequest("PUT", targetUrl, payload)

	req.Header.Add("Content-Type", "application/json")

	response, err := http.DefaultClient.Do(req)

	if err != nil {
		t.Error(err)
		panic(err)
	}

	defer response.Body.Close()
	t.Log(response)
}

func TestHttpPatch(t *testing.T) {
	targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"

	payload := strings.NewReader("{\"name\":\"张瑀楠\"}")

	req, _ := http.NewRequest("PATCH", targetUrl, payload)

	req.Header.Add("Content-Type", "application/json")

	response, err := http.DefaultClient.Do(req)

	if err != nil {
		t.Error(err)
		panic(err)
	}

	defer response.Body.Close()
	t.Log(response)
}

func TestHttpDelete(t *testing.T) {

	targetUrl := "https://ddbc5ffb-c596-4f78-a99d-a6ea93bdc14f.mock.pstmn.io/user/1"

	req, _ := http.NewRequest("DELETE", targetUrl, nil)

	req.Header.Add("Authorization", "xxxx")

	response, err := http.DefaultClient.Do(req)

	if err != nil {
		t.Error(err)
		panic(err)
	}

	defer response.Body.Close()
	t.Log(response)
}
```