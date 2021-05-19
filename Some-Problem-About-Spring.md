看下 `org.springframework.util` 包下的 `ClassUtils`
```java
public abstract class ClassUtils {
	private static final char PACKAGE_SEPARATOR = '.';
}
public static String getPackageName(String fqClassName) {
    Assert.notNull(fqClassName, "Class name must not be null");
    int lastDotIndex = fqClassName.lastIndexOf(PACKAGE_SEPARATOR);
    return (lastDotIndex != -1 ? fqClassName.substring(0, lastDotIndex) : "");
}
```
如果 `fqClassName` 参数是 `com.java` 那么返回结果为 `com` ，这里解答了我一个疑问，比如 `Spring` 启动类所在的包 和 `Controller`包 不在同一个包下，导致找不到 `Controller` 类。这里写下正常的包结构
- com
    - controller
    - model
    - service
    - mapper
    - Application 启动类

如果 `Application` 启动类 在其它包下，例如放在 `app` 包下，那么启动会报错，找不到对应的 `Bean`
- com
    - controller
    - model
    - service
    - mapper
- app
    - Application 启动类


那为啥在同一个包下就可以找到呢
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```
来看下 `@SpringBootApplication` 注解，这里省略了其它代码
```java
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
public @interface SpringBootApplication {}
```
`@SpringBootApplication` 开启了很多功能，其中一个关键功能就是 `@ComponentScan`
```java
public @interface ComponentScan {
	@AliasFor("basePackages")
	String[] value() default {};
}
```
这里 `basePackages` 默认是 `{}`，那么默认是什么包呢，先看下 `ConfigurationClassParser` 类的其中一段代码
```java
private final ComponentScanAnnotationParser componentScanParser;
// 拿到该类上面所有的@ComponentScan注解
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
                
for (AnnotationAttributes componentScan : componentScans) {

    Set<BeanDefinitionHolder> scannedBeanDefinitions =
            this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());

}              
```
看下  `this.componentScanParser.parse()`, 这里涉及 `ComponentScanAnnotationParser` 类，以下省略了其它代码，看下它的 `parse` 方法，`declaringClass` 默认传的 `sourceClass.getMetadata().getClassName()` 也就是所在的包名
```java
class ComponentScanAnnotationParser {
        public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
        Set<String> basePackages = new LinkedHashSet<>();
		String[] basePackagesArray = componentScan.getStringArray("basePackages");
        	if (basePackages.isEmpty()) {
			basePackages.add(ClassUtils.getPackageName(declaringClass));
		}
    }
}
```
看到这应该明白了。