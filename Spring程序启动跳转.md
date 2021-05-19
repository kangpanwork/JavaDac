实现代码：
```java
@Configuration
public class AfterRun {
    @Autowired
    Environment environment;
    @EventListener({ApplicationReadyEvent.class})
    public void test() throws Exception {
        String port = environment.getProperty("local.server.port");
        String url = "http://localhost:" + port;
        Runtime.getRuntime().exec("rundll32 url.dll,FileProtocolHandler " + url);
    }
}
```
### Environment
`Environment`即环境，也可以叫做上下文。`Environment`在容器中是一个抽象的集合，是指应用环境的2个方面：`profiles`和`properties`。
```java
public interface Environment extends PropertyResolver {
    String[] getActiveProfiles();

    String[] getDefaultProfiles();

    /** @deprecated */
    @Deprecated
    boolean acceptsProfiles(String... var1);

    boolean acceptsProfiles(Profiles var1);
}
```
通过 `environment` 获取当前端口拼接 `URL`
### @EventListener
`Spring`提供的一个事件监听、订阅的实现，内部实现原理是观察者设计模式；为的就是业务系统逻辑的解耦,提高可扩展性以及可维护性。事件发布者并不需要考虑谁去监听，监听具体的实现内容是什么，发布者的工作只是为了发布事件而已。
如下例子，控制台打印 `发布的消息:hello`
```java
@SpringBootApplication
public class App implements CommandLineRunner {

    public static void main(String[] args) {

        SpringApplication.run(App.class);
    }

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) throws Exception {
        applicationContext.publishEvent(new MyEvent(this,"hello"));
    }
}
```
```java
public class MyEvent extends ApplicationEvent {

    @Setter
    @Getter
    private String message;

    @Override
    public Object getSource() {
        return super.getSource();
    }

    public MyEvent(Object source,String message) {
        super(source);
        this.message = message;
    }
}
```
```java
@Configuration
@Slf4j
public class MyListener {
    @EventListener
    public void handleDemoEvent(MyEvent event) {
        log.info("发布的消息:{}", event.getMessage());
    }
}
```
### ApplicationReadyEvent
`spring boot`中支持的事件类型如下

- ApplicationFailedEvent：该事件为spring boot启动失败时的操作

- ApplicationPreparedEvent：上下文context准备时触发

- ApplicationReadyEvent：上下文已经准备完毕的时候触发

- ApplicationStartedEvent：spring boot 启动监听类

- SpringApplicationEvent：获取SpringApplication

- ApplicationEnvironmentPreparedEvent：环境事先准备

