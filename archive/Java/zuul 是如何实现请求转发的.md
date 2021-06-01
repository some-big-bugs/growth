<h1> zuul 1.x 是如何实现请求转发的 </h1>

> 文档写的再好，也不如源码写的好  
> 源码地址：  
> GitHub: https://github.com/Netflix/zuul  
> Gitee: https://gitee.com/github_mirror_plus/zuul

---

**Table of Contents**

- [简介](#简介)
- [实现逻辑](#实现逻辑)
- [源码](#源码)
  - [基于 Servlet 的请求转发](#基于-servlet-的请求转发)
  - [ZuulServlet 核心代码](#zuulservlet-核心代码)
  - [ZuulRunner 核心代码](#zuulrunner-核心代码)
  - [RequestContext 核心代码](#requestcontext-核心代码)
  - [FilterProcessor 核心代码](#filterprocessor-核心代码)
  - [在官方示例中，提供了两个简单的 Route 的 ZuulFilter 实现](#在官方示例中提供了两个简单的-route-的-zuulfilter-实现)
- [总结](#总结)
- [参考](#参考)

---

# 简介

官方简介，其实你要看这篇，说明你知道 zuul

Zuul is an edge service that provides dynamic routing, monitoring, resiliency, security, and more. Please view the wiki for usage, information, HOWTO, etc https://github.com/Netflix/zuul/wiki

Here are some links to help you learn more about the Zuul Project. Feel free to PR to add any other info, presentations, etc.

# 实现逻辑

上一篇文章 [Go | Go 结合 Consul 实现动态反向代理](./articles/Go/Go%20结合%20Consul%20实现动态反向代理.md) 里面简单的实现了一个反向代理，并简述了一下步骤，这里复述一下

根据代理的描述一共分成几个步骤：

1. 代理接收到客户端的请求，复制了原来的请求对象
2. 根据一些规则，修改新请求的请求指向
3. 把新请求发送到根据服务器端，并接收到服务器端返回的响应
4. 将上一步的响应根据需求处理一下，然后返回给客户端

# 源码

> 注意：这里的源码指的是 1.x 分支的代码

## 基于 Servlet 的请求转发

在一开始学习 Java Web 时，Servlet 是一个绕不过去的坎，zuul 也是基于 Servlet 实现的，在源码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
  
    <listener>
        <listener-class>com.netflix.zuul.StartServer</listener-class>
    </listener>

    <servlet>
        <servlet-name>ZuulServlet</servlet-name>
        <servlet-class>com.netflix.zuul.http.ZuulServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>ZuulServlet</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>ContextLifecycleFilter</filter-name>
        <filter-class>com.netflix.zuul.context.ContextLifecycleFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>ContextLifecycleFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
  
</web-app>
```

在这里需要重点关注下 `com.netflix.zuul.http.ZuulServlet` 和 `com.netflix.zuul.context.ContextLifecycleFilter`

## ZuulServlet 核心代码

代码在 `com.netflix.zuul.http.ZuulServlet`

下面的代码中省略了一部分，在这个过程中主要做了以下几件事

1. 将 将原始的 Request，Response 保存在 ThreadLocal 中，方便以后处理。因为 Tomcat 等 Servlet 容器默认使用了一个请求一个线程处理的方式，所以存在 ThreadLocal 即可在以后的处理流程中方便处理
2. 执行前置过滤器 `preRoute()`
3. 执行转发中过滤器 `route()`
4. 执行后置过滤器 `postRoute()`

其中转发的关键就在 `route()` 方法

```java
public class ZuulServlet extends HttpServlet {

    private ZuulRunner zuulRunner;

    @Override
    public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse) throws ServletException, IOException {
        try {
            // 这一步是将原始的 Request，Response 保存在 ThreadLocal 中
            init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);

            RequestContext context = RequestContext.getCurrentContext();
            context.setZuulEngineRan();

            try {
                // 前置处理
                preRoute();
            } catch (ZuulException e) {
                error(e);
                postRoute();
                return;
            }
            try {
                // 转发中处理
                route();
            } catch (ZuulException e) {
                error(e);
                postRoute();
                return;
            }
            try {
                // 后置处理
                postRoute();
            } catch (ZuulException e) {
                // 异常处理
                error(e);
                return;
            }

        } catch (Throwable e) {
            error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
        } finally {
            RequestContext.getCurrentContext().unset();
        }
    }

    void postRoute() throws ZuulException {
        zuulRunner.postRoute();
    }

    void route() throws ZuulException {
        zuulRunner.route();
    }

    void preRoute() throws ZuulException {
        zuulRunner.preRoute();
    }

    void init(HttpServletRequest servletRequest, HttpServletResponse servletResponse) {
        zuulRunner.init(servletRequest, servletResponse);
    }

    void error(ZuulException e) {
        RequestContext.getCurrentContext().setThrowable(e);
        zuulRunner.error();
    }
}

```

## ZuulRunner 核心代码

从上面的代码可以看出转发的关键在于 `ZuulServlet#route()`， 而  `ZuulServlet#route()` 在于 `zuulRunner.route()`

ZuulRunner 主要功能

1. `init` 将 Request 和 Response 保存到 `RequestContext.getCurrentContext()`, 这里面就是上面提到的 `ThreadLocal` 的处理类
2. 调用下 `FilterProcessor.getInstance().route()`

```java
public class ZuulRunner {

    private boolean bufferRequests;

    public ZuulRunner() {
        this.bufferRequests = true;
    }

    public ZuulRunner(boolean bufferRequests) {
        this.bufferRequests = bufferRequests;
    }

    public void init(HttpServletRequest servletRequest, HttpServletResponse servletResponse) {

        RequestContext ctx = RequestContext.getCurrentContext();
        if (bufferRequests) {
            ctx.setRequest(new HttpServletRequestWrapper(servletRequest));
        } else {
            ctx.setRequest(servletRequest);
        }

        ctx.setResponse(new HttpServletResponseWrapper(servletResponse));
    }

    public void route() throws ZuulException {
        FilterProcessor.getInstance().route();
    }
}
```

## RequestContext 核心代码

主要是 ThreadLocal 和 copy 

```java
public class RequestContext extends ConcurrentHashMap<String, Object> {

    private static final Logger LOG = LoggerFactory.getLogger(RequestContext.class);

    protected static Class<? extends RequestContext> contextClass = RequestContext.class;

    private static RequestContext testContext = null;

    protected static final ThreadLocal<? extends RequestContext> threadLocal = new ThreadLocal<RequestContext>() {
        @Override
        protected RequestContext initialValue() {
            try {
                return contextClass.newInstance();
            } catch (Throwable e) {
                throw new RuntimeException(e);
            }
        }
    };

    /**
     * sets the "responseBody" value as a String. This is the response sent back to the client.
     *
     * @param body
     */
    public void setResponseBody(String body) {
        set("responseBody", body);
    }

    /**
     * Use this instead of response.setStatusCode()
     *
     * @param nStatusCode
     */
    public void setResponseStatusCode(int nStatusCode) {
        getResponse().setStatus(nStatusCode);
        set("responseStatusCode", nStatusCode);
    }

    /**
     * Mkaes a copy of the RequestContext. This is used for debugging.
     *
     * @return
     */
    public RequestContext copy() {
        RequestContext copy = new RequestContext();
        // 这里省略了一部分代码，意思就是把原来的 request 深度复制一份
        return copy;
    }
}
```

## FilterProcessor 核心代码


主要逻辑就是找到对应 type 的 `List<ZuulFilter>` 并执行 `runFilter()`

```java
public class FilterProcessor {

    static FilterProcessor INSTANCE = new FilterProcessor();

    /**
     * @return the singleton FilterProcessor
     */
    public static FilterProcessor getInstance() {
        return INSTANCE;
    }

    /**
     * Runs all "route" filters. These filters route calls to an origin.
     *
     * @throws ZuulException if an exception occurs.
     */
    public void route() throws ZuulException {
        try {
            runFilters("route");
        } catch (ZuulException e) {
            throw e;
        } catch (Throwable e) {
            throw new ZuulException(e, 500, "UNCAUGHT_EXCEPTION_IN_ROUTE_FILTER_" + e.getClass().getName());
        }
    }

    /**
     * runs all filters of the filterType sType/ Use this method within filters to run custom filters by type
     *
     * @param sType the filterType.
     * @return
     * @throws Throwable throws up an arbitrary exception
     */
    public Object runFilters(String sType) throws Throwable {
        if (RequestContext.getCurrentContext().debugRouting()) {
            Debug.addRoutingDebug("Invoking {" + sType + "} type filters");
        }
        boolean bResult = false;
        List<ZuulFilter> list = FilterLoader.getInstance().getFiltersByType(sType);
        if (list != null) {
            for (int i = 0; i < list.size(); i++) {
                ZuulFilter zuulFilter = list.get(i);
                Object result = processZuulFilter(zuulFilter);
                if (result != null && result instanceof Boolean) {
                    bResult |= ((Boolean) result);
                }
            }
        }
        return bResult;
    }

    /**
     * Processes an individual ZuulFilter. This method adds Debug information. Any uncaught Thowables are caught by this method and converted to a ZuulException with a 500 status code.
     *
     * @param filter
     * @return the return value for that filter
     * @throws ZuulException
     */
    public Object processZuulFilter(ZuulFilter filter) throws ZuulException {

        RequestContext ctx = RequestContext.getCurrentContext();
        boolean bDebug = ctx.debugRouting();
        final String metricPrefix = "zuul.filter-";
        long execTime = 0;
        String filterName = "";
        try {
            long ltime = System.currentTimeMillis();
            filterName = filter.getClass().getSimpleName();
            
            RequestContext copy = null;
            Object o = null;
            Throwable t = null;

            if (bDebug) {
                Debug.addRoutingDebug("Filter " + filter.filterType() + " " + filter.filterOrder() + " " + filterName);
                copy = ctx.copy();
            }
            
            ZuulFilterResult result = filter.runFilter();
            ExecutionStatus s = result.getStatus();
            execTime = System.currentTimeMillis() - ltime;

            switch (s) {
                case FAILED:
                    t = result.getException();
                    ctx.addFilterExecutionSummary(filterName, ExecutionStatus.FAILED.name(), execTime);
                    break;
                case SUCCESS:
                    o = result.getResult();
                    ctx.addFilterExecutionSummary(filterName, ExecutionStatus.SUCCESS.name(), execTime);
                    if (bDebug) {
                        Debug.addRoutingDebug("Filter {" + filterName + " TYPE:" + filter.filterType() + " ORDER:" + filter.filterOrder() + "} Execution time = " + execTime + "ms");
                        Debug.compareContextState(filterName, copy);
                    }
                    break;
                default:
                    break;
            }
            
            if (t != null) throw t;

            usageNotifier.notify(filter, s);
            return o;

        } catch (Throwable e) {
            if (bDebug) {
                Debug.addRoutingDebug("Running Filter failed " + filterName + " type:" + filter.filterType() + " order:" + filter.filterOrder() + " " + e.getMessage());
            }
            usageNotifier.notify(filter, ExecutionStatus.FAILED);
            if (e instanceof ZuulException) {
                throw (ZuulException) e;
            } else {
                ZuulException ex = new ZuulException(e, "Filter threw Exception", 500, filter.filterType() + ":" + filterName);
                ctx.addFilterExecutionSummary(filterName, ExecutionStatus.FAILED.name(), execTime);
                throw ex;
            }
        }
    }

    /**
     * Publishes a counter metric for each filter on each use.
     */
    public static class BasicFilterUsageNotifier implements FilterUsageNotifier {
        private static final String METRIC_PREFIX = "zuul.filter-";

        @Override
        public void notify(ZuulFilter filter, ExecutionStatus status) {
            DynamicCounter.increment(METRIC_PREFIX + filter.getClass().getSimpleName(), "status", status.name(), "filtertype", filter.filterType());
        }
    }
}
```



通过上面的代码中，可以看到得到简单的流程图



![zuul](/img/zuul_filter.png)



## 在官方示例中，提供了两个简单的 Route 的 ZuulFilter 实现



**SimpleHostRoutingFilter.groovy**



在这个示例中，在 Filter 实现中将请求复制并转发到目标服务，这个是简单的逻辑



```java
class SimpleHostRoutingFilter extends ZuulFilter {

  	// 声明这个过滤器是 route 类型
    @Override
    String filterType() {
        return 'route'
    }

  	// 过滤器的执行逻辑
    Object run() {
        HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
        Header[] headers = buildZuulRequestHeaders(request)
        String verb = getVerb(request);
        InputStream requestEntity = request.getInputStream();
        CloseableHttpClient httpclient = CLIENT.get()

        String uri = request.getRequestURI()
        if (RequestContext.getCurrentContext().requestURI != null) {
            uri = RequestContext.getCurrentContext().requestURI
        }

        try {
          	// 将请求转发到指定服务器
            HttpResponse response = forward(httpclient, verb, uri, request, headers, requestEntity)
            setResponse(response)
        } catch (Exception e) {
            throw e;
        }
        return null
    }

    HttpResponse forward(CloseableHttpClient httpclient, String verb, String uri, HttpServletRequest request, Header[] headers, InputStream requestEntity) {

        requestEntity = debug(verb, uri, request, headers, requestEntity)
        HttpHost httpHost = getHttpHost()
        HttpRequest httpRequest;

        switch (verb) {
            case 'POST':
                httpRequest = new HttpPost(uri + getQueryString())
                InputStreamEntity entity = new InputStreamEntity(requestEntity, request.getContentLength())
                httpRequest.setEntity(entity)
                break
            case 'PUT':
                httpRequest = new HttpPut(uri + getQueryString())
                InputStreamEntity entity = new InputStreamEntity(requestEntity, request.getContentLength())
                httpRequest.setEntity(entity)
                break;
            default:
                httpRequest = new BasicHttpRequest(verb, uri + getQueryString())
        }

        try {
            httpRequest.setHeaders(headers)
            return forwardRequest(httpclient, httpHost, httpRequest)
        } finally {
            //httpclient.close();
        }
    }

    HttpResponse forwardRequest(HttpClient httpclient, HttpHost httpHost, HttpRequest httpRequest) {
        return httpclient.execute(httpHost, httpRequest);
    }
}
```



**ZuulNFRequest 结合 Netflix 的 route 过滤器**



这个示例中，从 `HttpClient` 转发改为了使用 `RibbonCommand` 转发，从而使用了 Ribbon 的功能。关于 `Ribbon` 以后有时间再说



```java
class ZuulNFRequest extends ZuulFilter {

    @Override
    String filterType() {
        return 'route'
    }

    boolean shouldFilter() {
        return NFRequestContext.currentContext.getRouteHost() == null && RequestContext.currentContext.sendZuulResponse()
    }

    Object run() {
        NFRequestContext context = NFRequestContext.currentContext
        HttpServletRequest request = context.getRequest();

        MultivaluedMap<String, String> headers = buildZuulRequestHeaders(request)
        MultivaluedMap<String, String> params = buildZuulRequestQueryParams(request)
        Verb verb = getVerb(request);
        Object requestEntity = getRequestBody(request)
        IClient restClient = ClientFactory.getNamedClient(context.getRouteVIP());

        String uri = request.getRequestURI()
        if (context.requestURI != null) {
            uri = context.requestURI
        }
        //remove double slashes
        uri = uri.replace("//", "/")

        HttpResponse response = forward(restClient, verb, uri, headers, params, requestEntity)
        setResponse(response)
        return response
    }

    def HttpResponse forward(RestClient restClient, Verb verb, uri, MultivaluedMap<String, String> headers, MultivaluedMap<String, String> params, InputStream requestEntity) {
        debug(restClient, verb, uri, headers, params, requestEntity)

//        restClient.apacheHttpClient.params.setVirtualHost(headers.getFirst("host"))

        String route = NFRequestContext.getCurrentContext().route
        if (route == null) {
            String path = RequestContext.currentContext.requestURI
            if (path == null) {
                path = RequestContext.currentContext.getRequest() getRequestURI()
            }
            route = "route" //todo get better name
        }
        route = route.replace("/", "_")


        RibbonCommand<AbstractLoadBalancerAwareClient<HttpRequest, HttpResponse>> command = new RibbonCommand<>(restClient, verb, uri, headers, params, requestEntity);
        try {
            HttpResponse response = command.execute();
            return response
        } catch (HystrixRuntimeException e) {
            if (e?.fallbackException?.cause instanceof ClientException) {
                ClientException ex = e.fallbackException.cause as ClientException
                throw new ZuulException(ex, "Forwarding error", 500, ex.getErrorType().toString())
            }
            throw new ZuulException(e, "Forwarding error", 500, e.failureType.toString())
        }
    }
}
```





# 总结

从 zuul 实现中看，还是基于 Servlet 的，并在过程中加入 前、中、后和异常处理链。因为基于 Servlet 其处理流程是阻塞的，性能会有所下降。

在 zuul 里面采用了 java 和 groovy 混合编程的方式，编程更加灵活。通过自定了一个 GroovyCompiler 来加载指定路径的 groovy 文件来实现在运行中动态添加 ZuulFilter 这种动态机制在一定程度上实现了热更新 ZuulFilter 功能，也是值得学习的。

# 参考

GitHub: https://github.com/Netflix/zuul