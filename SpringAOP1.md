

Spring AOP 是 Spring 中除了依赖注入外（DI）最为核心的功能，顾名思义，AOP 即 Aspect Oriented Programming，翻译为面向切面编程。

而 Spring AOP 则利用 CGlib 和 JDK 动态代理等方式来实现运行期动态方法增强，其目的是将与业务无关的代码单独抽离出来，使其逻辑不再与业务代码耦合，从而降低系统的耦合性，提高程序的可重用性和开发效率。因而 AOP 便成为了日志记录、监控管理、性能统计、异常处理、权限管理、统一认证等各个方面被广泛使用的技术。

追根溯源，我们之所以能无感知地在容器对象方法前后任意添加代码片段，那是由于 Spring 在运行期帮我们把切面中的代码逻辑动态“织入”到了容器对象方法内，所以说 AOP 本质上就是一个代理模式。然而在使用这种代理模式时，我们常常会用不好，那么这节课我们就来解析下有哪些常见的问题，以及背后的原理是什么。

案例 1：this 调用的当前类方法无法被拦截

假设我们正在开发一个宿舍管理系统，这个模块包含一个负责电费充值的类 ElectricService，它含有一个充电方法 charge()：
```java
@Service
public class ElectricService {

    public void charge() throws Exception {
        System.out.println("Electric charging ...");
        this.pay();
    }

    public void pay() throws Exception {
        System.out.println("Pay with alipay ...");
        Thread.sleep(1000);
    }

}
```
在这个电费充值方法 charge() 中，我们会使用支付宝进行充值。因此在这个方法中，我加入了 pay() 方法。为了模拟 pay() 方法调用耗时，代码执行了休眠 1 秒，并在 charge() 方法里使用 this.pay() 的方式调用这种支付方法。但是因为支付宝支付是第三方接口，我们需要记录下接口调用时间。这时候我们就引入了一个 @Around 的增强 ，分别记录在 pay() 方法执行前后的时间，并计算出执行 pay() 方法的耗时。
```java
@Aspect
@Service
@Slf4j
public class AopConfig {
    @Around("execution(* com.spring.puzzle.class5.example1.ElectricService.pay()) ")
    public void recordPayPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        joinPoint.proceed();
        long end = System.currentTimeMillis();
        System.out.println("Pay method time cost（ms）: " + (end - start));
    }
}
```
最后我们再通过定义一个 Controller 来提供电费充值接口，定义如下：
```java
@RestController
public class HelloWorldController {
    @Autowired
    ElectricService electricService;
    @RequestMapping(path = "charge", method = RequestMethod.GET)
    public void charge() throws Exception{
          electricService.charge();
    };
}
```
完成代码后，我们访问上述接口，会发现这段计算时间的切面并没有执行到，输出日志如下：
```
Electric charging ...
Pay with alipay ...
```
回溯之前的代码可知，在 @Around 的切面类中，我们很清晰地定义了切面对应的方法，但是却没有被执行到。这说明了在类的内部，通过 this 方式调用的方法，是没有被 Spring AOP 增强的。这是为什么呢？我们来分析一下。
#### 案例解析
我们可以从源码中找到真相。首先来设置个断点，调试看看 this 对应的对象是什么样的：
![](/1.png)
可以看到，this 对应的就是一个普通的 ElectricService 对象，并没有什么特别的地方。再看看在 Controller 层中自动装配的 ElectricService 对象是什么样：
![](/2.png)
可以看到，这是一个被 Spring 增强过的 Bean，所以执行 charge() 方法时，会执行记录接口调用时间的增强操作。而 this 对应的对象只是一个普通的对象，并没有做任何额外的增强。

为什么 this 引用的对象只是一个普通对象呢？这还要从 Spring AOP 增强对象的过程来看。但在此之前，有些基础我需要在这里强调下。

> Spring AOP 的实现

Spring AOP 的底层是动态代理。而创建代理的方式有两种，JDK 的方式和 CGLIB 的方式。JDK 动态代理只能对实现了接口的类生成代理，而不能针对普通类。而 CGLIB 是可以针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，来实现代理对象。具体区别可参考下图：
![](/3.png)


> 如何使用 Spring AOP

在 Spring Boot 中，我们一般只要添加以下依赖就可以直接使用 AOP 功能：

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

而对于非 Spring Boot 程序，除了添加相关 AOP 依赖项外，我们还常常会使用 @EnableAspectJAutoProxy 来开启 AOP 功能。这个注解类引入（Import）AspectJAutoProxyRegistrar，它通过实现 ImportBeanDefinitionRegistrar 的接口方法来完成 AOP 相关 Bean 的准备工作。

