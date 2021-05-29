![](/1.gif)

> 本文代码地址 https://github.com/kangpanwork/aopErrorDemo 

##### 案例 2：直接访问被拦截类的属性抛空指针异常

接上一个案例，在宿舍管理系统中，我们使用了 charge() 方法进行支付。在统一结算的时候我们会用到一个管理员用户付款编号，这时候就用到了几个新的类。

User 类，包含用户的付款编号信息：
```
public class User {
    private String payNum;
    public User(String payNum) {
        this.payNum = payNum;
    }
    public String getPayNum() {
        return payNum;
    }
    public void setPayNum(String payNum) {
        this.payNum = payNum;
    }
}
```
AdminUserService 类，包含一个管理员用户（User），其付款编号为 202101166；另外，这个服务类有一个 login() 方法，用来登录系统。
```

@Service
public class AdminUserService {
    public final User adminUser = new User("202101166");
    
    public void login() {
        System.out.println("admin user login...");
    }
}
```
我们需要修改 ElectricService 类实现这个需求：在电费充值时，需要管理员登录并使用其编号进行结算。完整代码如下：
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
@Service
public class ElectricService {
    @Autowired
    private AdminUserService adminUserService;
    public void charge() throws Exception {
        System.out.println("Electric charging ...");
        this.pay();
    }

    public void pay() throws Exception {
        adminUserService.login();
        String payNum = adminUserService.adminUser.getPayNum();
        System.out.println("User pay num : " + payNum);
        System.out.println("Pay with alipay ...");
        Thread.sleep(1000);
    }
}
```
代码完成后，执行 charge() 操作，一切正常：
```

Electric charging ...
admin user login...
User pay num : 202101166
Pay with alipay ...
```
这时候，由于安全需要，就需要管理员在登录时，记录一行日志以便于以后审计管理员操作。所以我们添加一个 AOP 相关配置类，具体如下：
```

@Aspect
@Service
@Slf4j
public class AopConfig {
    @Before("execution(* com.spring.puzzle.class5.example2.AdminUserService.login(..)) ")
    public void logAdminLogin(JoinPoint pjp) throws Throwable {
        System.out.println("! admin login ...");
    }
}
```
添加这段代码后，我们执行 charge() 操作，发现不仅没有相关日志，而且在执行下面这一行代码的时候直接抛出了 NullPointerException：`String payNum = dminUserService.user.getPayNum();`
本来一切正常的代码，因为引入了一个 AOP 切面，抛出了 NullPointerException。这会是什么原因呢？我们先 debug 一下，来看看加入 AOP 后调用的对象是什么样子。
![](/6.png)
可以看出，加入 AOP 后，我们的对象已经是一个代理对象了，如果你眼尖的话，就会发现在上图中，属性 adminUser 确实为 null。为什么会这样？为了解答这个诡异的问题，我们需要进一步理解 Spring 使用 CGLIB 生成 Proxy 的原理。
#### 案例解析
我们在上一个案例中解析了创建 Spring Proxy 的大体过程，在这里，我们需要进一步研究一下通过 Proxy 创建出来的是一个什么样的对象。正常情况下，AdminUserService 只是一个普通的对象，而 AOP 增强过的则是一个 `AdminUserService $$EnhancerBySpringCGLIB$$xxxx`。

这个类实际上是 AdminUserService 的一个子类。它会 overwrite 所有 public 和 protected 方法，并在内部将调用委托给原始的 AdminUserService 实例。

从具体实现角度看，CGLIB 中 AOP 的实现是基于 org.springframework.cglib.proxy 包中 Enhancer 和 MethodInterceptor 两个接口来实现的。

#### 整个过程，我们可以概括为三个步骤：

- 定义自定义的 MethodInterceptor 负责委托方法执行；
- 创建 Enhance 并设置 Callback 为上述 MethodInterceptor；
- enhancer.create() 创建代理。

接下来，我们来具体分析一下 Spring 的相关实现源码。

在上个案例分析里，我们简要提及了 Spring 的动态代理对象的初始化机制。在得到 Advisors 之后，会通过 ProxyFactory.getProxy 获取代理对象：
```

