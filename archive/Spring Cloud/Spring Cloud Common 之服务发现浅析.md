# Spring Cloud 是如何实现服务治理的

> 文档写的再好，也不如源码写的好  
> 源码地址：  
> Spring Cloud Consul https://github.com/spring-cloud/spring-cloud-consul  
> Spring Cloud Commons https://github.com/spring-cloud/spring-cloud-commons

---

**Table of Contents**

- [Spring Cloud 是如何实现服务治理的](#spring-cloud-是如何实现服务治理的)
  - [Spring Cloud Commons 之服务治理浅析](#spring-cloud-commons-之服务治理浅析)
    - [服务注册](#服务注册)
    - [服务发现](#服务发现)
    - [健康检测](#健康检测)
  - [Spring Cloud Consul 实现](#spring-cloud-consul-实现)
    - [实现 **ServiceRegistry** 功能](#实现-serviceregistry-功能)
    - [总结](#总结)
- [参考](#参考)

---

## Spring Cloud Commons 之服务治理浅析

Spring 在设计的时候，通常会考虑方便扩展和消除样板代码，在 Spring Clond 同样存在这样的设计。

在 Spring Cloud 体系中，Spring Cloud Commons 是最重要的一个项目，其中定义了服务注册、服务发现、复杂均衡相关的接口以及一些公共组件，通过看这个项目，我们可以简单的理解一下 Spring Cloud 注册发现的核心流程。



Spring Clond Commons 项目中提供了如下的项目结构（在这里省略了部分代码文件和结构）

```

└── src
    ├── main
    │   ├── java
    │   │   └── org
    │   │       └── springframework
    │   │           └── cloud
    │   │               ├── client
    │   │               │   ├── DefaultServiceInstance.java
    │   │               │   ├── ServiceInstance.java  Spring Cloud 对服务实例信息的定义
    │   │               │   ├── discovery 服务发现相关
    │   │               │   │   ├── DiscoveryClient.java
    │   │               │   │   ├── EnableDiscoveryClient.java
    │   │               │   │   ├── EnableDiscoveryClientImportSelector.java
    │   │               │   │   ├── ManagementServerPortUtils.java
    │   │               │   │   ├── ReactiveDiscoveryClient.java
    │   │               │   │   ├── composite
    │   │               │   │   │   ├── CompositeDiscoveryClient.java
    │   │               │   │   │   ├── CompositeDiscoveryClientAutoConfiguration.java
    │   │               │   │   │   └── reactive
    │   │               │   │   │       ├── ReactiveCompositeDiscoveryClient.java
    │   │               │   │   │       └── ReactiveCompositeDiscoveryClientAutoConfiguration.java
    │   │               │   │   ├── health 健康检查相关
    │   │               │   │       ├── DiscoveryClientHealthIndicator.java
    │   │               │   │       ├── DiscoveryClientHealthIndicatorProperties.java
    │   │               │   │       ├── DiscoveryCompositeHealthContributor.java
    │   │               │   │       ├── DiscoveryHealthIndicator.java
    │   │               │   │       └── reactive
    │   │               │   │           ├── ReactiveDiscoveryClientHealthIndicator.java
    │   │               │   │           ├── ReactiveDiscoveryCompositeHealthContributor.java
    │   │               │   │           └── ReactiveDiscoveryHealthIndicator.java
    │   │               │   ├── loadbalancer 这下面是负载均衡相关逻辑
    │   │               │   └── serviceregistry 服务注册相关
    │   │               │       ├── AbstractAutoServiceRegistration.java
    │   │               │       ├── AutoServiceRegistration.java
    │   │               │       ├── AutoServiceRegistrationAutoConfiguration.java
    │   │               │       ├── AutoServiceRegistrationConfiguration.java
    │   │               │       ├── AutoServiceRegistrationProperties.java
    │   │               │       ├── Registration.java
    │   │               │       ├── ServiceRegistry.java
    │   │               │       ├── ServiceRegistryAutoConfiguration.java
    │   │               ├── commons
    │   │                   ├── httpclient http 工厂类，在配置中可以选择使用 Apache Http 还是 OKHttp
    │   │                   │   ├── ApacheHttpClientFactory.java
    │   │                   │   └── OkHttpClientFactory.java
    │   │                   └── util
    │   │                      ├── IdUtils.java   通过这工具类来生成实例 id
    │   │                      └── InetUtils.java Spring Cloud 就是通过这个工具类是获取服务项目的 ip 地址的
    │   └── resources
    │       └── META-INF
    │           ├── additional-spring-configuration-metadata.json
    │           └── spring.factories
    └── test
        ├── java 测试相关代码

```



在项目结构中可以看出各个部分对应的源码，在服务治理中，首先是服务信息 `ServiceInstance` , 其中包括

- 服务名 ServiceId 这个就是我们类似的 xxx-server （spring.application.name）
- 服务实例唯一标识符 InstanceId
- host
- port
- 一些扩展信息 metadata, 这个主要用来提供给三方实现增加以下扩展信息

```java
// 为了缩短篇幅，删除了一些注释
public interface ServiceInstance {

	default String getInstanceId() {
		return null;
	}

	String getServiceId();

	String getHost();

	int getPort();

	boolean isSecure();

	URI getUri();

	Map<String, String> getMetadata();

	default String getScheme() {
		return null;
	}

}
```



### 服务注册

**Registration** 是 Spring Cloud 提供的一个注册实现

```java
public interface Registration extends ServiceInstance {
	// 这里面是真没有代码
}
```



服务注册的实际接口是 **ServiceRegistry**

```java
public interface ServiceRegistry<R extends Registration> {

	/**
	 * Registers the registration. A registration typically has information about an
	 * instance, such as its hostname and port.
	 * @param registration registration meta data
	 */
	void register(R registration);

	/**
	 * Deregisters the registration.
	 * @param registration registration meta data
	 */
	void deregister(R registration);

	/**
	 * Closes the ServiceRegistry. This is a lifecycle method.
	 */
	void close();

	/**
	 * Sets the status of the registration. The status values are determined by the
	 * individual implementations.
	 * @param registration The registration to update.
	 * @param status The status to set.
	 * @see org.springframework.cloud.client.serviceregistry.endpoint.ServiceRegistryEndpoint
	 */
	void setStatus(R registration, String status);

	/**
	 * Gets the status of a particular registration.
	 * @param registration The registration to query.
	 * @param <T> The type of the status.
	 * @return The status of the registration.
	 * @see org.springframework.cloud.client.serviceregistry.endpoint.ServiceRegistryEndpoint
	 */
	<T> T getStatus(R registration);

}
```



通过实现 **ServiceRegistry** 即可完成一个简单服务注册功能



### 服务发现

在 discovery 下存在两个服务发现定义接口 `DiscoveryClient` 和 `ReactiveDiscoveryClient`

其提供了如下功能：

1. 获取所有的服务名称
2. 根据服务名称获取对应的服务实例列表

```java
public interface DiscoveryClient extends Ordered {

	/**
	 * Default order of the discovery client.
	 */
	int DEFAULT_ORDER = 0;

	/**
	 * A human-readable description of the implementation, used in HealthIndicator.
	 * @return The description.
	 */
	String description();

	/**
	 * Gets all ServiceInstances associated with a particular serviceId.
	 * @param serviceId The serviceId to query.
	 * @return A List of ServiceInstance.
	 */
	List<ServiceInstance> getInstances(String serviceId);

	/**
	 * @return All known service IDs.
	 */
	List<String> getServices();

	/**
	 * Default implementation for getting order of discovery clients.
	 * @return order
	 */
	@Override
	default int getOrder() {
		return DEFAULT_ORDER;
	}

}
```

通过实现 `DiscoveryClient` 即可完成服务发现

### 健康检测

ReactiveDiscoveryClientHealthIndicator 提供了健康检测功能

1. 从 DiscoveryClient 中获取所有的服务名列表
2. 根据服务名列表获取对应的服务实例列表
3. 对每个实例进行健康检测，如果响应成功则 UP 否则为 DOWN

```java
public class ReactiveDiscoveryClientHealthIndicator
		implements ReactiveDiscoveryHealthIndicator, Ordered, ApplicationListener<InstanceRegisteredEvent<?>> {

	private final ReactiveDiscoveryClient discoveryClient;

	private final DiscoveryClientHealthIndicatorProperties properties;

	private final Log log = LogFactory.getLog(ReactiveDiscoveryClientHealthIndicator.class);

	private AtomicBoolean discoveryInitialized = new AtomicBoolean(false);

	private int order = Ordered.HIGHEST_PRECEDENCE;

	public ReactiveDiscoveryClientHealthIndicator(ReactiveDiscoveryClient discoveryClient,
			DiscoveryClientHealthIndicatorProperties properties) {
		this.discoveryClient = discoveryClient;
		this.properties = properties;
	}

	@Override
	public void onApplicationEvent(InstanceRegisteredEvent<?> event) {
		if (this.discoveryInitialized.compareAndSet(false, true)) {
			this.log.debug("Discovery Client has been initialized");
		}
	}

	@Override
	public Mono<Health> health() {
		if (this.discoveryInitialized.get()) {
			return doHealthCheck();
		}
		else {
			return Mono.just(
					Health.status(new Status(Status.UNKNOWN.getCode(), "Discovery Client not initialized")).build());
		}
	}

	private Mono<Health> doHealthCheck() {
		// @formatter:off
		return Mono.justOrEmpty(this.discoveryClient)
				.flatMapMany(ReactiveDiscoveryClient::getServices)
				.collectList()
				.defaultIfEmpty(emptyList())
				.map(services -> {
					ReactiveDiscoveryClient client = this.discoveryClient;
					String description = (this.properties.isIncludeDescription())
							? client.description() : "";
					return Health.status(new Status("UP", description))
							.withDetail("services", services).build();
				})
				.onErrorResume(exception -> {
					this.log.error("Error", exception);
					return Mono.just(Health.down().withException(exception).build());
				});
		// @formatter:on
	}

	@Override
	public String getName() {
		return discoveryClient.description();
	}

	@Override
	public int getOrder() {
		return this.order;
	}

	public void setOrder(int order) {
		this.order = order;
	}

}
```



通过上面的接口定义和自带的健康检测逻辑可以看出做一个服务治理需要实现的最简单的逻辑

1. 实现 **ServiceRegistry** 功能
2. 实现 **DiscoveryClient** 功能



## Spring Cloud Consul 实现



### 实现 **ServiceRegistry** 功能 

在 Spring Cloud Consul 中，首先自定义了 **Registration** 的实现



其中 `NewService` 为 Consul 定义的一些服务实例信息 



```java
public class ConsulRegistration implements Registration {

	private final NewService service;

	private ConsulDiscoveryProperties properties;

	public ConsulRegistration(NewService service, ConsulDiscoveryProperties properties) {
		this.service = service;
		this.properties = properties;
	}

	public NewService getService() {
		return this.service;
	}

	protected ConsulDiscoveryProperties getProperties() {
		return this.properties;
	}

	public String getInstanceId() {
		return getService().getId();
	}

	public String getServiceId() {
		return getService().getName();
	}

	@Override
	public String getHost() {
		return getService().getAddress();
	}

	@Override
	public int getPort() {
		return getService().getPort();
	}

	@Override
	public boolean isSecure() {
		return this.properties.getScheme().equalsIgnoreCase("https");
	}

	@Override
	public URI getUri() {
		return DefaultServiceInstance.getUri(this);
	}

	@Override
	public Map<String, String> getMetadata() {
		return getService().getMeta();
	}

}
```



**NewService**

其包含了服务的基本信息和 Consul 本身提供一些特有功能如：Tags、Check 

```java
// 删除了通用的 getter、setter、toString 方法
public class NewService {
  @SerializedName("ID")
  private String id;
  @SerializedName("Name")
  private String name;
  @SerializedName("Tags")
  private List<String> tags;
  @SerializedName("Address")
  private String address;
  @SerializedName("Meta")
  private Map<String, String> meta;
  @SerializedName("Port")
  private Integer port;
  @SerializedName("EnableTagOverride")
  private Boolean enableTagOverride;
  @SerializedName("Check")
  private NewService.Check check;
  @SerializedName("Checks")
  private List<NewService.Check> checks;

  public NewService() {
  }

  public static class Check {
    @SerializedName("Script")
    private String script;
    @SerializedName("DockerContainerID")
    private String dockerContainerID;
    @SerializedName("Shell")
    private String shell;
    @SerializedName("Interval")
    private String interval;
    @SerializedName("TTL")
    private String ttl;
    @SerializedName("HTTP")
    private String http;
    @SerializedName("Method")
    private String method;
    @SerializedName("Header")
    private Map<String, List<String>> header;
    @SerializedName("TCP")
    private String tcp;
    @SerializedName("Timeout")
    private String timeout;
    @SerializedName("DeregisterCriticalServiceAfter")
    private String deregisterCriticalServiceAfter;
    @SerializedName("TLSSkipVerify")
    private Boolean tlsSkipVerify;
    @SerializedName("Status")
    private String status;
    @SerializedName("GRPC")
    private String grpc;
    @SerializedName("GRPCUseTLS")
    private Boolean grpcUseTLS;

    public Check() {
    }
  }
}
```

**ConsulServiceRegistry** 实现 **ServiceRegistry**

```java
public class ConsulServiceRegistry implements ServiceRegistry<ConsulRegistration> {

	private static Log log = LogFactory.getLog(ConsulServiceRegistry.class);

	private final ConsulClient client;

	private final ConsulDiscoveryProperties properties;

	private final TtlScheduler ttlScheduler;

	private final HeartbeatProperties heartbeatProperties;

	public ConsulServiceRegistry(ConsulClient client, ConsulDiscoveryProperties properties, TtlScheduler ttlScheduler,
			HeartbeatProperties heartbeatProperties) {
		this.client = client;
		this.properties = properties;
		this.ttlScheduler = ttlScheduler;
		this.heartbeatProperties = heartbeatProperties;
	}

	@Override
	public void register(ConsulRegistration reg) {
		log.info("Registering service with consul: " + reg.getService());
		try {
			// 同样是通过 consul 提供的 api 接口进行服务注册
			this.client.agentServiceRegister(reg.getService(), this.properties.getAclToken());
			NewService service = reg.getService();
			if (this.heartbeatProperties.isEnabled() && this.ttlScheduler != null && service.getCheck() != null
					&& service.getCheck().getTtl() != null) {
				this.ttlScheduler.add(reg.getInstanceId());
			}
		}
		catch (ConsulException e) {
			if (this.properties.isFailFast()) {
				log.error("Error registering service with consul: " + reg.getService(), e);
				ReflectionUtils.rethrowRuntimeException(e);
			}
			log.warn("Failfast is false. Error registering service with consul: " + reg.getService(), e);
		}
	}

	@Override
	public void deregister(ConsulRegistration reg) {
		if (this.ttlScheduler != null) {
			this.ttlScheduler.remove(reg.getInstanceId());
		}
		if (log.isInfoEnabled()) {
			log.info("Deregistering service with consul: " + reg.getInstanceId());
		}
		this.client.agentServiceDeregister(reg.getInstanceId(), this.properties.getAclToken());
	}

	@Override
	public void close() {

	}

	@Override
	public void setStatus(ConsulRegistration registration, String status) {
		if (status.equalsIgnoreCase(OUT_OF_SERVICE.getCode())) {
			this.client.agentServiceSetMaintenance(registration.getInstanceId(), true);
		}
		else if (status.equalsIgnoreCase(UP.getCode())) {
			this.client.agentServiceSetMaintenance(registration.getInstanceId(), false);
		}
		else {
			throw new IllegalArgumentException("Unknown status: " + status);
		}

	}

  // 服务实例状态
	@Override
	public Object getStatus(ConsulRegistration registration) {
		String serviceId = registration.getServiceId();
		Response<List<Check>> response = this.client.getHealthChecksForService(serviceId,
				HealthChecksForServiceRequest.newBuilder().setQueryParams(QueryParams.DEFAULT).build());
		List<Check> checks = response.getValue();

		for (Check check : checks) {
			if (check.getServiceId().equals(registration.getInstanceId())) {
				if (check.getName().equalsIgnoreCase("Service Maintenance Mode")) {
					return OUT_OF_SERVICE.getCode();
				}
			}
		}

		return UP.getCode();
	}
}
```

**ConsulDiscoveryClient** 实现 **DiscoveryClient**

在发现逻辑中也是通过 consul 提供的 api 接口进行查询

```java
public class ConsulDiscoveryClient implements DiscoveryClient {

	private final ConsulClient client;

	private final ConsulDiscoveryProperties properties;

	public ConsulDiscoveryClient(ConsulClient client, ConsulDiscoveryProperties properties) {
		this.client = client;
		this.properties = properties;
	}

	@Override
	public String description() {
		return "Spring Cloud Consul Discovery Client";
	}

	@Override
	public List<ServiceInstance> getInstances(final String serviceId) {
		return getInstances(serviceId, new QueryParams(this.properties.getConsistencyMode()));
	}

	public List<ServiceInstance> getInstances(final String serviceId, final QueryParams queryParams) {
		List<ServiceInstance> instances = new ArrayList<>();

		addInstancesToList(instances, serviceId, queryParams);

		return instances;
	}

	private void addInstancesToList(List<ServiceInstance> instances, String serviceId, QueryParams queryParams) {
		HealthServicesRequest.Builder requestBuilder = HealthServicesRequest.newBuilder()
				.setPassing(this.properties.isQueryPassing()).setQueryParams(queryParams)
				.setToken(this.properties.getAclToken());
		String queryTag = this.properties.getQueryTagForService(serviceId);
		if (queryTag != null) {
			requestBuilder.setTag(queryTag);
		}
		HealthServicesRequest request = requestBuilder.build();
		Response<List<HealthService>> services = this.client.getHealthServices(serviceId, request);

		for (HealthService service : services.getValue()) {
			instances.add(new ConsulServiceInstance(service, serviceId));
		}
	}

	public List<ServiceInstance> getAllInstances() {
		List<ServiceInstance> instances = new ArrayList<>();

		Response<Map<String, List<String>>> services = this.client
				.getCatalogServices(CatalogServicesRequest.newBuilder().setQueryParams(QueryParams.DEFAULT).build());
		for (String serviceId : services.getValue().keySet()) {
			addInstancesToList(instances, serviceId, QueryParams.DEFAULT);
		}
		return instances;
	}

	@Override
	public List<String> getServices() {
		CatalogServicesRequest request = CatalogServicesRequest.newBuilder().setQueryParams(QueryParams.DEFAULT)
				.setToken(this.properties.getAclToken()).build();
		return new ArrayList<>(this.client.getCatalogServices(request).getValue().keySet());
	}

	@Override
	public int getOrder() {
		return this.properties.getOrder();
	}

}
```



### 总结

简要的 Spring Cloud Consul 的服务治理逻辑大致如此，当然 Spring Cloud Consul 还要处理大量的细节，代码还是很多的

> 在 Spring Cloud 体系中 Consul 并不提供服务请求转发的功能，只是提供对服务信息的保存、查询、健康检测剔除功能

# 参考

1. Consul 官方介绍 https://www.consul.io/docs/intro
2. Spring Cloud Consul https://github.com/spring-cloud/spring-cloud-consul
3. Spring Cloud Commons https://github.com/spring-cloud/spring-cloud-commons

