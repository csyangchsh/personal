# Hotspot JVM 垃圾收集器

## Stop-the-world

当Stop-the-world发生时，除了GC所需的线程以外，所有线程都处于等待状态，直到GC任务完成。GC优化很多时候就是指减少Stop-the-world发生的时间。

## 分代垃圾收集

###Weak Generational Hypothesis

* 大多数分配对象的存活时间很短
* 存活时间长的对象很少引用存活时间短的对象

### Hotspot JVM 分代

#### 新生代 (Young Generation) - Minor GC

大多数新创建的对象都分配在新生代中。通常新生代空间较小，收集频繁，收集之后存活的对象较少。

#### 老年代 (Old Generation) - Full GC

新生代中长期存活的对象最后会Promote或Tenure到老年代。通常老年代空间比新生代大，空间占用的增长比新生代慢。Full GC执行频率低，但是执行时间较长。

#### 永久代 (Permanent Generation) - Full GC

存储元数据，Class的数据结构、Interned String等。

**Hotspot JVM Full GC收集整个堆。**

### 新生代

**Eden** <br />
**Survivor (一对)**

1. 绝大多数刚刚被创建的对象会存放在伊甸园空间。
2. 在伊甸园空间执行了第一次GC之后，存活的对象被移动到其中一个幸存者空间。
3. 此后，在伊甸园空间执行GC之后，存活的对象会被堆积在同一个幸存者空间。
4. 当一个幸存者空间饱和，还在存活的对象会被移动到另一个幸存者空间。之后会清空已经饱和的那个幸存者空间。
5. 在以上的步骤中重复几次依然存活的对象，就会被移动到老年代。

**过早提升 (Premature Promotion)**
在Minor GC的过程中，Survivor可能不足以容纳Eden和另一个Survior中的存活对象。如果Survivor中的存活对象溢出，多余的对象将被移动老年代。<br />
老年代中的短期存活对象增长，可能会引发严重的性能问题。(什么样的性能问题?)

**提升失败 (Promotion Failure)**
在Minor GC的过程中，如果老年代满了而无法容纳更多对象，Minor GC之后通常就会进行Full GC，这将导致遍历整个Java堆。


####快速内存分配

**Bump-the-pointer** <br />
**TLABs (Thread-Local Allocation Buffers)** <br />

大部分时候， Hotspot JVM的new Object()操作只需要大约十条指令。

### 老年代

垃圾收集器不需要扫描整个老年代就能识别新生代中的存活对象，从而缩短Minor GC的时间。Hotspot VM采用卡表(Card Table)的数据结构来达到这个目的。老年代以512字节为块划分成若干Card。卡表是单字节数组，每个元素对应堆上的一张Card。每次老年代对象中某个引用新生代对象的字段发生变化时，就必须将该卡对应的卡表元素设置为适当的值，将卡标记为脏。<br />
在Minor GC过程中，只会在脏卡中扫描排查老年代-新生代引用。


### 分代垃圾收集的优点

每个分代都可以依据自身特性使用最适当的垃圾收集算法。


## 垃圾收集器

### 1. Serial收集器 (-XX:+UseSerialGC)

单线程 <br />
新生代 (DefNew): 采用复制算法 <br />
老年代: 采取mark-sweep-compact算法 <br />

### 2. Parallel GC (-XX:+UseParallelGC)

新生代 (PSYoungGen): Parallel Scavenge收集器是一个多线程收集器，也是使用复制算法，但它的对象分配规则与回收策略都与ParNew收集器有所不同，它是以吞吐量最大化（即GC时间占总运行时间最小）为目标的收集器实现，它允许较长时间的STW换取总吞吐量最大化。 

老年代 (PSOldGen): 单线程，mark-sweep-compact

### 3. Parallel Old GC(-XX:+UseParallelOldGC)

新生代 (PSYoungGen)

老年代 (ParOldGen) <br />
Parallel Old GC分为三步：标记-汇总-压缩（mark–summary–compaction）。汇总（summary）步骤与清理（sweep）的不同之处在于，其将依然幸存的对象分发到GC预先处理好的不同区域，算法相对清理来说略微复杂一点。

### 4. CMS GC (-XX:+UseConcMarkSweepGC)

CMS是一种以最短停顿时间为目标的收集器，使用CMS并不能达到GC效率最高（总体GC时间最小），但它能尽可能降低GC时服务的停顿时间，这一点对于实时或者高交互性应用来说至关重要，这类应用对于长时间STW一般是不可容忍的。CMS收集器使用的是标记－清除算法，也就是说它在运行期间会产生空间碎片，所以虚拟机提供了参数开启CMS收集结束后再进行一次内存压缩。 

在使用这个GC类型之前你需要慎重考虑。如果因为内存碎片过多而导致压缩任务不得不执行，那么stop-the-world的时间要比其他任何GC类型都长，你需要考虑压缩任务的发生频率以及执行时间。


### 5. G1

Todo


## References

Java性能优化权威指南 <br />
[JVM内存管理：深入垃圾收集器与内存分配策略](http://hllvm.group.iteye.com/group/wiki/2859-JVM) <br />
[成为JavaGC专家Part I — 深入浅出Java垃圾回收机制](http://www.importnew.com/1993.html)


