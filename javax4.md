#### 享元模式
享元模式是设计模式中少数几个以提高系统性能为目的的模式之一。它的核心思想是：如果在一个系统中存在多个相同的对象，那么只需要共享一份对象的引用的拷贝。而不必为每一次使用都创建新的对象。在享元模式中，由于需要构造和维护这些可以共享的对象，因此，常常会出现一个工厂类，用于维护和创建对象。
享元模式对性能提升的主要帮助有两点：
- 可以节省重复创建对象的开销，因为被享元模式维护的相同对象只会被创建一次，当创建对象比较耗时时，便可以节省大量时间。
- 由于创建对象的数量减少，所以对系统内存的需求也减小，这将使得 GC 的压力也相应地降低，进而使得系统拥有一个更健康的内存结构和更快的反应速度。

看了例子：
```
public class Main {
    public static void main(String[] args) {
        Factory factory = new Factory();
        for(int i = 0 ; i < 3; i++) {
            IDo fly = factory.getFly(1);
            System.out.println(fly);
        }
        factory.getFly(1).todo();
    }
}

```
打印如下：
```
com.company.Fly@1b6d3586
com.company.Fly@1b6d3586
com.company.Fly@1b6d3586
fly...
```
可以看出是同一个对象在使用。

享元工厂，维护相同的享元对象，保证相同的享元对象可以被系统共享。内部可使用单例模式，当请求对象存在时直接返回，不存在则创建。
```
public class Factory {
    Map<Integer,IDo> fly = new HashMap<>();
    Map<Integer,IDo> run = new HashMap<>();

    public IDo getFly(int id) {
        IDo flyDo = fly.get(id);
        if (null == flyDo) {
            flyDo = new Fly(id);
            fly.put(id,flyDo);
        }
        return flyDo;
    }

    public IDo getRun(int id) {
        IDo runDo = run.get(id);
        if (null == runDo) {
            runDo = new Run(id);
            run.put(id,runDo);
        }
        return runDo;
    }
}
```
抽象享元，也就是接口，抽象出来的某些业务逻辑
```
public interface IDo {
    void todo();
}
```
具体享元类，接口的实现类
```
public class Fly implements IDo{

    private int id;

    public Fly(int id) {
        this.id = id;
    }
    @Override
    public void todo() {
        System.out.println("fly...");
    }
}
```
```
public class Run implements IDo {

    private int id;

    public Run(int id) {
        this.id = id;
    }

    @Override
    public void todo() {
        System.out.println("run...");
    }
}
```