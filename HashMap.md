### HashMap
#### 定义
```
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```
#### 构造函数
HashMap 提供了三个构造函数
- HashMap()：构造一个具有默认初始容量 16 和 默认加载因子 0.75 的空 HashMap
- HashMap(int initialCapacity)：构造一个带指定初始容量和默认加载因子 0.75 的空 HashMap
- HashMap(int initialCapacity, float loadFactor)：
构造一个带指定初始容量和加载因子的空 HashMap

#### 初始容量 加载因子
这两个参数是影响 HashMap 性能的重要参数，其中容量表示哈希表中桶的数量，初始容量是创建哈希表时的容量，加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。

对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为 0.75，一般情况下我们是无需修改的。

#### 数据结构
数组实现：  `Node<K,V>[] table`

链表实现：静态的内部类 Node
```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}
```
红黑树实现：静态的内部类 TreeNode
```
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev; 
        boolean red;
}
```
#### 存储实现
HashMap 存放 key value 的时候，先对 key 进行 hash 计算
```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
先 resize 一个 `Node[]` 数组，计算的 hash 值来判断该 `tab[i = (n - 1) & hash]` 位置上是不是为 null，是 null
就 `new Node<>(hash, key, value, next)`，生成一个索引为 hash 的 Node 链表
```
Node<K,V>[] tab; Node<K,V> p; int n, i;
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
            
```
如果不为 null，说明数组中存在相同的 hash 索引，这个时候就往链表里面存储数据，key 相同的时候会覆盖
```
Node<K,V> p;
Node<K,V> e; K k;
if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
if (e != null) {
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```
JDK1.8 优化了链表存储方式，当链表节点数量 `binCount >= TREEIFY_THRESHOLD - 1` 的时候 也就是 8 的时候使用的是红黑树存储，参考 `treeifyBin(tab,hash)`方法
```
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1)
            treeifyBin(tab, hash);
        break;
    }
}

```
#### 扩容
参考源码 `resize()` 方法，newCap 为新的容量，oldCap 为旧的容量，新的容量为旧的容量的 2 倍
```
 if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
  newThr = oldThr << 1; 
```
#### 读取值
通过 key 的 hash 值找到在数组中的索引处，然后根据 hash 及 key 找到对应的 Node，再返回 Node 的 value
```
 Node<K,V> e;
if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
     return e;
```
```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```