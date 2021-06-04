### 介绍
Spring-Cloud Euraka 是 Spring Cloud 集合中一个组件，它是对 Euraka 的集成，用于服务注册和发现。 Eureka 是 Netflix 中的一个开源框架。它和  zookeeper、Consul 一样，都是用于服务注册管理的，同样， Spring-Cloud  还集成了 Zookeeper 和 Consul 。

在项目中使用 Spring Cloud Euraka 的原因是它可以利用 Spring Cloud Netfilix 中其他的组件，如 zull 等，因为 Euraka 是属于 Netfilix 的。

 Eureka 由多个 instance (服务实例)组成，这些服务实例可以分为两种： Eureka Server 和 Eureka Client 。为了便于理解，我们将 Eureka client 再分为 Service Provider 和 Service Consumer 。

- Eureka Server 提供服务注册和发现
- Service Provider 服务提供方，将自身服务注册到 Eureka ，从而使服务消费方能够找到
- Service Consumer 服务消费方，从 Eureka 获取注册服务列表，从而能够消费服务


### Eureka与Zookeeper比较

首先介绍下 cap 原理，可以参考：[http://www.ruanyifeng.com/blog/2018/07/cap.html](http://www.ruanyifeng.com/blog/2018/07/cap.html。)

- P：Partition tolerance，网络分区容错。类似多机房部署，保证服务稳定性。
- A：Availability，可用性。
- C：Consistency，一致性。

 CAP 定理：CAP 三个属性对于分布式系统不同同时做到。如 AP/CP/AC 。再来看 Zookeepr 区别：

-  Zookeeper 是 CP，分布式协同服务，突出一致性。对 ZooKeeper 的的每次请求都能得到一致的数据结果，但是无法保证每次访问服务可用性。如请求到来时， zookeer 正在做 leader 选举，此时不能提供服务，即不满足 A 可用性。

-  Euere 是 AP ，高可用与可伸缩的 Service 发现服务，突出可用性。相对于 Zookeeper而言，可能返回数据没有一致性，但是保证能够返回数据，服务是可用的。

这里有篇文章介绍为什么服务发现使用 Eureka ，而不是 Zookeeper 。[https://medium.com/knerd/eureka-why-you-shouldnt-use-zookeeper-for-service-discovery-4932c5c7e764](https://medium.com/knerd/eureka-why-you-shouldnt-use-zookeeper-for-service-discovery-4932c5c7e764)

中文翻译  [http://dockone.io/article/78](http://dockone.io/article/78)

### 部署 Eureka Server
 pom.xml 文件手动创建，或者到官方网站 [https://start.spring.io/](https://start.spring.io/) 创建，或者使用 Docker 拉取镜像，注意 springboot 和 springcloud 版本问题[https://start.spring.io/actuator/info](https://start.spring.io/actuator/info)
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springCloudEureka</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
### 启动类
通过 @EnableEurekaSwerver 来标识该服务为 Eureka Server
```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class,args);
    }
}

```
### 配置
```
server.port=8761
eureka.instance.hostname=localhost
eureka.client.registerWithEureka=false
eureka.client.fetchRegistry=false
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:8761/eureka/
```
启动，访问 [http://localhost:8761/](http://localhost:8761/)，这里我把 Eureka Server 打包镜像，每次都用 docker 运行
### Docker 打包镜像
- 使用 mvn install  打包，在 target 目录下生成了 springCloudEureka-1.0-SNAPSHOT.jar 文件
- 新建一个 Dockerfile 文件与 pom 文件并级
```
FROM hub.c.163.com/library/java:8-alpine
ADD target/springCloudEureka-1.0-SNAPSHOT.jar app.jar
EXPOSE 8761
ENTRYPOINT ["java","-jar","/app.jar"]
```
- 制作文件之后，进入 target 目录下使用命令运行 Dockerfile 文件 :  docker build -t springcloud/eureka .
- 生成镜像之后， docker run -p 8761:8761 3046 运行， 3046 为镜像的短 ID，可输入 docker images 查看当前镜像获取 ID

```
docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
springcloud/eureka           latest              304644cb66d4        13 hours ago        189MB
hub.c.163.com/library/java   8-alpine            d991edd81416        4 years ago         145MB
```
```
docker run -p 8762:8762 3046
2021-05-16 02:53:47.611  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$c3183f26] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.4.RELEASE)
```
### 部署 Eureka Client
-  Eureka Client 包括两个服务模块： Service Provider （服务提供方）和 Service Consumer（服务消费方）。
-  Eureka Client 和 Eureka Server 目录类似， 不同点在于：

    - 启动类，使用 @EnableDiscoveryClient  标识该服务为 Euraka Client
    - 配置文件，需要指定 Euraka Server 地址和当前服务注册时的名称。
    

#### 部署 Servcie Provider
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>serviceProvider</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ServiceProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceProviderApplication.class,args);
    }
}
```
```
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
server.port=0
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
```
bootstrap.properties 文件
```
spring.application.name=serviceProvider
```
参数说明：

eureka.client.serviceUrl.defaultZone：指定 Eureka Server 的地址
 spring.application.name ：在 Eureka Server 进行注册时，当前服务的名称。

在提供服务的客户端新建一个测试的服务
```
@RestController
@RequestMapping("/ServiceController")
public class ServiceController {

    @GetMapping
    public String test() {
        return "ServiceController";
    }
}
```
使用 httpie 请求
```
http get http://localhost:52468/ServiceController
HTTP/1.1 200
Content-Length: 17
Content-Type: text/plain;charset=UTF-8
Date: Sun, 16 May 2021 03:44:10 GMT

ServiceController
```
#### 部署 Servcie Customer
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>ClientCustomer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.11</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```
```
@SpringBootApplication
@EnableDiscoveryClient
public class ClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class,args);
    }
}
```
```
public class CustomConnectionKeepAliveStrategy implements ConnectionKeepAliveStrategy {
    private final long DEFAULT_SECONDS = 30;

    @Override
    public long getKeepAliveDuration(HttpResponse response, HttpContext context) {
        return Arrays.asList(response.getHeaders(HTTP.CONN_KEEP_ALIVE))
                .stream()
                .filter(h -> StringUtils.equalsIgnoreCase(h.getName(), "timeout")
                        && StringUtils.isNumeric(h.getValue()))
                .findFirst()
                .map(h -> NumberUtils.toLong(h.getValue(), DEFAULT_SECONDS))
                .orElse(DEFAULT_SECONDS) * 1000;
    }
}
```
版本问题，需要自定义 restTemplate
```
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
    @Bean
    public HttpComponentsClientHttpRequestFactory requestFactory() {
        PoolingHttpClientConnectionManager connectionManager =
                new PoolingHttpClientConnectionManager(30, TimeUnit.SECONDS);
        connectionManager.setMaxTotal(200);
        connectionManager.setDefaultMaxPerRoute(20);

        CloseableHttpClient httpClient = HttpClients.custom()
                .setConnectionManager(connectionManager)
                .evictIdleConnections(30, TimeUnit.SECONDS)
                .disableAutomaticRetries()
                // 有 Keep-Alive 认里面的值，没有的话永久有效
                //.setKeepAliveStrategy(DefaultConnectionKeepAliveStrategy.INSTANCE)
                // 换成自定义的
                .setKeepAliveStrategy(new CustomConnectionKeepAliveStrategy())
                .build();

        HttpComponentsClientHttpRequestFactory requestFactory =
                new HttpComponentsClientHttpRequestFactory(httpClient);

        return requestFactory;
    }
}
```
```
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
server.port=0
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
```
```
spring.application.name=clientCustomer
```
通过注册中心，客户端调用服务端的服务测试
```
@RestController
@RequestMapping("/ClientController")
public class ClientController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping
    public String test() {
        List<ServiceInstance> instanceList = discoveryClient.getInstances("serviceProvider");
        StringBuilder urls = new StringBuilder();
        for(ServiceInstance instance : instanceList) {
            urls.append(instance.getHost() + ":" + instance.getPort()).append(",");
        }
        String result = restTemplate.getForObject("http://serviceProvider/ServiceController",String.class);
        urls.append("ServiceController result:" + result);
        return urls.toString();
    }
}
```
```
 http get http://localhost:53601/ClientController
HTTP/1.1 200
Content-Length: 67
Content-Type: text/plain;charset=UTF-8
Date: Sun, 16 May 2021 04:51:26 GMT

kangpan.mshome.net:52468,ServiceController result:ServiceController
```
### 自我保护模式介绍
保护模式，是 Eureka  提供的一个特性，在默认的情况下，这个属性是打开的，而且也建议线上都使用这个特性。

如果 Eureka Server 在一定时间内没有接收到某个微服务实例的心跳， Eureka Server 将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与 Eureka Server 之间无法正常通信，此时会触发 Eureka Server 进入保护模式，进入自我保护模式后，将会保护服务注册表中的信息，不再删除服务注册表中的数据。参考官网：[https://github.com/Netflix/eureka/wiki/Understanding-Eureka-Peer-to-Peer-Communication](https://github.com/Netflix/eureka/wiki/Understanding-Eureka-Peer-to-Peer-Communication)
#### Eureka Server 端
默认情况下自我保护机制是打开的，线上建议都是打开的，即不需要设置。如果在测试环境需要设置为关闭，可以通过如下配置：
```
# 设为false，关闭自我保护。默认是打开的。
eureka.server.enable-self-preservation=false
```
设置清零失效服务的间隔时间，但是不建议更改
```
# 清理间隔（单位毫秒，默认是60*1000）
eureka.server.eviction-interval-timer-in-ms=4000
```
这个 eureka.server.eviction-interval-timer-in-ms 时间，是 Eureka Server 执行清理无效服务的时间间隔，执行新清理任务时，如果下面判断生效，则清除服务，当前时间 - 上次心跳时间 >  lease-expiration-duration-in-seconds 
其中， lease-expiration-duration-in-seconds  属性在客户端进行配置

这里一个规范就是：服务端的 eureka.server.eviction-interval-timer-in-ms   值要比客户端配置  lease-expiration-duration-in-seconds 的时间短
#### Eureka Client 端
开启健康检查，默认是开启的，如下
```
eureka.client.healthcheck.enabled=true
```
心跳相关的设置
```
# 单位是秒，默认30秒。此客户端发送心跳的频率
eureka.instance.lease-renewal-interval-in-seconds=30
 
# 单位是秒，默认90秒，表示eureka server在收到此client上次心跳之后，间隔多久没有收到，就摘除此服务。
eureka.instance.lease-expiration-duration-in-seconds=10
```
#### kill -9 注册中心自动下线实例
在客户度服务kill -9或者宕机之后，需要自动从主从中心下掉。

1、服务端配置

因为我们测试实例只有一个，所以这里关闭了自我保护，在生成环境不需要设置这个属性，默认是打开的。因为开启自我保护，客户端是单实例时候，在被 kill -9 之后是不会从注册中心下掉的，开启自我保护要保护注册中心注册表信息。
```
# 单位是秒，默认30秒。此客户端发送心跳的频率
eureka.instance.lease-renewal-interval-in-seconds=10
# 单位是秒，默认90秒，表示eureka server在收到此client上次心跳之后，间隔多久没有收到，就摘除此服务。
eureka.instance.lease-expiration-duration-in-seconds=15
# 清理间隔（单位毫秒，默认是60*1000）
eureka.server.eviction-interval-timer-in-ms=3000
# 设为false，关闭自我保护。默认是打开的。
eureka.server.enable-self-preservation=false
```

2、客户端配置
```
# 单位是秒，默认30秒。此客户端发送心跳的频率
eureka.instance.lease-renewal-interval-in-seconds=10
# 单位是秒，默认90秒，表示eureka server在收到此client上次心跳之后，间隔多久没有收到，就摘除此服务。
eureka.instance.lease-expiration-duration-in-seconds=15
```

3、测试结果

发现需要 28s 左右自动下线。为什么是 28s？

 eureka.instance.lease-renewal-interval-in-seconds + eureka.instance.lease-expiration-duration-in-seconds =25 秒。由于定时任务间隔为 3s ，所以下线时间预计为 25s~25+3s  之间。
### Eureka 操作
#### 查看服务实例信息
```
/eureka/apps/{appName}
方法 GET
接口 http://localhost:8761/eureka/apps/配置的`spring.application.name`
```
```
http get http://localhost:8762/eureka/apps/clientCustomer
HTTP/1.1 200
Content-Type: application/xml
Date: Sun, 16 May 2021 05:44:06 GMT
Transfer-Encoding: chunked

<application>
  <name>CLIENTCUSTOMER</name>
  <instance>
    <instanceId>kangpan.mshome.net:clientCustomer:0</instanceId>
    <hostName>kangpan.mshome.net</hostName>
    <app>CLIENTCUSTOMER</app>
    <ipAddr>192.168.137.1</ipAddr>
    <status>UP</status>
    <overriddenstatus>UNKNOWN</overriddenstatus>
    <port enabled="true">53601</port>
    <securePort enabled="false">443</securePort>
    <countryId>1</countryId>
    <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
      <name>MyOwn</name>
    </dataCenterInfo>
    <leaseInfo>
      <renewalIntervalInSecs>30</renewalIntervalInSecs>
      <durationInSecs>90</durationInSecs>
      <registrationTimestamp>1621140664737</registrationTimestamp>
      <lastRenewalTimestamp>1621141625008</lastRenewalTimestamp>
      <evictionTimestamp>0</evictionTimestamp>
      <serviceUpTimestamp>1621139863310</serviceUpTimestamp>
    </leaseInfo>
    <metadata class="java.util.Collections$EmptyMap"/>
    <homePageUrl>http://kangpan.mshome.net:53601/</homePageUrl>
    <statusPageUrl>http://kangpan.mshome.net:53601/actuator/info</statusPageUrl>
    <healthCheckUrl>http://kangpan.mshome.net:53601/actuator/health</healthCheckUrl>
    <vipAddress>clientCustomer</vipAddress>
    <secureVipAddress>clientCustomer</secureVipAddress>
    <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
    <lastUpdatedTimestamp>1621140664737</lastUpdatedTimestamp>
    <lastDirtyTimestamp>1621140664672</lastDirtyTimestamp>
    <actionType>ADDED</actionType>
  </instance>
</application>
```
#### 优雅停机
```
#开启所有的端点
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
#启用shutdown
management.endpoint.shutdown.enabled=true
#禁用密码验证
management.endpoint.shutdown.sensitive=false
```
```
 http post http://kangpan:8089/actuator/shutdown
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8
Date: Sun, 16 May 2021 06:10:36 GMT
Transfer-Encoding: chunked

{
    "message": "Shutting down, bye..."
}
```
### 参考
Spring Cloud 的 Eureka 的官网文档：http://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html

Spring Cloud 的官方文档：https://springcloud.cc/spring-cloud-dalston.html

Eureka 的相关接口 https://github.com/Netflix/eureka/wiki/Eureka-REST-operations