public Object getProxy(ClassLoader classLoader) {
  return createAopProxy().getProxy(classLoader);
}
```

在这里，我们以 CGLIB 的 Proxy 的实现类 CglibAopProxy 为例，来看看具体的流程：
```
public Object getProxy(@Nullable ClassLoader classLoader) {
    // 省略非关键代码
    // 创建及配置 Enhancer
    Enhancer enhancer = createEnhancer();
    // 省略非关键代码
    // 获取Callback：包含DynamicAdvisedInterceptor，亦是MethodInterceptor
    Callback[] callbacks = getCallbacks(rootClass);
    // 省略非关键代码
    // 生成代理对象并创建代理（设置 enhancer 的 callback 值）
    return createProxyClassAndInstance(enhancer, callbacks);
    // 省略非关键代码
}
```
上述代码中的几个关键步骤大体符合之前提及的三个步骤，其中最后一步一般都会执行到 CglibAopProxy 子类 ObjenesisCglibAopProxy 的 createProxyClassAndInstance() 方法：
```
protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
   //创建代理类Class
   Class<?> proxyClass = enhancer.createClass();
   Object proxyInstance = null;
   //spring.objenesis.ignore默认为false
   //所以objenesis.isWorthTrying()一般为true
   if (objenesis.isWorthTrying()) {
      try {
         // 创建实例
         proxyInstance = objenesis.newInstance(proxyClass, enhancer.getUseCache());
      }
      catch (Throwable ex) {
          // 省略非关键代码
      }
   }
       
    if (proxyInstance == null) {
       // 尝试普通反射方式创建实例
       try {
          Constructor<?> ctor = (this.constructorArgs != null ?
                proxyClass.getDeclaredConstructor(this.constructorArgTypes) :
                proxyClass.getDeclaredConstructor());
          ReflectionUtils.makeAccessible(ctor);
          proxyInstance = (this.constructorArgs != null ?
                ctor.newInstance(this.constructorArgs) : ctor.newInstance());
      //省略非关键代码
       }
    }
   // 省略非关键代码
   ((Factory) proxyInstance).setCallbacks(callbacks);
   return proxyInstance;
}
```

这里我们可以了解到，Spring 会默认尝试使用 objenesis 方式实例化对象，如果失败则再次尝试使用常规方式实例化对象。现在，我们可以进一步查看 objenesis 方式实例化对象的流程。
![](/7.png)

参照上述截图所示调用栈，objenesis 方式最后使用了 JDK 的 ReflectionFactory.newConstructorForSerialization() 完成了代理对象的实例化。而如果你稍微研究下这个方法，你会惊讶地发现，这种方式创建出来的对象是不会初始化类成员变量的。

所以说到这里，聪明的你可能已经觉察到真相已经暴露了，我们这个案例的核心是代理类实例的默认构建方式很特别。在这里，我们可以总结和对比下通过反射来实例化对象的方式，包括：

- java.lang.Class.newInsance()
- java.lang.reflect.Constructor.newInstance()
- sun.reflect.ReflectionFactory.newConstructorForSerialization().newInstance()

前两种初始化方式都会同时初始化类成员变量，但是最后一种通过 ReflectionFactory.newConstructorForSerialization().newInstance() 实例化类则不会初始化类成员变量，这就是当前问题的最终答案了。

#### 问题修正
了解了问题的根本原因后，修正起来也就不困难了。既然是无法直接访问被拦截类的成员变量，那我们就换个方式，在 UserService 里写个 getUser() 方法，从内部访问获取变量。我们在 AdminUserService 里加了个 getUser() 方法：
```

public User getUser() {
    return user;
}
```

在 ElectricService 里通过 getUser() 获取 User 对象：
原来出错的方式：`String payNum = = adminUserService.adminUser.getPayNum();`
修改后的方式：`String payNum = adminUserService.getAdminUser().getPayNum();`

运行下来，一切正常，可以看到管理员登录日志了：
```
Electric charging ...
! admin login ...
admin user login...
User pay num : 202101166
Pay with alipay ...
```

但你有没有产生另一个困惑呢？既然代理类的类属性不会被初始化，那为什么可以通过在 AdminUserService 里写个 getUser() 方法来获取代理类实例的属性呢？我们再次回顾 createProxyClassAndInstance 的代码逻辑，创建代理类后，我们会调用 setCallbacks 来设置拦截后需要注入的代码：
```
protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
   Class<?> proxyClass = enhancer.createClass();
   Object proxyInstance = null;
   if (objenesis.isWorthTrying()) {
      try {
         proxyInstance = objenesis.newInstance(proxyClass, enhancer.getUseCache());
      }
   // 省略非关键代码
   ((Factory) proxyInstance).setCallbacks(callbacks);
   return proxyInstance;
}
```
通过代码调试和分析，我们可以得知上述的 callbacks 中会存在一种服务于 AOP 的 DynamicAdvisedInterceptor，它的接口是 MethodInterceptor（callback 的子接口），实现了拦截方法 intercept()。我们可以看下它是如何实现这个方法的：
```
public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
   // 省略非关键代码
    TargetSource targetSource = this.advised.getTargetSource();
    // 省略非关键代码 
      if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
         Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
         retVal = methodProxy.invoke(target, argsToUse);
      }
      else {
         // We need to create a method invocation...
         retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
      }
      retVal = processReturnType(proxy, target, method, retVal);
      return retVal;
   }
   //省略非关键代码
}
```
当代理类方法被调用，会被 Spring 拦截，从而进入此 intercept()，并在此方法中获取被代理的原始对象。而在原始对象中，类属性是被实例化过且存在的。因此代理类是可以通过方法拦截获取被代理对象实例的属性。说到这里，我们已经解决了问题。但如果你看得仔细，就会发现，其实你改变一个属性，也可以让产生的代理对象的属性值不为 null。例如修改启动参数 spring.objenesis.ignore 如下：
![](/8.png)
此时再调试程序，你会发现 adminUser 已经不为 null 了：
![](/9.png)
所以这也是解决这个问题的一种方法，相信聪明的你已经能从前文贴出的代码中找出它能够工作起来的原理了。
#### 重点回顾
通过以上两个案例的介绍，相信你对 Spring AOP 动态代理的初始化机制已经有了进一步的了解，这里总结重点如下：使用 AOP，实际上就是让 Spring 自动为我们创建一个 Proxy，使得调用者能无感知地调用指定方法。而 Spring 有助于我们在运行期里动态织入其它逻辑，因此，AOP 本质上就是一个动态代理。我们只有访问这些代理对象的方法，才能获得 AOP 实现的功能，所以通过 this 引用是无法正确使用 AOP 功能的。在不能改变代码结果前提下，我们可以通过 @Autowired、AopContext.currentProxy() 等方式获取相应的代理对象来实现所需的功能。我们一般不能直接从代理类中去拿被代理类的属性，这是因为除非我们显示设置 spring.objenesis.ignore 为 true，否则代理类的属性是不会被 Spring 初始化的，我们可以通过在被代理类中增加一个方法来间接获取其属性。