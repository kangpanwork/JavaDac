### 设计优化
![](/10.png)
![](/11.png)
#### 善用设计模式
设计模式是前人工作的总结和提炼，通常，被人们广泛流传的设计模式都是对某一特定问题的成熟的解决方案。如果能合理地使用设计模式，不仅能使系统更容易被他人理解，同时也能使系统拥有更加合理的结构，本节总结归纳了一些经典的设计模式，并详细说明它们与软件性能之间的关系。

> 单例模式
> 代理模式
> 享元模式
> 装饰器模式
> 观察者模式
> Value Object 模式
> 业务代理模式

#### Value Object 模式和业务代理模式
在 J2EE 软件开发中，通常会对系统模块进行分层，展示层主要负责数据的展示，定义数据库的 UI 组织模式；业务逻辑层负责具体业务逻辑处理；持久层通常指数据库以及相关操作。

在一个大型系统中，这次层次很有可能被分离，并部署在不同的服务器上。而在两个层次之间，可能通过远程过程调用 RMI 等方式进行通信。

例如：客户端获取服务端的订单客户名、商品名
数量信息，需要三次交互，分别是 getClientName()、getProName()、getNumber()，基于这种模式的通信方式是一种可行的解决方案，但它存在两个严重的问题：
（1）对于获取一个订单对象而言，这个操作模式略显繁琐，且不具备较好的可维护性。
（2）前后累计进行了 3 次客户端与服务端的通信，性能成本较高。
为了解决这两个问题，就可以使用 Value Object 模式。这种模式提倡将一个对象的各个属性进行封装，将封装后的对象在网络中传递，从而是系统拥有更好的交互模型，并且减少网络通信次数，从而提高系统性能。

#### 业务代理模式
Value Object 模式是将远程调用的传递数据封装在一个串行化对象中进行传输，而业务代理模式则是将一组由远程方法调用构成的业务流程，封装在一个位于展示层的代理类中。
比如：如果用户需要修改一个订单，订单修改操作可分为 3 个操作：
（1）校验用户 checkUser()
（2）获取旧的订单信息 getOrder()
（3）更新订单 updateOrder()
参考代码（用来解释，并不是完整的）
```
/**
 * 订单信息
 */
public class Order implements Serializable {
    private static final long serialVersionUID = 606071611515064692L;
    private int orderId;
    private String clientName;
    private int number;
    private String productName;
}
```
```
public interface IOrderManager extends Remote {
    // Value Object 模式
    /**
     * 获取用户订单信息
     *
     * @param id
     * @return
     */
    public Order getOrder(int id);

    /**
     * 检查用户权限
     *
     * @param id
     * @return
     */
    boolean checkUser(int id);

    /**
     * 更新用户订单
     *
     * @param order
     */
    void updateOrder(Order order);


//    public String getClientName(int id);
//    public String getProName(int id);
//    public int getNumber(int id);
```
```
/**
 * 业务代理模式类
 */
public class BusinessDelegate {
    /**
     * 代理的对象 封装了检查用户权限 获取用户订单信息 更新用户订单业务操作
     */
    IOrderManager manager = null;

    /**
     * 初始化代理的对象
     *
     * @throws RemoteException
     * @throws NotBoundException
     * @throws MalformedURLException
     */
    public BusinessDelegate() throws RemoteException, NotBoundException, MalformedURLException {
        manager = (IOrderManager) Naming.lookup("OrderManager");
    }

    /**
     * 检查用户动作 缓存起来方便重复使用
     *
     * @param id
     * @return
     */
    public boolean checkUserFromCache(int id) {
        // todo
        return false;
    }

    /**
     * 执行用户权限检查
     *
     * @param id
     * @return
     */
    public boolean checkUser(int id) {
        if(!checkUserFromCache(id)) {
            return manager.checkUser(id);
        }
        return false;
    }

    /**
     * 获取用户订单信息缓存
     *
     * @param id
     * @return
     */
    public Order getOrderFromCache(int id) {
        // todo
        return null;
    }

    /**
     * 获取用户订单信息
     *
     * @param id
     * @return
     */
    public Order getOrder(int id) {
        Order order = getOrderFromCache(id);
        if(null == order) {
            return manager.getOrder(id);
        }
        return order;
    }

    /**
     * 更新用户订单信息
     *
     * @param order
     * @return
     */
    public boolean updateOrder(Order order) {
        int id = order.getOrderId();
        if(checkUser(id)) {
            Order orderVO = getOrder(id);
            orderVO.setNumber(100);
            orderVO.setProductName("New Product Name");
            orderVO.setClientName("New Client Name");
            manager.updateOrder(orderVO);
        }
        return true;
    }
}
```