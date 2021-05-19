### 原型 Bean 被固定
明明定义的是多例`SCOPE_PROTOTYPE`，为啥每次服务请求打印的地址都是同一个呢
```java
@Service
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class ServiceImpl {
}
```
浏览器输入`http://localhost:8080/test` 打印的都是同一个地址 `service:demo.generator.ServiceImpl@3501ee32`
```java
@RestController
public class Controller {
    @Autowired
    private ServiceImpl service;

    @RequestMapping(path = "test",method = RequestMethod.GET)
    public String test() {
        return "service:" + service;
    }
}
```
### 分析
当一个属性成员 `service` 声明为 `@Autowired` 后，那么在创建 `Controller` 这个 `Bean` 时，会先使用构造器反射出实例，然后来装配各个标记为 `@Autowired` 的属性成员（装配方法参考 `AbstractAutowireCapableBeanFactory #populateBean`）。具体到执行过程，它会使用很多 `BeanPostProcessor` 来做完成工作，其中一种是 `AutowiredAnnotationBeanPostProcessor`，它会通过 `DefaultListableBeanFactory#findAutowireCandidates` 寻找到 `ServiceImp` 类型的 `Bean`，然后设置给对应的属性（即 `service` 成员）。
关键执行步骤可参考 `AutowiredAnnotationBeanPostProcessor.AutowiredFieldElement#inject`
```java

protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
   Field field = (Field) this.member;
   Object value;
   //寻找“bean”
   if (this.cached) {
      value = resolvedCachedArgument(beanName, this.cachedFieldValue);
   }
   else {
     //省略其他非关键代码
     value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
   }
   if (value != null) {
      //将bean设置给成员字段
      ReflectionUtils.makeAccessible(field);
      field.set(bean, value);
   }
}
```
待我们寻找到要自动注入的 `Bean` 后，即可通过反射设置给对应的 `field`。这个`field` 的执行只发生了一次，所以后续就固定起来了，它并不会因为 `ServiceImpl` 标记了 `SCOPE_PROTOTYPE` 而改变。

所以，当一个单例的 `Bean`，使用 `autowired` 注解标记其属性时，（这里指单例的 `Controller` 这个 `Bean`），你一定要注意这个属性值 （这里指`service`）会被固定下来。

### 解决方案
通过上述源码分析，我们可以知道要修正这个问题，肯定是不能将 `ServiceImpl` 的 `Bean` 固定到属性上的，而应该是每次使用时都会重新获取一次。所以这里我提供了两种修正方式：
#### 自动注入 `Context`

```java
@RestController
public class Controller {

    @Autowired
    private ApplicationContext applicationContext;

    @RequestMapping(path = "test", method = RequestMethod.GET)
    public String test(){
         return  "service: " + getService();
    };
 
    public ServiceImpl getService(){
        return applicationContext.getBean(ServiceImpl.class);
    }

}
```
#### 使用 Lookup 注解
```java
@RestController
public class Controller {

    @RequestMapping(path = "test",method = RequestMethod.GET)
    public String test() {
        return "service:" + getService();
    }

    @Lookup
    public ServiceImpl getService() {
        return null;
    }
}
```
### 拓展
在修正代码中，我们看到 `getService()` 方法的实现返回值是 null，这或许很难说服自己，为啥返回`null`可以生成新的实例对象。
```java
@Lookup
public ServiceImpl getService(){
    //下面的日志会输出么？
    log.info("executing this method");
    return null;
}  
```
以上代码，添加了一行代码输出日志。测试后，我们会发现并没有日志输出。
那它是怎么实现的呢，看下`CglibSubclassingInstantiationStrategy`的内部类`LookupOverrideMethodInterceptor`的 `intercept` 方法，这里省略了代码
```java
    private final BeanFactory owner;
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy mp) throws Throwable {
            this.owner.getBean(method.getReturnType())); // 方法的返回类型
    }
```
通过 `BeanFactory` 来获取 `Bean`，`getBean(method.getReturnType())`。当使用 `Lookup` 注解一个方法时，这个方法的具体实现已并不重要，里面代码随便怎么写都可以。
### 代理
可以使用`scope`注解的`proxyMode`，设置成`target_class`，这样注入到`controller`的`bean`就是代理对象了，每次都会从`beanfactory`里面重新拿过
```java
@Service
// @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Scope(value = "prototype",proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ServiceImpl {
}
```
### Scope 注解说明
```
@Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)这个是说在每次注入的时候回自动创建一个新的bean实例

@Scope(value=ConfigurableBeanFactory.SCOPE_SINGLETON)单例模式，在整个应用中只能创建一个实例

@Scope(value=WebApplicationContext.SCOPE_GLOBAL_SESSION)全局session中的一般不常用

@Scope(value=WebApplicationContext.SCOPE_APPLICATION)在一个web应用中只创建一个实例

@Scope(value=WebApplicationContext.SCOPE_REQUEST)在一个请求中创建一个实例

@Scope(value=WebApplicationContext.SCOPE_SESSION)每次创建一个会话中创建一个实例

`Scope` 里面还有个属性

proxyMode=ScopedProxyMode.INTERFACES创建一个JDK代理模式

proxyMode=ScopedProxyMode.TARGET_CLASS基于类的代理模式

proxyMode=ScopedProxyMode.NO（默认）不进行代理
```