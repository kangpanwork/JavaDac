安利一款好用的谷歌`json`格式化插件 `jsonview`，地址 [https://jsonview.com/](https://jsonview.com/)
搭建 `springboot` 项目 引入 `actuator` 组件
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```
 `actuator` 目的：监控并管理应用程序，详情见官方文档 [https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready)

一些常用的 `Endpoint`

| id      | 说明  | 默认开启  | 默认HTTP | 默认JMX|
| :-: |:-:| :-:|:-:|:-:|
| beans |   显示容器的Bean | Y        |N|Y|
| health    | 显示健康信息      | Y        |Y|Y|
| info  | 应用信息       | Y        |Y |Y|
| cache  | 应用缓存       | Y        |N|Y|

`application.properties` 配置打开所有监控
```
management.endpoints.web.exposure.include=*
```
浏览器输入 `http://localhost:8080/actuator/health` 打印
```
{
status: "UP"
}
```
进入 安装的`JDK/bin` 目录 ，`cmd` 输入 `jconsole` 可以看到 `Actuator EndPoint` 信息
`Spring boot` 健康检查 通过实现 `HealthIndicator` 接口
``` java
@FunctionalInterface
public interface HealthIndicator extends HealthContributor {

	default Health getHealth(boolean includeDetails) {
		Health health = health();
		return includeDetails ? health : health.withoutDetails();
	}
	Health health();

}
```
举个例子 数据库的健康检查 `DataSourceHealthIndicator` 类，构造的时候传入  `DataSource`
```java
public DataSourceHealthIndicator(DataSource dataSource) {
		this(dataSource, null);
	}
```
健康检查
```java
	@Override
	protected void doHealthCheck(Health.Builder builder) throws Exception {
		if (this.dataSource == null) {
			builder.up().withDetail("database", "unknown"); // 没有数据库 就当是健康的 并提示 unknown
		}
		else {
			doDataSourceHealthCheck(builder);
		}
	}
```
`doDataSourceHealthCheck` 方法判断当前数据库健康状态
``` java
	private void doDataSourceHealthCheck(Health.Builder builder) throws Exception {
		String product = getProduct(); // 什么数据库
		builder.up().withDetail("database", product);
		String validationQuery = getValidationQuery(product);
		try {
			// Avoid calling getObject as it breaks MySQL on Java 7
			List<Object> results = this.jdbcTemplate.query(validationQuery, new SingleColumnRowMapper()); // 通过 jdbcTemplate 查询
			Object result = DataAccessUtils.requiredSingleResult(results); // 取得结果
			builder.withDetail("result", result);
		}
		finally {
			builder.withDetail("validationQuery", validationQuery);
		}
	}
```
实现自己程序的健康状态 定义 `HealthIndicatorTest` 类
``` java
@Component
public class HealthIndicatorTest implements HealthIndicator {

    @Override
    public Health health() {
        Map<String,Object> map = new HashMap<>();
        // 这里根据你的业务逻辑去判断
        map.put("血压","正常");
        map.put("体重",120);
        return Health.up().withDetails(map).build();
    }
}
```
浏览器输入 `http://localhost:8080/actuator/health` 查看结果
```
healthIndicatorTest: {
status: "UP",
details: {
  体重: 120,
  血压: "正常"
  }
}
```
上面通过 `Actuator Health` 了解程序是否健康，在程序中还需要收集更多的度量指标，比如操作系统 或者 JVM、业务指标 等，通过 `Micrometer` 收集这些信息，官方网站 [https://micrometer.io/](https://micrometer.io/) ，应用在 `Spring Boot`， 查看文档  [https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics)

利用 `Micrometer` 后端埋点 统计接口请求次数 
`pom` 引入组件
``` xml
	<dependency>
			<groupId>io.micrometer</groupId>
			<artifactId>micrometer-registry-prometheus</artifactId>
		</dependency>
```
新建 `Test` 类
``` java
@RestController
@RequestMapping("/test")
@Slf4j
public class Test implements MeterBinder {

    private Counter counter;

    @GetMapping(path = "/")
    public String test() {
        counter.increment(); // 埋点
        return "";
    }

    @Override
    public void bindTo(MeterRegistry registry) {
        this.counter = this.counter = registry.counter("counter.number");
    }
}
```
浏览器 输入 `http://localhost:8081/test/` 请求服务
输入 `http://localhost:8081/actuator/metrics/counter.number` 查看请求次数
 ```
{
name: "counter.number",
description: null,
baseUnit: null,
measurements: [
{
statistic: "COUNT",
value: 1
}
],
availableTags: [ ]
}
 ```
 `windows` 安装 `grafana` ，地址 ： [https://grafana.com/grafana/download?platform=windows](https://grafana.com/grafana/download?platform=windows)
安装之后，打开 `\grafana\conf` 文件 查看配置
- 默认配置文件是在/conf/defaults.ini
- 用户配置文件是在/conf/custom.ini
更改 端口
```
[server]
http_port = 8888
```
更改 账号密码

```
[security]
# default admin user, created on startup
admin_user = admin
# default admin password, can be changed before first start of grafana, or in profile settings
admin_password = admin
```


进入目录 `grafana\bin` 运行 `grafana-server.exe`
浏览器输入 `http://localhost:8888/` ，输入 `admin admin`


![](/2.jpg)


 `windows` 安装 `prometheus` ，地址 ：[https://prometheus.io/download/](https://prometheus.io/download/)
配置 任务和实例，打开 `prometheus.yml` 文件，详情见 [任务和实例](https://www.prometheus.wang/quickstart/prometheus-job-and-instance.html)
```
scrape_configs:
  #The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    #metrics_path defaults to '/metrics'
    #scheme defaults to 'http'.
    
    metrics_path: '/actuator/prometheus'

    static_configs:
    - targets: ['localhost:8080']
    
  - job_name: 'node'

    #metrics_path defaults to '/metrics'
    #scheme defaults to 'http'.
    
    static_configs:
    - targets: ['localhost:9182']
```


浏览器输入 `http://localhost:9090/targets` ，`prometheus` 默认端口 `9090`，看到这两个实例都是 `down` 状态


![](/5.jpg)


了解下  `prometheus` 架构，`prometheus` 主要是通过定时拉取应用程序中暴露的时间序列进行工作的，可以在 `prometheus.yml` 文件配置 `Jobs/Exporters`

![](/6.jpg)


为什么要监控应用程序的各个指标，推荐文章 [度量驱动开发](https://www.infoq.cn/article/metrics-driven-development) 感兴趣的可以看下

`springboot` 引入  `micrometer-jvm-extras` 组件，查看 JVM 相关信息
``` xml
        <dependency>
            <groupId>io.github.mweirauch</groupId>
            <artifactId>micrometer-jvm-extras</artifactId>
            <version>0.1.4</version>
        </dependency>
```
启动项目，看到 `springboot` 项目实例 状态是 `up`


![](/7.jpg)

点击 `Endpoint` 可以看到 `JVM` 各项指标

![](/8.jpg)

通过 `grafana` 图形化展示，`JVM` 堆内存 进程内存  `HTTP`请求持续时间  详情见 `4701` 指标 

[https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards) 

![](/9.jpg)

参考文章 
[Spring Boot Actuator:健康检查、审计、统计和监控](https://bigjar.github.io/2018/08/19/Spring-Boot-Actuator-%E5%81%A5%E5%BA%B7%E6%A3%80%E6%9F%A5%E3%80%81%E5%AE%A1%E8%AE%A1%E3%80%81%E7%BB%9F%E8%AE%A1%E5%92%8C%E7%9B%91%E6%8E%A7/)
[Spring Boot Metrics监控之Prometheus&amp;Grafana](https://bigjar.github.io/2018/08/19/Spring-Boot-Metrics%E7%9B%91%E6%8E%A7%E4%B9%8BPrometheus-Grafana/)