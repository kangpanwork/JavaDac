搭建一个 `springboot` 的项目，引入  `spring-boot-starter-parent`父组件，` spring-boot-starter-web`组件， `lombok`组件，`jedis`客户端组件   ，谷歌的`guava` 组件
创建 `Runners` 类
```java
@Slf4j
@Component
public class Runners implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        Jedis jedis = new Jedis("127.0.0.1");
        String lua = "return ARGV[1]"; // lua 脚本
        String result = (String)jedis.eval(lua,0,"100"); // 0个key 值100
        log.info("result:{}",result);
    }
}
```
启动项目 `result` 打印 `100`
命令格式
```
EVAL lua脚本 numkeys key [key ...] arg [arg ...]
```
- `numkeys` 指 key 的数量
- `key [key ...]`，键，多个，在`lua`脚本中通过`KEYS[1], KEYS[2]`获取
- `arg [arg ...]`，值，多个，在`lua`脚本中通过`ARGV[1], ARGV[2]`获取

`ScriptingCommands` 类 `jedis` 对 `lua` 命令的支持
``` java
public interface ScriptingCommands {
  Object eval(String script, int keyCount, String... params);

  Object eval(String script, List<String> keys, List<String> args);

  Object eval(String script);

  Object evalsha(String sha1);

  Object evalsha(String sha1, List<String> keys, List<String> args);

  Object evalsha(String sha1, int keyCount, String... params);

  Boolean scriptExists(String sha1);

  List<Boolean> scriptExists(String... sha1);

  String scriptLoad(String script);
}
```
`Runners` 类改造
``` java
        Jedis jedis = new Jedis("127.0.0.1");
        String lua = "redis.call('set', KEYS[1], ARGV[1])";
        List<String> keys = new ArrayList<>();
        List<String> values = new ArrayList<>();
        keys.add("key");
        values.add("100");
        jedis.eval(lua,keys,values);
```
打开 `redis` 客户端 
```
127.0.0.1:6379> get key
"100"
```
或者 修改
```
String lua = "local str = redis.call('get', KEYS[1]);return str";
String result = (String) jedis.eval(lua,keys,values);
log.info(result);
```
控制台打印 `100`
使用 `Spring Data Redis` 组件，支持的客户端 `Jedis / Lettuce`，配置 `RedisTemplate`
```java
@Configuration
@AutoConfigureAfter(RedisAutoConfiguration.class)
public class RedisConfig {
    /**
     * 默认情况下的模板只能支持RedisTemplate<String, String>，也就是只能存入字符串，因此支持序列化
     */
    @Bean
    public RedisTemplate<String, Serializable> redisTemplate(LettuceConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Serializable> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```
```java
@Slf4j
@Component
public class Runners implements ApplicationRunner {

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        String lua = "redis.call('EXPIRE', KEYS[1], ARGV[2]);"; // 这里使用的是第二个参数30， 101未用到
        ImmutableList<Object> keys = ImmutableList.of("key");
        RedisScript<Number> redisScript = new DefaultRedisScript<>(lua, Number.class);
        redisTemplate.execute(redisScript,keys,"101",30);
    }
}
```
`key` 有效时间为 `30` 秒，`30` 秒之后查询 `nil`
```
127.0.0.1:6379> get key
"100"
127.0.0.1:6379> get key
(nil)
```
redis 可视化客户端 `https://github.com/qishibo/AnotherRedisDesktopManager`，查看有效时间`30`秒递减
更改 `Runners` 类
```java
        String lua = " redis.call('INCR',KEYS[1]);";
        ImmutableList<Object> keys = ImmutableList.of("key");
        RedisScript<Number> redisScript = new DefaultRedisScript<>(lua, Number.class);
        redisTemplate.execute(redisScript,keys,"100",120);
```
执行命令，值增加 `1`
```
127.0.0.1:6379> get key
"100"
127.0.0.1:6379> get key
"101"
127.0.0.1:6379>
```
以上例子涉及 `redis` 四个命令 `SET`，`GET`， `EXPIRE`， `INCR`

>`INCR`    对存储在指定 `key`的数值执行原子的加`1`操作
如果指定的`key`不存在，那么在执行`incr`操作之前，会先将它的值设定为`0`

>`EXPIRE` 设置`key`的过期时间，超过时间后，将会自动删除该`key`

查看其它命令 `http://www.redis.cn/commands.html`
详细`lua`学习 `https://github.com/52fhy/lua-book`