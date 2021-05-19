`ConfigurationProperties` 接口
```Java
public @interface ConfigurationProperties {

	@AliasFor("prefix")
	String value() default "";

	@AliasFor("value")
	String prefix() default "";
}
```

在类名上加上 `@ConfigurationProperties` 注解，读取 `application.properties`  配置 `Bean`
```Java
@ConfigurationProperties("spring.datasource")
@Component
@Data
public class DataSourceBean {
    private String url;
    private String username;
    private String password;
}
```
```Java
spring.datasource.url=jdbc:mysql://localhost:3306/kangpan?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=UTC&useAffectedRows=true
spring.datasource.username=root
spring.datasource.password=
```
启动项目 控制台打印 `url`

```Java
@Component
@Slf4j
public class TestRunner implements ApplicationRunner {

    @Autowired
    private DataSourceBean dataSourceBean;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info(dataSourceBean.getUrl());
}
```

配置 `Bean` 的属性
```Java
@RestController
@RequestMapping("/test")
@ConfigurationProperties(prefix = "spring.datasource")
@Slf4j
public class Test {
    private String username;
    public void setUsername(String username) {
        this.username = username;
    }
    @GetMapping(path = "/")
    public String test() {
        return username;
    }
}
```
浏览器输入 `http://localhost:8081/test/` 打印了 `root`

`@Component` 配置 `Bean` （`@Repository @Service @Controller` 等效），定义一个接口 `Todo` 
```Java
public interface Todo {
    void print();
}
```
两个实现类 `Foo`  `Bar`
```Java
@Slf4j
@Component
public class Foo implements Todo {
    @Override
    public void print() {
        log.info("foo");
    }
}
```
```Java
@Slf4j
@Component
public class Bar implements Todo {
    @Override
    public void print() {
        log.info("bar");
    }
}
```

 `@Autowired` 根据类型注入，所有类型为 `Todo` 的全部注入 `list` ( 指 `For` 和 `Bar`) 
`@Inject` 也是按类型注入，不过没有 `required` 属性
`@Qualifier` 根据名称注入
```Java
@Component("todos")
@Data
public class Todos {
    @Autowired(required = false)
    private List<Todo> list;
}
```
```Java
@RestController
@RequestMapping("/test")
@Slf4j
public class Test {

    @Qualifier("todos")
    @Autowired
    private Todos todos;

    @GetMapping(path = "/")
    public String test() {
        todos.getList().stream().forEach( e -> e.print());
        return "";
    }
}
```
浏览器输入 `http://localhost:8081/test/` 控制台打印 `bar` 和 `foo`
如果 `foo` 要优先打印，在 `Foo` 类上加  `@Order(value = 1)`，值越小优先级越高，`Foo` 优先加载

实现延迟加载 定义一个 `LazyDo`
```Java
@Lazy
@Service
@Slf4j
public class LazyDo {

    public void todo() {
        log.info("lazy todo");
    }
}
```
构造注入 `lazyDao`，  `@Resource` 默认按名称注入，匹配不上按类型
```Java
@RestController
@RequestMapping("/test")
@Slf4j
public class Test {

    private LazyDo lazyDo;

    @Lazy
    @Resource(name = "lazyDo")
    private void setLazyDo(LazyDo lazyDo) {
        this.lazyDo = lazyDo;
    }

    @GetMapping(path = "/")
    public String test() {
        lazyDo.todo();
        return "";
    }
}
```
启动项目 `lazyDao` 加载为 `null`

```
this.lazyDo = lazyDo; lazyDao: null lazyDo: "com.kangpan.model.lazy@7a65a360"
```
浏览器访问地址 `http://localhost:8081/test/` 控制台打印 `lazy todo`

` Bean` 默认是单例，在类上加注解 `@Scope("prototype")` 多例

看下 `Bean` 初始化 及 销毁前 执行
```Java
@Service
@Slf4j
public class Todo {

    @PostConstruct
    public void init() {
        log.info("init");
    }

    @PreDestroy
    public void destroy() {
        log.info("destroy");
    }
}
```
运行项目 打印台输出 `init` ，关闭项目 打印台输出 `destroy`

根据当前环境配置 `Bean` 
```Java
@Data
public class TestDev {
    private String name;
    public TestDev(String name) {
        this.name = name;
    }
}
```
`application.properties` 配置 
```Java
test.name=testName // @Value 赋值
spring.profiles.active=dev // 当前环境
```
配置 `dev` 和 `test`  环境下的 `Bean`
```Java
@Configuration
public class Dev {

    @Bean
    @Profile("dev")
    public TestDev testDev(@Value("${test.name}") String name) {
        return new TestDev(name);
    }

    @Bean
    @Profile("test")
    public TestDev testTest(@Value("${test.name}") String name) {
        return new TestDev(name);
    }
}
```
```Java
@RestController
@RequestMapping("/test")
public class Test {

    @Autowired
    private TestDev testDev;


    @GetMapping(path = "/")
    public String test() {
        return testDev.getName();
    }
}
```
浏览器输入 `http://localhost:8081/test/` 打印 testName

