#### 观察者模式

观察者模式是非常常用的一种设计模式，在软件系统中，当一个对象的行为依赖于另一个对象的状态时，观察者模式就相当有用。若不使用观察者模式提供的通用结构，而需要实现其类似的功能，则只能在另一个线程中不停监听对象所依赖的状态。在一个复杂系统中，可能会因此开启很多线程来实现这一功能，这将使系统的性能产生额外的负担。观察者模式的意义也就在此，它可以在单线程中，使某一个对象，及时得知自身所依赖的状态的变化。

```
public class Test {
    public static void main(String[] args) {
        // 被观察者
        Subject subject = new Subject();
        // 观察者
        Observer observer = new Observer();
        // 添加观察者
        subject.attach(observer);
        // 通知观察者
        subject.inform();
    }
}
```
观察者模式的经典结构参考如下代码：
```
/**
 * 被观察者
 */
public interface ISubject {
    /**
     * 添加观察者
     * @param observer
     */
    void attach(IObserver observer);

    /**
     * 删除观察者
     * @param observer
     */
    void detach(IObserver observer);

    /**
     * 通知所有观察者
     */
    void inform();
}
```
```
/**
 * 被观察者实例
 */
public class Subject implements ISubject{

    Vector<IObserver> observers = new Vector<>();

    @Override
    public void attach(IObserver observer) {
        observers.addElement(observer);
    }

    @Override
    public void detach(IObserver observer) {
        observers.removeElement(observer);
    }

    @Override
    public void inform() {
        Event event = new Event();
        event.setMessage("message...");
        for(IObserver observer : observers) {
            observer.update(event);
        }
    }
}
```
```
/**
 * 观察者
 */
public interface IObserver {
    /**
     * 当依赖状态发生改变时调用
     * @param event
     */
    void update(Event event);
}
```
```
/**
 * 观察者实例
 */
public class Observer implements IObserver {
    @Override
    public void update(Event event) {
        System.out.println(event.getMessage());
    }
}
```
```
/**
 * 事件
 */
public class Event {
    /**
     * 事件消息
     */
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

```
在 java.util.Observable 类中，已经实现了主要的功能，如添加，删除，通知观察者，开发人员可以直接继承 Observable 使用这些功能。它的 update() 方法会在 Observable 的 notifyObservers() 方法中被回调，以获得最新的状态变化。