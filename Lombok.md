注解 | 说明 
 :- | :-
@Data |  用在类上，提供`Getter、Setter、equals、hasCode、toString` 、构造方法（参数：`@NonNull和final字段`）
@AllArgsConstructor |  用在类上，提供全参数构造方法
@NoArgsConstructor |  用在类上，提供无参构造方法
@RequiredArgsConstructor|  用在类上，提供构造方法（参数：`@NonNull和final字段`）
@Value |  用在类上，提供 `get、equals、hashCode、toString`、全参数构造方法
@EqualsAndHashCode |  用在类上，提供`equals、hasCode`方法，继承父类使用 `@EqualsAndHashCode(callSuper = true)`
@NonNull |  用在字段上，提供无参构造方法，为空则抛出`NullPointerException`
@Cleanup |  用在字段上，释放资源，`try{}finally{}` 
@SneakyThrows |  用在类上，捕获异常，指定异常`@SneakyThrows(Exception.class)` 

> 注解 `@Getter @Setter`，`@Getter` 支持懒加载 

```java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Getter {
	lombok.AccessLevel value() default lombok.AccessLevel.PUBLIC;
	AnyAnnotation[] onMethod() default {};
	boolean lazy() default false;
}
```
```java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Setter {
	lombok.AccessLevel value() default lombok.AccessLevel.PUBLIC;
	AnyAnnotation[] onMethod() default {};
	AnyAnnotation[] onParam() default {};
}
```
AccessLevel	  | 描述
 :----                 | :-
PUBLIC        |	生成 `public` 修饰的 `getter    setter` 方法
MODULE	| 生成没有修饰符修饰的 `getter  setter` 方法
PROTECTED|	生成 `protected` 修饰的 `getter  setter` 方法
PACKAGE|	生成没有修饰符修饰的 `getter  setter` 方法
PRIVATE	|生成 `private` 修饰的 `getter  setter` 方法
NONE	|不生成 `getter  setter` 方法

> 注解  `@EqualsAndHashCode`
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface EqualsAndHashCode {
	String[] exclude() default {};
	String[] of() default {};
	boolean callSuper() default false;
	boolean doNotUseGetters() default false;
}
```
属性| 描述
 :--- | :-
exclude	 |在生成的 `equals` 和 `hashCode` 方法中排除的字段
of    |	在生成的 `equals` 和 `hashCode` 方法中列出的字段
callSuper  	|属性设置为 `true`，表示父类的 `equals` 和 `hashCode` 参与计算，默认为 `false`
doNotUseGetters  |	通常都是通过字段的 `getter` 方法获取字段值，设置为 `true`，表示不通过 `getter` 方法获取，而是直接访问字段值，默认为 `false`

> 注解 `@Accessors` ，配合 `@Setter、@Getter 、@Data `注解，用在类上和字段上

 属性	|描述
 :-- | :-
   fluent   |	默认为 `false`，如果为 `true` 则生成的 `getter/setter` 方法没有 `set/get` 前缀，如果为 `true`, `chain` 未设置，则 `chain` 会被设置为 `true`
chain   |	默认为 `false`， `setter` 方法返回是 `void`，如果为 `true` 则返回` this`，当 `fluent` 为 `true` 时，`chain` 会设置为 `true`
prefix |	 `getter    setter `方法时会去掉指定的前缀
> 注解 `Log`

 注解	|描述
 :-- | :--
@CommonsLog| `org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class)`
@JBossLog| `org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class)`
@Log| `java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName())`
@Log4j| `org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class)`
@Log4j2| `org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class)`
@Slf4j| `org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class)`
@XSlf4j| `org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class)`

详情见 [使用 Lombok 进行 Java 开发](https://blog.csdn.net/wb1046329430/article/details/106105767/)
使用 `@Data` 会遇到什么坑，看下 `Mybatis` 的 `PropertyNamer` 类 ，`methodToProperty` 方法转属性
```java
  public static String methodToProperty(String name) {
     // is开头的一般是bool类型，直接从第二个(索引)开始截取
    if (name.startsWith("is")) {
      name = name.substring(2);
    // set get的就从第三个(索引)开始截取
    } else if (name.startsWith("get") || name.startsWith("set")) {
      name = name.substring(3);
    }
    return name;
  }
```
测试
```java
@Data
public class Test {
    private boolean isTrue;
    private String mName;
    private Boolean isFalse;

    public static void main(String[] args) {
        Test test = new Test();
        test.getMName(); // mName 属性的 get 方法
        test.isTrue(); // isTrue 属性的  get 方法

        System.out.println(PropertyNamer.methodToProperty("getMName")); // 打印 MName 属性 (错误)
        System.out.println(PropertyNamer.methodToProperty("isTrue")); // 打印 true 属性 (错误)

        test.getIsFalse();
        System.out.println(PropertyNamer.methodToProperty("getIsFalse")); // 打印 isFalse 属性  (正确)
    }
}
```
可以看出 `Mybatis` 把 `@Data` 生成的方法转属性之后，跟原属性对不上，所以使用的时候， `boolean` 类型 不要 `is加大写字母`命名，其它基本类型属性不要 `第一个字母小写 第二个字母大写` 命名
如果类实现 `Serializable`接口，  `boolean` 类型的字段命令为  `isTrue` 的会有问题 ，字段序列号后是 `true`

如果要 `Builder` 链式调用父类的字段 ，父类和子类都要加 `@SuperBuilder`
 `Coffee.builder().name("name").id(100L).build();`
```java
@SuperBuilder
@Data
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class Coffee extends BaseEntity implements Serializable {
    @NotBlank(message = "name can not be null") //只能作用在 String 上,不能为null, 而且调用trim()后, 长度必须大于0
    private String name;
}
```

```java
@SuperBuilder
@AllArgsConstructor
@NoArgsConstructor
@Setter
@Getter
public class BaseEntity implements Serializable {
    private long id;
}
```
