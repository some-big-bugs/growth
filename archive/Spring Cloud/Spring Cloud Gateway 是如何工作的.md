# Spring Cloud Gateway 是如何工作的



> 文档写的再好，也不如源码写的好  
> 源码地址：  
> GitHub: https://github.com/spring-cloud/spring-cloud-gateway  
> Gitee: https://gitee.com/github_mirror_plus/spring-cloud-gateway

---

- [Spring Cloud Gateway 是如何工作的](#spring-cloud-gateway-是如何工作的)
- [负责转发请求的 NettyRoutingFilter](#负责转发请求的-nettyroutingfilter)
- [负责将响应回写到原连接的 NettyWriteResponseFilter](#负责将响应回写到原连接的-nettywriteresponsefilter)
- [如何实现负载均衡的](#如何实现负载均衡的)
- [总结](#总结)
- [参考](#参考)
- [扩展阅读](#扩展阅读)
- [鸣谢](#鸣谢)

---

在 Spring Cloud Gateway 流程图中，可以看出优先级低的 Filter 则在 Request 发送前最后处理，在 Response 响应后优先处理



![Spring Cloud Gateway 流程图](./../../img/scg.png)



# 负责转发请求的 NettyRoutingFilter



熟悉 Spring Cloud Gateway 用法的应该都知道 `GlobalFilter` 在 Spring Cloud Gateway 中，有一个有趣的 `GlobalFilter` 其优先级最低



> 其优先级根据 `getOrder()` 来判断，其实值越大则优先级越小，反之亦然



在其中 filter 方法做了以下几件事：



1. 判断请求是否已经发送过，如果发送过，则不在处理
2. 标记请求为发送过，并开始处理请求
3. 处理下请求头信息
4. 获取 Route 信息，里面设置了请求超时时间
5. 使用 HttpClient 发送请求
6. 发送请求并获取响应，将响应重新处理到 `exchange` ，最终得到 `Flux<HttpClientResponse> responseFlux`
7. 判断 `responseFlux` 是否超时，如果超时则进行超时处理
8. 结束



```java
public class NettyRoutingFilter implements GlobalFilter, Ordered {

	private static final Log log = LogFactory.getLog(NettyRoutingFilter.class);

	/**
	 * 设置最低的优先级，此处为了最后处理请求，第一处理响应
	 */
	public static final int ORDER = Ordered.LOWEST_PRECEDENCE;

    // httpclient 负责转发请求
	private final HttpClient httpClient;

	private final ObjectProvider<List<HttpHeadersFilter>> headersFiltersProvider;

	private final HttpClientProperties properties;

	// do not use this headersFilters directly, use getHeadersFilters() instead.
	private volatile List<HttpHeadersFilter> headersFilters;

	public NettyRoutingFilter(HttpClient httpClient, ObjectProvider<List<HttpHeadersFilter>> headersFiltersProvider,
			HttpClientProperties properties) {
		this.httpClient = httpClient;
		this.headersFiltersProvider = headersFiltersProvider;
		this.properties = properties;
	}

	public List<HttpHeadersFilter> getHeadersFilters() {
		if (headersFilters == null) {
			headersFilters = headersFiltersProvider.getIfAvailable();
		}
		return headersFilters;
	}

	@Override
	public int getOrder() {
		return ORDER;
	}

	@Override
	@SuppressWarnings("Duplicates")
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		URI requestUrl = exchange.getRequiredAttribute(GATEWAY_REQUEST_URL_ATTR);

		String scheme = requestUrl.getScheme();
		if (isAlreadyRouted(exchange) || (!"http".equals(scheme) && !"https".equals(scheme))) {
			return chain.filter(exchange);
		}
		setAlreadyRouted(exchange);

		ServerHttpRequest request = exchange.getRequest();

		final HttpMethod method = HttpMethod.valueOf(request.getMethodValue());
		final String url = requestUrl.toASCIIString();

		HttpHeaders filtered = filterRequest(getHeadersFilters(), exchange);

		final DefaultHttpHeaders httpHeaders = new DefaultHttpHeaders();
		filtered.forEach(httpHeaders::set);

		boolean preserveHost = exchange.getAttributeOrDefault(PRESERVE_HOST_HEADER_ATTRIBUTE, false);
		Route route = exchange.getAttribute(GATEWAY_ROUTE_ATTR);

		Flux<HttpClientResponse> responseFlux = getHttpClient(route, exchange).headers(headers -> {
			headers.add(httpHeaders);
			// Will either be set below, or later by Netty
			headers.remove(HttpHeaders.HOST);
			if (preserveHost) {
				String host = request.getHeaders().getFirst(HttpHeaders.HOST);
				headers.add(HttpHeaders.HOST, host);
			}
		}).request(method).uri(url).send((req, nettyOutbound) -> {
			if (log.isTraceEnabled()) {
				nettyOutbound.withConnection(connection -> log.trace("outbound route: "
						+ connection.channel().id().asShortText() + ", inbound: " + exchange.getLogPrefix()));
			}
			return nettyOutbound.send(request.getBody().map(this::getByteBuf));
		}).responseConnection((res, connection) -> {

			// Defer committing the response until all route filters have run
			// Put client response as ServerWebExchange attribute and write
			// response later NettyWriteResponseFilter
			exchange.getAttributes().put(CLIENT_RESPONSE_ATTR, res);
			exchange.getAttributes().put(CLIENT_RESPONSE_CONN_ATTR, connection);

			ServerHttpResponse response = exchange.getResponse();
			// put headers and status so filters can modify the response
			HttpHeaders headers = new HttpHeaders();

			res.responseHeaders().forEach(entry -> headers.add(entry.getKey(), entry.getValue()));

			String contentTypeValue = headers.getFirst(HttpHeaders.CONTENT_TYPE);
			if (StringUtils.hasLength(contentTypeValue)) {
				exchange.getAttributes().put(ORIGINAL_RESPONSE_CONTENT_TYPE_ATTR, contentTypeValue);
			}

			setResponseStatus(res, response);

			// make sure headers filters run after setting status so it is
			// available in response
			HttpHeaders filteredResponseHeaders = HttpHeadersFilter.filter(getHeadersFilters(), headers, exchange,
					Type.RESPONSE);

			if (!filteredResponseHeaders.containsKey(HttpHeaders.TRANSFER_ENCODING)
					&& filteredResponseHeaders.containsKey(HttpHeaders.CONTENT_LENGTH)) {
				// It is not valid to have both the transfer-encoding header and
				// the content-length header.
				// Remove the transfer-encoding header in the response if the
				// content-length header is present.
				response.getHeaders().remove(HttpHeaders.TRANSFER_ENCODING);
			}

			exchange.getAttributes().put(CLIENT_RESPONSE_HEADER_NAMES, filteredResponseHeaders.keySet());

			response.getHeaders().putAll(filteredResponseHeaders);

			return Mono.just(res);
		});

		Duration responseTimeout = getResponseTimeout(route);
		if (responseTimeout != null) {
			responseFlux = responseFlux
					.timeout(responseTimeout,
							Mono.error(new TimeoutException("Response took longer than timeout: " + responseTimeout)))
					.onErrorMap(TimeoutException.class,
							th -> new ResponseStatusException(HttpStatus.GATEWAY_TIMEOUT, th.getMessage(), th));
		}

		return responseFlux.then(chain.filter(exchange));
	}

	protected ByteBuf getByteBuf(DataBuffer dataBuffer) {
		if (dataBuffer instanceof NettyDataBuffer) {
			NettyDataBuffer buffer = (NettyDataBuffer) dataBuffer;
			return buffer.getNativeBuffer();
		}
		// MockServerHttpResponse creates these
		else if (dataBuffer instanceof DefaultDataBuffer) {
			DefaultDataBuffer buffer = (DefaultDataBuffer) dataBuffer;
			return Unpooled.wrappedBuffer(buffer.getNativeBuffer());
		}
		throw new IllegalArgumentException("Unable to handle DataBuffer of type " + dataBuffer.getClass());
	}

	private void setResponseStatus(HttpClientResponse clientResponse, ServerHttpResponse response) {
		HttpStatus status = HttpStatus.resolve(clientResponse.status().code());
		if (status != null) {
			response.setStatusCode(status);
		}
		else {
			while (response instanceof ServerHttpResponseDecorator) {
				response = ((ServerHttpResponseDecorator) response).getDelegate();
			}
			if (response instanceof AbstractServerHttpResponse) {
				((AbstractServerHttpResponse) response).setRawStatusCode(clientResponse.status().code());
			}
			else {
				// TODO: log warning here, not throw error?
				throw new IllegalStateException("Unable to set status code " + clientResponse.status().code()
						+ " on response of type " + response.getClass().getName());
			}
		}
	}

	/**
	 * Creates a new HttpClient with per route timeout configuration. Sub-classes that
	 * override, should call super.getHttpClient() if they want to honor the per route
	 * timeout configuration.
	 * @param route the current route.
	 * @param exchange the current ServerWebExchange.
	 * @return
	 */
	protected HttpClient getHttpClient(Route route, ServerWebExchange exchange) {
		Object connectTimeoutAttr = route.getMetadata().get(CONNECT_TIMEOUT_ATTR);
		if (connectTimeoutAttr != null) {
			Integer connectTimeout = getInteger(connectTimeoutAttr);
			return this.httpClient.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, connectTimeout);
		}
		return httpClient;
	}

	static Integer getInteger(Object connectTimeoutAttr) {
		Integer connectTimeout;
		if (connectTimeoutAttr instanceof Integer) {
			connectTimeout = (Integer) connectTimeoutAttr;
		}
		else {
			connectTimeout = Integer.parseInt(connectTimeoutAttr.toString());
		}
		return connectTimeout;
	}

	private Duration getResponseTimeout(Route route) {
		Object responseTimeoutAttr = route.getMetadata().get(RESPONSE_TIMEOUT_ATTR);
		Long responseTimeout = null;
		if (responseTimeoutAttr != null) {
			if (responseTimeoutAttr instanceof Number) {
				responseTimeout = ((Number) responseTimeoutAttr).longValue();
			}
			else {
				responseTimeout = Long.valueOf(responseTimeoutAttr.toString());
			}
		}
		return responseTimeout != null ? Duration.ofMillis(responseTimeout) : properties.getResponseTimeout();
	}

}
```



# 负责将响应回写到原连接的 NettyWriteResponseFilter



`NettyRoutingFilter` 是最后处理请求的，那么  `NettyWriteResponseFilter` 就应该是最后处理响应的，其 Order 为 -1



> 在自己配置 `GlobalFilter` 和 `GatewayFilter` 一定要记得这两个优先级
>
> 如果想在请求被转发到目标服务器之前处理 Request 其设置的优先级一定要比 `NettyRoutingFilter` 高，即 Order 比 **2147483647** 小
>
> 如果想在响应被重写会发起方前处理下 Response 其设置的优先级一定要比 `NettyWriteResponseFilter` 低，即 Order 比 **-1** 大



```java
public class NettyWriteResponseFilter implements GlobalFilter, Ordered {

	/**
	 * Order for write response filter.
	 */
	public static final int WRITE_RESPONSE_FILTER_ORDER = -1;

	private static final Log log = LogFactory.getLog(NettyWriteResponseFilter.class);

	private final List<MediaType> streamingMediaTypes;

	public NettyWriteResponseFilter(List<MediaType> streamingMediaTypes) {
		this.streamingMediaTypes = streamingMediaTypes;
	}

	@Override
	public int getOrder() {
		return WRITE_RESPONSE_FILTER_ORDER;
	}

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		// NOTICE: nothing in "pre" filter stage as CLIENT_RESPONSE_CONN_ATTR is not added
		// until the NettyRoutingFilter is run
		// @formatter:off
		return chain.filter(exchange)
				.doOnError(throwable -> cleanup(exchange))
				.then(Mono.defer(() -> {
					Connection connection = exchange.getAttribute(CLIENT_RESPONSE_CONN_ATTR);

					if (connection == null) {
						return Mono.empty();
					}
					if (log.isTraceEnabled()) {
						log.trace("NettyWriteResponseFilter start inbound: "
								+ connection.channel().id().asShortText() + ", outbound: "
								+ exchange.getLogPrefix());
					}
					ServerHttpResponse response = exchange.getResponse();

					// TODO: needed?
					final Flux<DataBuffer> body = connection
							.inbound()
							.receive()
							.retain()
							.map(byteBuf -> wrap(byteBuf, response));

					MediaType contentType = null;
					try {
						contentType = response.getHeaders().getContentType();
					}
					catch (Exception e) {
						if (log.isTraceEnabled()) {
							log.trace("invalid media type", e);
						}
					}
					return (isStreamingMediaType(contentType)
							? response.writeAndFlushWith(body.map(Flux::just))
							: response.writeWith(body));
				})).doOnCancel(() -> cleanup(exchange));
		// @formatter:on
	}

	protected DataBuffer wrap(ByteBuf byteBuf, ServerHttpResponse response) {
		DataBufferFactory bufferFactory = response.bufferFactory();
		if (bufferFactory instanceof NettyDataBufferFactory) {
			NettyDataBufferFactory factory = (NettyDataBufferFactory) bufferFactory;
			return factory.wrap(byteBuf);
		}
		// MockServerHttpResponse creates these
		else if (bufferFactory instanceof DefaultDataBufferFactory) {
			DataBuffer buffer = ((DefaultDataBufferFactory) bufferFactory).allocateBuffer(byteBuf.readableBytes());
			buffer.write(byteBuf.nioBuffer());
			byteBuf.release();
			return buffer;
		}
		throw new IllegalArgumentException("Unkown DataBufferFactory type " + bufferFactory.getClass());
	}

	private void cleanup(ServerWebExchange exchange) {
		Connection connection = exchange.getAttribute(CLIENT_RESPONSE_CONN_ATTR);
		if (connection != null && connection.channel().isActive() && !connection.isPersistent()) {
			connection.dispose();
		}
	}

	// TODO: use framework if possible
	// TODO: port to WebClientWriteResponseFilter
	private boolean isStreamingMediaType(@Nullable MediaType contentType) {
		return (contentType != null && this.streamingMediaTypes.stream().anyMatch(contentType::isCompatibleWith));
	}

}
```



# 如何实现负载均衡的

实现负载均衡的过滤器为 `ReactiveLoadBalancerClientFilter` 该过滤器的主要功能为

1. 处理转发地址为 lb 开头的配置，在 Spring Cloud Gateway 的 routes 配置中 lb 是需要进行负载均衡的
2. 根据 lb 信息找到对应的 serviceId，例如 `lb://user-server` 则 serviceId 为 `user-server`
3. 根据 serviceId 从 LoadBalancer 中找到对应的可用实例，lb 实现主要依赖 consul、eureka、zk 等
4. 如果无可用的实例，则判断是否使用 404 提示，如果用则用 404 状态码，如果否则用 502
5. 从获取到的可用的服务实例 serviceInstance 获取目标服务器的 host 信息
6. 将获取到的 host 信息设置到 Attributes 中, 方便在 `NettyRoutingFilter`进行请求转发时获取到这个地址，并转发到对应服务器 `exchange.getAttributes().put(GATEWAY_REQUEST_URL_ATTR, requestUrl)`



```java
public class ReactiveLoadBalancerClientFilter implements GlobalFilter, Ordered {

	private static final Log log = LogFactory.getLog(ReactiveLoadBalancerClientFilter.class);

	/**
	 * Order of filter.
	 */
	public static final int LOAD_BALANCER_CLIENT_FILTER_ORDER = 10150;

	private final LoadBalancerClientFactory clientFactory;

	private final GatewayLoadBalancerProperties properties;

	private final LoadBalancerProperties loadBalancerProperties;

	public ReactiveLoadBalancerClientFilter(LoadBalancerClientFactory clientFactory,
			GatewayLoadBalancerProperties properties, LoadBalancerProperties loadBalancerProperties) {
		this.clientFactory = clientFactory;
		this.properties = properties;
		this.loadBalancerProperties = loadBalancerProperties;
	}

	@Override
	public int getOrder() {
		return LOAD_BALANCER_CLIENT_FILTER_ORDER;
	}

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		URI url = exchange.getAttribute(GATEWAY_REQUEST_URL_ATTR);
		String schemePrefix = exchange.getAttribute(GATEWAY_SCHEME_PREFIX_ATTR);
		if (url == null || (!"lb".equals(url.getScheme()) && !"lb".equals(schemePrefix))) {
			return chain.filter(exchange);
		}
		// preserve the original url
		addOriginalRequestUrl(exchange, url);

		if (log.isTraceEnabled()) {
			log.trace(ReactiveLoadBalancerClientFilter.class.getSimpleName() + " url before: " + url);
		}

		URI requestUri = exchange.getAttribute(GATEWAY_REQUEST_URL_ATTR);
		String serviceId = requestUri.getHost();
		Set<LoadBalancerLifecycle> supportedLifecycleProcessors = LoadBalancerLifecycleValidator
				.getSupportedLifecycleProcessors(clientFactory.getInstances(serviceId, LoadBalancerLifecycle.class),
						RequestDataContext.class, ResponseData.class, ServiceInstance.class);
		DefaultRequest<RequestDataContext> lbRequest = new DefaultRequest<>(new RequestDataContext(
				new RequestData(exchange.getRequest()), getHint(serviceId, loadBalancerProperties.getHint())));
		return choose(lbRequest, serviceId, supportedLifecycleProcessors).doOnNext(response -> {

			if (!response.hasServer()) {
				supportedLifecycleProcessors.forEach(lifecycle -> lifecycle
						.onComplete(new CompletionContext<>(CompletionContext.Status.DISCARD, lbRequest, response)));
				throw NotFoundException.create(properties.isUse404(), "Unable to find instance for " + url.getHost());
			}

			ServiceInstance retrievedInstance = response.getServer();

			URI uri = exchange.getRequest().getURI();

			// if the `lb:<scheme>` mechanism was used, use `<scheme>` as the default,
			// if the loadbalancer doesn't provide one.
			String overrideScheme = retrievedInstance.isSecure() ? "https" : "http";
			if (schemePrefix != null) {
				overrideScheme = url.getScheme();
			}

			DelegatingServiceInstance serviceInstance = new DelegatingServiceInstance(retrievedInstance,
					overrideScheme);

			URI requestUrl = reconstructURI(serviceInstance, uri);

			if (log.isTraceEnabled()) {
				log.trace("LoadBalancerClientFilter url chosen: " + requestUrl);
			}
			exchange.getAttributes().put(GATEWAY_REQUEST_URL_ATTR, requestUrl);
			exchange.getAttributes().put(GATEWAY_LOADBALANCER_RESPONSE_ATTR, response);
			supportedLifecycleProcessors.forEach(lifecycle -> lifecycle.onStartRequest(lbRequest, response));
		}).then(chain.filter(exchange))
				.doOnError(throwable -> supportedLifecycleProcessors.forEach(lifecycle -> lifecycle
						.onComplete(new CompletionContext<ResponseData, ServiceInstance, RequestDataContext>(
								CompletionContext.Status.FAILED, throwable, lbRequest,
								exchange.getAttribute(GATEWAY_LOADBALANCER_RESPONSE_ATTR)))))
				.doOnSuccess(aVoid -> supportedLifecycleProcessors.forEach(lifecycle -> lifecycle
						.onComplete(new CompletionContext<ResponseData, ServiceInstance, RequestDataContext>(
								CompletionContext.Status.SUCCESS, lbRequest,
								exchange.getAttribute(GATEWAY_LOADBALANCER_RESPONSE_ATTR),
								new ResponseData(exchange.getResponse(), new RequestData(exchange.getRequest()))))));
	}

	protected URI reconstructURI(ServiceInstance serviceInstance, URI original) {
		return LoadBalancerUriTools.reconstructURI(serviceInstance, original);
	}

	private Mono<Response<ServiceInstance>> choose(Request<RequestDataContext> lbRequest, String serviceId,
			Set<LoadBalancerLifecycle> supportedLifecycleProcessors) {
		ReactorLoadBalancer<ServiceInstance> loadBalancer = this.clientFactory.getInstance(serviceId,
				ReactorServiceInstanceLoadBalancer.class);
		if (loadBalancer == null) {
			throw new NotFoundException("No loadbalancer available for " + serviceId);
		}
		supportedLifecycleProcessors.forEach(lifecycle -> lifecycle.onStart(lbRequest));
		return loadBalancer.choose(lbRequest);
	}

	private String getHint(String serviceId, Map<String, String> hints) {
		String defaultHint = hints.getOrDefault("default", "default");
		String hintPropertyValue = hints.get(serviceId);
		return hintPropertyValue != null ? hintPropertyValue : defaultHint;
	}

}
```





# 总结



这样 Spring Cloud Gateway 通过这两个过滤器就可完成将请求转发到目标服务器和将目标服务器的响应重写到发起方，为了方便开发者扩展其功能，我们可以自己实现一些过滤器来实现在请求被转发到目标服务器前对请求就行一些自定义处理，比如：修改请求头、修改请求参数、重写请求路径、重写目标服务；在响应被重写回发起方之前对响应就行处理，比如：增加跨域请求头、对响应内容进行加密等；当然我们也可以即处理请求又处理响应，比如：统计请求时间等等功能

**再次提醒**

> 在自己配置 `GlobalFilter` 和 `GatewayFilter` 一定要记得这两个优先级
> 如果想在请求被转发到目标服务器之前处理 Request 其设置的优先级一定要比 `NettyRoutingFilter` 高，即 Order 比 **2147483647** 小
> 如果想在响应被重写会发起方前处理下 Response 其设置的优先级一定要比 `NettyWriteResponseFilter` 低，即 Order 比 **-1** 大



# 参考



GitHub: https://github.com/spring-cloud/spring-cloud-gateway  
Gitee: https://gitee.com/github_mirror_plus/spring-cloud-gateway



# 扩展阅读

除了上面的三个过滤器，在 `org.springframework.cloud.gateway.filter` 包下还有很多有意思的过滤器



**GatewayMetricsFilter** 统计请求时间



```java
public class GatewayMetricsFilter implements GlobalFilter, Ordered {

	private static final Log log = LogFactory.getLog(GatewayMetricsFilter.class);

	private final MeterRegistry meterRegistry;

	private GatewayTagsProvider compositeTagsProvider;

	private final String metricsPrefix;

	public GatewayMetricsFilter(MeterRegistry meterRegistry, List<GatewayTagsProvider> tagsProviders,
			String metricsPrefix) {
		this.meterRegistry = meterRegistry;
		this.compositeTagsProvider = tagsProviders.stream().reduce(exchange -> Tags.empty(), GatewayTagsProvider::and);
		if (metricsPrefix.endsWith(".")) {
			this.metricsPrefix = metricsPrefix.substring(0, metricsPrefix.length() - 1);
		}
		else {
			this.metricsPrefix = metricsPrefix;
		}
	}

	public String getMetricsPrefix() {
		return metricsPrefix;
	}

	@Override
	public int getOrder() {
		// start the timer as soon as possible and report the metric event before we write
		// response to client
		return NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER + 1;
	}

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		Sample sample = Timer.start(meterRegistry);

		return chain.filter(exchange).doOnSuccess(aVoid -> endTimerRespectingCommit(exchange, sample))
				.doOnError(throwable -> endTimerRespectingCommit(exchange, sample));
	}

	private void endTimerRespectingCommit(ServerWebExchange exchange, Sample sample) {

		ServerHttpResponse response = exchange.getResponse();
		if (response.isCommitted()) {
			endTimerInner(exchange, sample);
		}
		else {
			response.beforeCommit(() -> {
				endTimerInner(exchange, sample);
				return Mono.empty();
			});
		}
	}

	private void endTimerInner(ServerWebExchange exchange, Sample sample) {
		Tags tags = compositeTagsProvider.apply(exchange);

		if (log.isTraceEnabled()) {
			log.trace(metricsPrefix + ".requests tags: " + tags);
		}
		sample.stop(meterRegistry.timer(metricsPrefix + ".requests", tags));
	}

}
```



# 鸣谢



感谢同事送的键盘，终于在半夜敲代码的时候，听不到哒哒哒声了