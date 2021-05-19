### jackson
`springBoot` 项目中，当引入 `spring-boot-starter-web` 组件，会自动引入了 `Jackson` 相关组件，`@RestController` 注解和`@ResponseBody`注解使用的`Jackson`组件将对象转化`json`返回给的前端。自动引入了 `Jackson` 相关组件如下，其中 `jackson-databind` 包含了另外两个组件。
```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```
### 过滤 NULL、空、默认值
`application.properties`文件中配置默认值不参与转化，全局配置，所有转化都生效
```
spring.jackson.default-property-inclusion=NON_DEFAULT
```
举个例子
```
@Data
@Component
@ConfigurationProperties("local.user")
public class User {
    private String name;
    private int age;
    private int id;
    private String sex;
    private String love;
}
```
`application.properties`文件中配置 `user` 信息
```
local.user.id=1
local.user.sex=boy
local.user.love=love
```
请求`URL` http://localhost:8080/user 查看接口返回值
```
@RestController
public class Controller {

    @Autowired
    private User user;

    @GetMapping(path = "/user")
    public User test() {
        return user;
    }
}
```
```
{
id: 1,
sex: "boy",
love: "love"
}
```
其它配置
- 都参与
`spring.jackson.default-property-inclusion=ALWAYS`

- 为默认值不参与
`spring.jackson.default-property-inclusion=NON_DEFAULT`

- "" 或者 null 不参与
`spring.jackson.default-property-inclusion=NON_EMPTY`

- null 不参与
`spring.jackson.default-property-inclusion=NON_NULL` 

使用注解的形式 `@JsonInclude`，只对当前实例转化生效
```
@Data
@Component
@JsonInclude(JsonInclude.Include.NON_DEFAULT)
@ConfigurationProperties("local.user")
public class User {
    private String name;
    private int age;
    private int id;
    private String sex;
    private String love;
}
```
硬编码形式
```
    @GetMapping(path = "/user")
    public String test() throws Exception{
        ObjectMapper mapper = new ObjectMapper();
        mapper.setSerializationInclusion(JsonInclude.Include.NON_DEFAULT);
        String result =  mapper.writeValueAsString(user);
        return result;
    }
```
Java代码全局配置
```
@Configuration
public class JacksonConfig {

    @Bean
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_DEFAULT);
        return objectMapper;
    }
}
```
或者使用 `SerializerProvider` 进行配置，代码示例只是对`setNullValueSerializer` 进行了配置
```
@Configuration
public class JacksonConfig {

    @Bean
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        objectMapper.getSerializerProvider().setNullValueSerializer(new NullValue());
        return objectMapper;
    }

    class NullValue extends JsonSerializer<Object> {
        @Override
        public void serialize(Object value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeString("默认值");
        }
    }
}
```
```
{
name: "默认值",
age: 0,
id: 1,
sex: "boy",
love: "love"
}
```
### 枚举值转化
举个例子
```
public enum UserEnum {
    A(18,"men"), B(20,"women");
    @Setter
    @Getter
    private int age;
    @Setter
    @Getter
    private String sex;
    UserEnum(int age, String sex) {
        this.age = age;
        this.sex = sex;
    }
}
```
```
    @GetMapping(path = "/enum")
    public UserEnum test() throws Exception{
        UserEnum userEnum = UserEnum.A;
        return userEnum;
    }
```
输入请求`URL`，返回
```
"A"
```
想得到它的`value`咋办，在对应的属性上增加`@JsonValue`注解，访问`URL`返回`"men"`
```
public enum UserEnum {
    A(18,"men"), B(20,"women");
    @Setter
    @Getter
    private int age;
    @Setter
    @Getter
    @JsonValue
    private String sex;
    UserEnum(int age, String sex) {
        this.age = age;
        this.sex = sex;
    }
}
```
返回整个枚举信息`json`，在类上加上`@JsonFormat(shape = JsonFormat.Shape.OBJECT)`注解即可
```
@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum UserEnum {...}
```
### 属性及对象不转化
假设 `User` 有个其它对象属性`other`，不想它转化`JSON`
```
@Data
@Component
@ConfigurationProperties("local.user")
public class User {
    private String name;
    private int age;
    private int id;
    private String sex;
    private String love;
    private Other other;
}
```
只有在类上加上`@JsonIgnoreType`注解
```
@Data
@JsonIgnoreType
public class Other {
    private String createdBy;
    private String lastUpdatedBy;
}
```
运行结果对比：
```
{
name: null,
age: 0,
id: 1,
sex: "boy",
love: "love",
superUser: null
}

{
name: null,
age: 0,
id: 1,
sex: "boy",
love: "love"
}
```
属性或者方法添加`@JsonIgnore`注解，则对应的属性不转化`JSON`，类上添加`@JsonIgnoreProperties`注解，并且要指定哪个属性，如果不指定是不起作用的，如下：
```
@Data
@JsonIgnoreProperties(value = {"createdBy"})
@Component
public class Other {
    @Value("${local.other.createdBy}")
    private String createdBy;
    @Value("${local.other.lastUpdatedBy}")
    private String lastUpdatedBy;
}
```
```
@Data
@Component
@ConfigurationProperties("local.user")
public class User {
    private String name;
    private int age;
    private int id;
    private String sex;
    private String love;
    @Autowired
    private Other other;
}
```
配置文件中添加
```
local.other.createdBy=2021
local.other.lastUpdatedBy=2021
```
```
{
name: null,
age: 0,
id: 1,
sex: "boy",
love: "love",
superUser: {
lastUpdatedBy: "2021"
}
}
```
[了解更多](https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations)