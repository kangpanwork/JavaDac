缓存行 `Cache Line`  `CPU` 高速缓存中分配的最小存储单位。`CPU` 缓存分为 `Cache L1 L2 L3` 一级 二级 三级 缓存，查看本机缓存
`pom.xml` 引入
```
        <dependency>
            <groupId>fr.ujm.tse.lt2c.satin</groupId>
            <artifactId>cachesize</artifactId>
            <version>0.2.1</version>
        </dependency>
```
不同处理器的缓存行 `L1 L2 L3` 高速缓存行字节宽不一样 有 `32` 字节和 `64` 字节
``` java
     CacheInfo info = CacheInfo.getInstance();
     CacheLevelInfo cacheLevelInfo = info.getCacheInformation(CacheLevel.L1, CacheType.INSTRUCTION_CACHE);
     System.out.println("L1 Cache info:" + cacheLevelInfo.toString());
```
缓存行在内存中加载的地址是连续的 假如缓存行是 `64` 字节 有两个变量 `a` 和 `b` ，`a` + `b` 不足 64 字节，那么会在同一个缓存行造成伪共享，处理器修改 `a` 的时候，其他处理器读取 `b` 的时候该缓存行是失效的( `MESI` 协议)，其他处理器不得不重新从系统内存中加载。

处理伪共享 可以采取 `padding` 方式 ，当字节不足缓存行大小时进行填满  和 `Java8` 的 `@sum.misc.Contended` 使两个变量 在不同缓存行中

```java
   @sun.misc.Contended static final class CounterCell {
        volatile long value;
        CounterCell(long x) { value = x; }
    }
```
`ConcurrentHashMap` 的 `CounterCell` 上加了 `@Contended` 注解  解决了 `++` 操作产生伪共享

看下 `padding` 实现 , 使 `long` 变量进行填充

```java
abstract class SingleProducerSequencerPad extends RingBufferProducer
{
	protected long p1, p2, p3, p4, p5, p6, p7; // 4*7 = 28 字节
}
abstract class SingleProducerSequencerFields extends SingleProducerSequencerPad
{

	/** Set to -1 as sequence starting point */
	protected long nextValue = RingBuffer.Sequence.INITIAL_VALUE; // 28 + 4 = 32 字节
	protected long cachedValue = RingBuffer.Sequence.INITIAL_VALUE; // 32 + 4 = 36 字节
}
final class SingleProducerSequencer extends SingleProducerSequencerFields {
	protected long p1, p2, p3, p4, p5, p6, p7; // 36 + 28 = 64 字节
}
```
`volatile`  也会使处理器缓存回写到内存并且导致其他内存缓存无效，在 `64` 字节宽的处理器  早期`JDK`版本采取了追加字节的方式来进行性能优化







