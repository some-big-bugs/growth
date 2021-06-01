<h1> Go 结合 Consul 实现动态反向代理 </h1>

代理的核心功能可以用一句话概括：接受客户端的请求，转发到后端服务器，获得应答之后返回给客户端。

---

**Table of Contents**

- [反向代理](#反向代理)
- [实现逻辑](#实现逻辑)
- [Go 语言实现](#go-语言实现)
	- [原生代码](#原生代码)
	- [httputil.ReverseProxy 工具实现](#httputilreverseproxy-工具实现)
- [接入 consul 实现动态代理](#接入-consul-实现动态代理)
- [参考](#参考)

---

# 反向代理

反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

# 实现逻辑

根据代理的描述一共分成几个步骤：

1. 代理接收到客户端的请求，复制了原来的请求对象
2. 根据一些规则，修改新请求的请求指向
3. 把新请求发送到根据服务器端，并接收到服务器端返回的响应
4. 将上一步的响应根据需求处理一下，然后返回给客户端

# Go 语言实现

## 原生代码

因为要接收并转发 http 请求，所以要实现 `http.Handler`

```go
type OriginReverseProxy struct {
	servers []*url.URL
}

func NewOriginReverseProxy(targets []*url.URL) *OriginReverseProxy {
	return &OriginReverseProxy{
		servers: targets,
	}
}

// 实现 http.Handler, 用于接收所有的请求
func (proxy *OriginReverseProxy) ServeHTTP(rw http.ResponseWriter, req *http.Request) {
	// 1. 复制了原来的请求对象
	r2 := clone(req)

	// 2. 修改请求的地址，换为对应的服务器地址
	target := proxy.servers[rand.Int()%len(proxy.servers)]
	r2.URL.Scheme = target.Scheme
	r2.URL.Host = target.Host

	// 3. 发送复制的新请求
	transport := http.DefaultTransport

	res, err := transport.RoundTrip(r2)
	
	// 4。处理响应
	if err != nil {
		rw.WriteHeader(http.StatusBadGateway)
		return
	}

	for key, value := range res.Header {
		for _, v := range value {
			rw.Header().Add(key, v)
		}
	}

	rw.WriteHeader(res.StatusCode)
	io.Copy(rw, res.Body)
	res.Body.Close()
}

// 复制原请求，生成新的请求
func clone(req *http.Request) *http.Request {
	r2 := new(http.Request)
	*r2 = *req
	r2.URL = cloneURL(req.URL)
	if req.Header != nil {
		r2.Header = req.Header.Clone()
	}
	if req.Trailer != nil {
		r2.Trailer = req.Trailer.Clone()
	}
	if s := req.TransferEncoding; s != nil {
		s2 := make([]string, len(s))
		copy(s2, s)
		r2.TransferEncoding = s2
	}
	r2.Form = cloneURLValues(req.Form)
	r2.PostForm = cloneURLValues(req.PostForm)
	return r2
}

func cloneURLValues(v url.Values) url.Values {
	if v == nil {
		return nil
	}
	return url.Values(http.Header(v).Clone())
}

func cloneURL(u *url.URL) *url.URL {
	if u == nil {
		return nil
	}
	u2 := new(url.URL)
	*u2 = *u
	if u.User != nil {
		u2.User = new(url.Userinfo)
		*u2.User = *u.User
	}
	return u2
}
```

**测试**

```go

// 先用 gin 起一个 web 项目，方便转发
func TestGin(t *testing.T)  {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run(":9091") // listen and serve on 0.0.0.0:9091
}

func main() {
	proxy := proxy.NewOriginReverseProxy([]*url.URL{
		{
			Scheme: "http",
			Host:   "localhost:9091",
		},
	})
	http.ListenAndServe(":19090", proxy)
}

```

请求 `http://127.0.0.1:19090/ping` 返回 `{"message":"pong"}`

## httputil.ReverseProxy 工具实现

在上面的例子中，我们自己实现了请求的接收、复制、转发和处理。自己写的代码还算凑合吧。

从上面的示例中，其实我们主要关心的是**第二步: 修改请求的地址，换为对应的服务器地址**, 其余的逻辑都是通用的，还好官方已经帮我们处理了重复逻辑，那我们看看官方是怎么实现的

在 `httputil.ReverseProxy` 源码中，可以看出，通过自定义 `Director` 方法就可以在原请求复制后，新请求转发出之前对复制出的新请求进行修改，这里就是我们真正改动的地方，当然如果有其他定制需求，可以通过自定义 `ModifyResponse` 实现对响应的修改，自定义 `ErrorHandler` 来处理异常

```go
type ReverseProxy struct {
	// Director must be a function which modifies
	// the request into a new request to be sent
	// using Transport. Its response is then copied
	// back to the original client unmodified.
	// Director must not access the provided Request
	// after returning.
	Director func(*http.Request)

	Transport http.RoundTripper

	FlushInterval time.Duration

	ErrorLog *log.Logger

	BufferPool BufferPool

	ModifyResponse func(*http.Response) error

	ErrorHandler func(http.ResponseWriter, *http.Request, error)
}
```

这里我们通过自定义 `Director` 来修改请求地址

```go

func NewMultipleHostsReverseProxy(targets []*url.URL) *httputil.ReverseProxy {
	director := func(req *http.Request) {
		target := targets[rand.Int()%len(targets)]
		req.URL.Scheme = target.Scheme
		req.URL.Host = target.Host
		req.URL.Path = target.Path
	}
	return &httputil.ReverseProxy{Director: director}
}


```

**测试**

gin 的项目还是要启动的，这里不在赘叙

```go
func TestMultipleHostsReverseProxy(t *testing.T) {
	proxy := proxy.NewMultipleHostsReverseProxy([]*url.URL{
		{
			Scheme: "http",
			Host:   "localhost:9091",
		},
	})
	http.ListenAndServe(":9090", proxy)
}
```

# 接入 consul 实现动态代理

在前面的一篇文章中讲了如何用 [Go 实现 Consul 的服务发现](https://www.jianshu.com/p/6d4461c91c10), 如果要结合 consul 实现动态代理，需要考虑如何将请求的地址和对应的服务对应上。这里需要在原理的基础上加上一下功能：

1. 根据请求地址找到对应的服务
2. 根据服务找到对应的示例

针对第一步先实现最简单的，就以 `请求地址开头` 为规则

```go
type LoadBalanceRoute interface {
	ObtainInstance(path string) *url.URL
}

type Route struct {
	Path string
	ServiceName string
}

type DiscoveryLoadBalanceRoute struct {

	DiscoveryClient DiscoveryClient

	Routes []Route

}

func (d DiscoveryLoadBalanceRoute) ObtainInstance(path string) *url.URL {
	for _, route := range d.Routes {
		if strings.Index(path, route.Path) == 0 {
			instances, _ := d.DiscoveryClient.GetInstances(route.ServiceName)
			instance := instances[rand.Int()%len(instances)]
			scheme := "http"
			return &url.URL{
				Scheme: scheme,
				Host: instance.GetHost(),
			}
		}
	}
	return nil
}

func NewLoadBalanceReverseProxy(lb LoadBalanceRoute) *httputil.ReverseProxy {
	director := func(req *http.Request) {
		target := lb.ObtainInstance(req.URL.Path)
		req.URL.Scheme = target.Scheme
		req.URL.Host = target.Host
	}
	return &httputil.ReverseProxy{Director: director}
}
```

**测试**

```go
func main() {
	registry, _ := proxy.NewConsulServiceRegistry("127.0.0.1", 8500, "")
	reverseProxy := proxy.NewLoadBalanceReverseProxy(&proxy.DiscoveryLoadBalanceRoute{
		DiscoveryClient: registry,
		Routes: []proxy.Route{
			{
				Path: "abc",
				ServiceName: "abc",
			},
		},
	})
	http.ListenAndServe(":19090", reverseProxy)
}
```


# 参考

