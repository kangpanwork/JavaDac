#### 装饰者模式
装饰者模式拥有一个设计非常巧妙的结构，它可以动态添加对象功能。在基本的设计原则中，有一条重要的设计准则叫做`合成/聚合复用原则`。根据该原则的思想，代码复用应该尽可能使用委托，而不是使用继承。因为继承是一种紧密耦合，任何父类的改动都会影响其子类，不利于系统维护。而委托则是松散耦合，只要接口不变，委托类的改动并不会影响其上层对象。

装饰者模式就充分运用了这种思想，通过委托机制，复用系统中的各个组件，在运行时，可以将这些功能组件进行叠加，从而构造一个超级对象，使其拥有所有这些组件的功能。而各个子功能模块，被很好地维护在各个组件的相关类中，拥有整洁的系统结构。

装饰者模式可以有效分离组件，从而提示模块的可维护性并增加模块的复用性。

装饰者和被装饰者拥有相同的接口 `ICore`， 被装饰者通常是系统的核心组件 `CoreComponent` 。而装饰者可以在被装饰者的方法前后，加上前置处理和后置处理，增强装饰者的功能。具体参考我的代码：
```
public class Main {
    public static void main(String[] args) {
	// write your code here
        ICore component = new BeforeCore(new AfterCore(new CoreComponent()));
        component.handler();
    }
}
```
打印
```
前置处理...
核心组件处理
后置处理...
```
组件接口，装饰者和被装饰者的接口，它定义了被装饰者的核心功能和装饰者需要加强的功能点。
```
public interface ICore {
    void handler();
}
```
核心组件，被装饰者的对象
```
/**
 * 核心组件
 */
public class CoreComponent implements ICore{
    @Override
    public void handler() {
        System.out.println("核心组件处理");
    }
}
```
```
/**
 * AbstractCoreComponent 维护核心组件 component 对象
 * 它负责告知其子类，其核心业务逻辑应该全权委托 component 完成
 * 自己仅仅是做增强处理
 */
public abstract class AbstractCoreComponent implements ICore{
    ICore component;
    public AbstractCoreComponent(ICore component) {
        this.component = component;
    }
}
```
```
/**
 * 具体的装饰器
 * 它负责前置处理
 * 委托了具体组件 component 进行核心处理
 */
public class BeforeCore extends AbstractCoreComponent{

    public BeforeCore(ICore component) {
        super(component);
    }

    @Override
    public void handler() {
        System.out.println("前置处理...");
        component.handler();
    }
}
```
```
/**
 * 具体装饰器
 * 功能和前置处理类似
 */
public class AfterCore extends AbstractCoreComponent{
    public AfterCore(ICore component) {
        super(component);
    }

    @Override
    public void handler() {
        component.handler();
        System.out.println("后置处理...");
    }
}
```
装饰者模式的核心思想在于：无需将所有的逻辑，比如核心处理，前置处理，后置处理功能模块粘合在一起实现。通过装饰者模式，可以将它们分解完全独立的组件，并且使用时灵活地进行装配，从而构造一个功能强大的组件对象。` ICore component = new BeforeCore(new AfterCore(new CoreComponent()));`