补充完最基本的 Spring 底层知识和使用知识后，我们具体看下创建代理对象的过程。先来看下调用栈：
![](/4.png)

创建代理对象的时机就是创建一个 Bean 的时候，而创建的的关键工作其实是由 AnnotationAwareAspectJAutoProxyCreator 完成的。它本质上是一种 BeanPostProcessor。所以它的执行是在完成原始 Bean 构建后的初始化 Bean（initializeBean）过程中。而它到底完成了什么工作呢？我们可以看下它的 postProcessAfterInitialization 方法：

``` java
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
   if (bean != null) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      if (this.earlyProxyReferences.remove(cacheKey) != bean) {
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   return bean;
}
```

上述代码中的关键方法是 wrapIfNecessary，顾名思义，在需要使用 AOP 时，它会把创建的原始的 Bean 对象 wrap 成代理对象作为 Bean 返回。具体到这个 wrap 过程，可参考下面的关键代码行：
``` java

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
   // 省略非关键代码
   Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
   if (specificInterceptors != DO_NOT_PROXY) {
      this.advisedBeans.put(cacheKey, Boolean.TRUE);
      Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
      this.proxyTypes.put(cacheKey, proxy.getClass());
      return proxy;
   }
   // 省略非关键代码 
}

```
上述代码中，第 6 行的 createProxy 调用是创建代理对象的关键。具体到执行过程，它首先会创建一个代理工厂，然后将通知器（advisors）、被代理对象等信息加入到代理工厂，最后通过这个代理工厂来获取代理对象。一些关键过程参考下面的方法：
```
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
      @Nullable Object[] specificInterceptors, TargetSource targetSource) {
  // 省略非关键代码
  ProxyFactory proxyFactory = new ProxyFactory();
  if (!proxyFactory.isProxyTargetClass()) {
   if (shouldProxyTargetClass(beanClass, beanName)) {
      proxyFactory.setProxyTargetClass(true);
   }
   else {
      evaluateProxyInterfaces(beanClass, proxyFactory);
   }
  }
  Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
  proxyFactory.addAdvisors(advisors);
  proxyFactory.setTargetSource(targetSource);
  customizeProxyFactory(proxyFactory);
   // 省略非关键代码
  return proxyFactory.getProxy(getProxyClassLoader());
}

```
经过这样一个过程，一个代理对象就被创建出来了。我们从 Spring 中获取到的对象都是这个代理对象，所以具有 AOP 功能。而之前直接使用 this 引用到的只是一个普通对象，自然也就没办法实现 AOP 的功能了。

#### 问题修正
从上述案例解析中，我们知道，只有引用的是被动态代理创建出来的对象，才会被 Spring 增强，具备 AOP 该有的功能。那什么样的对象具备这样的条件呢？有两种。一种是被 @Autowired 注解的，于是我们的代码可以改成这样，即通过 @Autowired 的方式，在类的内部，自己引用自己：

``` java
@Service
public class ElectricService {
    @Autowired
    ElectricService electricService;
    public void charge() throws Exception {
        System.out.println("Electric charging ...");
        //this.pay();
        electricService.pay();
    }
    public void pay() throws Exception {
        System.out.println("Pay with alipay ...");
        Thread.sleep(1000);
    }
}
```
另一种方法就是直接从 AopContext 获取当前的 Proxy。那你可能会问了，AopContext 是什么？简单说，它的核心就是通过一个 ThreadLocal 来将 Proxy 和线程绑定起来，这样就可以随时拿出当前线程绑定的 Proxy。不过使用这种方法有个小前提，就是需要在 @EnableAspectJAutoProxy 里加一个配置项 exposeProxy = true，表示将代理对象放入到 ThreadLocal，这样才可以直接通过 AopContext.currentProxy() 的方式获取到，否则会报错如下：

![](/5.png)

按这个思路，我们修改下相关代码：
``` java
import org.springframework.aop.framework.AopContext;
import org.springframework.stereotype.Service;
@Service
public class ElectricService {
    public void charge() throws Exception {
        System.out.println("Electric charging ...");
        ElectricService electric = ((ElectricService) AopContext.currentProxy());
        electric.pay();
    }
    public void pay() throws Exception {
        System.out.println("Pay with alipay ...");
        Thread.sleep(1000);
    }
}
```
同时，不要忘记修改 EnableAspectJAutoProxy 注解的 exposeProxy 属性，示例如下：
``` java
@SpringBootApplication
@EnableAspectJAutoProxy(exposeProxy = true)
public class Application {
    // 省略非关键代码
}
```
这两种方法的效果其实是一样的，最终我们打印出了期待的日志，到这，问题顺利解决了。
```
Electric charging ...
Pay with alipay ...
Pay method time cost(ms): 1005
```