`XML` 配置  `Bean` 简化

属性
```Java
<property name = "name",value = "value" />
```
构造参数
```Java
<constructor-arg type="java.lang.String" value="value" />
<constructor-arg ref="refBean" />
```
集合
```Java
<property name = "name">
  <map>
    <entry key="key" value="value">
  </map>
</property>

<property name = "name">
  <map>
    <entry key-ref="keyBean" value-ref="valueBean">
  </map>
</property>
```
引用
```Java
<property name="name" ref="refBean" />
```
或者 `XML` 声明 `p` 命名空间，然后配置 `Bean` ，特殊字符 用 `<![CDATA[]]>` 转义

项目启动 `@SpringBootApplication` 注解中的  `@EnableAutoConfiguration` 会加载配置文件的 `Bean` 

```Java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}

@EnableAutoConfiguration
public @interface SpringBootApplication {
}
```
在 `resources/META-INF/spring.factories` 文件中配置
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.kangpan.model.Configs
```
`Configs` 类

```Java
@EnableConfigurationProperties
@Import({ConfigFoo.class,ConfigBar.class})
public class Configs {

}
```
引入的 `ConfigFoo ConfigBar`  类
```Java
@Slf4j
public class ConfigBar {
    public ConfigBar() {
        log.info("ConfigBar");
    }
}
```
```Java
@Slf4j
public class ConfigFoo {
    public ConfigFoo() {
        log.info("ConfigFoo");
    }
}
```
启动项目 控制台打印 `ConfigBar ConfigFoo`，如果 `@EnableConfigurationProperties` 改成  `@Configuration` 不需要在 `resources/META-INF/spring.factories` 文件中配置
看下 `Configuration` 接口，加了 `@Component` 注解 会实例化 `Bean`
```
@Component
public @interface Configuration {}
```

 `@ConfigurationProperties`  结合 `@EnableConfigurationProperties` 配置Bean
```Java
@ConfigurationProperties("spring.datasource") // 读取  application.properties 中的配置
@Data
public class DataSourceBean2 {
    private String url;
    private String username;
    private String password;
}
```
`application.properties` 文件中配置
```Java
spring.datasource.url=jdbc:mysql://localhost:3306/kangpan?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=UTC&useAffectedRows=true
spring.datasource.username=root
spring.datasource.password=
```
`@EnableConfigurationProperties` 使 `@ConfigurationProperties` 注解的类注入自身
```Java
@EnableConfigurationProperties(DataSourceBean2.class)
@Slf4j
public class DataSourceBeanConfig {

    private DataSourceBean2 dataSourceBean2;

    @Autowired
    public DataSourceBeanConfig(DataSourceBean2 dataSourceBean2) {
        this.dataSourceBean2 = dataSourceBean2;
        log.info("url: {}",dataSourceBean2.getUrl());
    }
}
```
`spring.factories` 配置 或者 `DataSourceBeanConfig` 类 加上 `@Configuration` 或者 `@Component` 注解
```Java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.kangpan.model.DataSourceBeanConfig
```
控制台打印 `url`

自定义 `XML` 配置 `Bean`， 在 `resource/bean.xml` 配置 `dataSourceBean`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       ">
    <bean id="dataSourceBean" class="com.kangpan.model.DataSourceBean"> </bean>
</beans>
```
结合 `@ConfigurationProperties` 属性赋值
```Java
@ConfigurationProperties("spring.datasource")
@Data
public class DataSourceBean {
    private String url;
    private String username;
    private String password;
}
```
利用 `@ImportResource` 导入 `bean.xml` 配置的 `Bean`
```Java
@Component
@Slf4j
@ImportResource("classpath:bean.xml")
public class TestRunner implements ApplicationRunner {

    @Autowired
    private DataSourceBean dataSourceBean;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("dataSourceBean.getUrl():{}",dataSourceBean.getUrl());
}
```
项目启动 控制台打印 `url`

自定义 `properties` 配置 `Bean`，在 `resources/dataSource.properties` 配置
```Java
spring.datasource.url=jdbc:mysql://localhost:3306/kangpan?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=UTC&useAffectedRows=true
spring.datasource.username=root
spring.datasource.password=
```
`@PropertySource` 加载指定文件的属性， `@Value` 赋值
```Java
@PropertySource("classpath:dataSource.properties")  
@Data
@Component
public class DataSourcesBean {
    @Value("${spring.datasource.url}")
    private String url;
    @Value("${spring.datasource.username}")
    private String username;
    @Value("${spring.datasource.password}")
    private String password;
}
```
使用  `@ConfigurationProperties` 属性赋值
```Java
@PropertySource("classpath:dataSource.properties")
@Data
@Component
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourcesBean {
    private String url;
    private String username;
    private String password;
}
```
```Java
@Component
@Slf4j
public class TestRunner implements ApplicationRunner {

    @Autowired
    private DataSourcesBean dataSourcesBean;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("dataSourcesBean.getUrl():{}",dataSourcesBean.getUrl());
}
```
运行项目 打印 `url`



