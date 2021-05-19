`Spring Cloud` 提供了一个接口 `DiscoveryClient` ， 为 `Eureka` 或者 `consul` 等注册中心去实现，  `getInstances` 方法获取注册的实例，一个实例对应一个工程
```
List<ServiceInstance> getInstances(String serviceId);
```
看下接口 `ServiceInstance` 下面两个方法 获取 主机名（IP）/  端口
```
	String getHost();
	int getPort();
```
注册了实例之后，使用  阻塞式同步`RestTemplate` 调用 或者 异步非阻塞 `WebClient` 调用，`RestTemplate` 为每个HTTP请求创建一个线程，在响应之前一直是阻塞状态，占用系统内存资源。

多个实例  `@LoadBalaced` 为 `RestTemplate`  或者 `WebClient` 做负载均衡的支持，`LoadBalancerInterceptor` 实现 `ClientHttpRequestInterceptor` 接口 (`intercept` 方法)
```
	ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution)
			throws IOException;
```
通过请求 `URL` 获取 `host`  然后使用 `LoadBalancerClient` 进行调用
```java
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {

	private LoadBalancerClient loadBalancer; 

    @Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		Assert.state(serviceName != null,
				"Request URI does not contain a valid hostname: " + originalUri);
		return this.loadBalancer.execute(serviceName,
				this.requestFactory.createRequest(request, body, execution));
	}
```

看下 `execute` 方法 默认引用的是 `RibbonLoadBalancerClient`  通过 `serviceId`  和 调用的实例 `ribbonServer`  及 请求`request` 做服务调用
```java
	public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint)
			throws IOException {
		ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
		Server server = getServer(loadBalancer, hint);
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}
		RibbonServer ribbonServer = new RibbonServer(serviceId, server,
				isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));

		return execute(serviceId, ribbonServer, request);
	}
```

配置 `RestTemplate `
```java
@Configuration
public class RestTemplateConfiguration {

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
                .setConnectTimeout(Duration.ofMillis(100))
                .setReadTimeout(Duration.ofMillis(500))
                .requestFactory(this::requestFactory)
                .build();
    }
}
```
