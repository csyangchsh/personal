# Hotspot JVM GC Log

## 重要的GC数据

* 当前使用的垃圾收集器
* Java Heap的大小
* 新生代和老年代的大小
* 永久代的大小
* Minor GC的持续时间
* Minor GC的频率
* Minor GC的空间回收量
* Full GC的持续时间
* Full GC的频率
* 每个并发垃圾收集周期内的空间回收量
* 垃圾收集前后Java Heap的占用量
* 垃圾收集前后新生代和老年代的占用量
* 垃圾收集前后永久代的占用量
* 是否老年代或永久代的占用触发了Full GC
* 应用是否显示调用了System.gc()

## 测试使用的JVM Options

-XX:+PrintGCDetails <br />
-XX:+PrintGCTimeStamps <br />
-XX:+PrintGCDateStamps <br />
-XX:+PrintGCCause <br />
-XX:+PrintGCApplicationStoppedTime <br />
-XX:+PrintPromotionFailure <br />
-XX:+PrintGCApplicationConcurrentTime <br />
-XX:+UseG1GC <br />
-XX:+ScavengeBeforeFullGC <br />
-Xms512m <br />
-Xmx512m <br />
-Xloggc:/Users/yang/workspace/jvm/UseG1GC.log <br />

使用-XX:+UseParalleGC或-XX:+ParallelOldGC时，如果关闭-XX:-ScavengeBeforeFullGC，在Full GC之前不会进行Minor GC。如果开启-XX:+ScavengeBeforeFullGC，在Full GC之前会做一次Minor GC，分担一部分Full GC原本要做的工作。

我分别测试了7个GC选项，并且输出到log文件中，以上例子使用的是G1。

## Log file

### -XX:+UseParalleGC -XX:-UseParalleOldGC

**从JDK 7 update 4 开始，Paralle Old GC 默认会开启。-XX:+UseParalleGC 和 -XX:+UseParalleOldGC 效果是一样的。**

#### Minor GC

```
2014-07-02T14:42:58.141-0800: 0.625: Application time: 0.5410920 seconds

2014-07-02T14:42:58.141-0800: 0.625: 
[GC (Allocation Failure) 
	[PSYoungGen: 132096K->21488K(153600K)] 
	132096K->115536K(503296K), 0.1372000 secs] 
	[Times: user=0.39 sys=0.07, real=0.13 secs] 

2014-07-02T14:42:58.278-0800: 0.762: Total time for which application threads were stopped: 0.1373680 seconds
```

`2014-07-02T14:42:58.141-0800:` 对应 `-XX:+PrintGCDateStamps`

`0.625` 对应 `-XX:+PrintGCTimeStamps`，自JVM启动以来流逝的时间，单位是秒。

`Application time: 0.5410920 seconds` 对应 `-XX:+PrintGCApplicationConcurrentTime`， 应用在安全点操作之间的运行时间。

`[GC` 说明是 Minor GC

`(Allocation Failure)` 对应 `-XX:+PrintGCCause`，这里是因为为新对象分配空间失败。

`[PSYoungGen: 132096K->21488K(153600K)]` 是新生代信息，PSYoungGen表示新生代使用的是Paralle Scavenge垃圾收集器。132096K是垃圾收集前新生代的占用量，21488K是垃圾收集后新生代的占用量。因为Minor GC后，Eden为空，所以21488K也是Survivor占用量。**括号里的153600K不是新生代的占用量，而是Eden和一块正被占用的Survivor的和。**

`132096K->115536K(503296K)` 是垃圾收集前后Java Heap的使用情况（新生代和老年代的占用量总和），及Java Heap的大小（新生代和老年代的总和）。132096K是垃圾回收前Java Heap的占用量，115536K是垃圾回收后Java Heap的占用量。503296K是Java Heap的大小。(Eden + 一块Survivor + Old ?)

**老年代大小: 503296K - 153600K = 349696K**

`0.1372000 secs` 是垃圾收集花费的时间

`[Times: user=0.39 sys=0.07, real=0.13 secs]` 是CPU的使用时间

`2014-07-02T14:42:58.278-0800: 0.762: Total time for which application threads were stopped: 0.1373680 seconds` 对应 `-XX:+PrintGCApplicationStoppedTime`，应用停顿时间。

#### Full GC

```
2014-07-02T16:30:40.633-0800: 1.289: 
[Full GC (Ergonomics) 
	[PSYoungGen: 21504K->16625K(153600K)] 
	[PSOldGen: 344856K->349695K(349696K)] 
	366360K->366321K(503296K) 
	[PSPermGen: 2985K->2985K(21504K)], 
	0.6499850 secs] 
	[Times: user=0.64 sys=0.01, real=0.65 secs] 

2014-07-02T16:30:41.283-0800: 1.939: Total time for which application threads were stopped: 0.9797870 seconds
```
`[Full GC` 说明是完全垃圾收集。

`[PSOldGen: 344856K->349695K(349696K)]` 是老年代信息。PSOldGen表示是使用的Serial Mark Sweep Compact收集器。

`[PSPermGen: 2985K->2985K(21504K)]` 是永久代信息。

Full GC中值得重点关注的是垃圾收集前老年代和永久代的占用量。当老年代或永久代的占用量接近其容量时都会触发Full GC。

##Reference

Java性能优化权威指南 <br />
Pierre-Hugues Charbonneau's [verbosegc output tutorial - Java 7](http://javaeesupportpatterns.blogspot.de/2011/10/verbosegc-output-tutorial-java-7.html) 
我采用了这篇文章中的测试代码。