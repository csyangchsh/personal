# Common Hotspot JVM 6 Options

**This is NOT a full list, I just list common options during my study.** <br />
**[Alexey Ragozin created a very nice sheet.](http://blog.ragozin.info/2013/11/hotspot-jvm-garbage-collection-options.html)**

## GC Algorithms
Young Generation | Old Generation | JVM Flags
---------------- | -------------- | ---------
Serial (DefNew)  | Serial Mark Sweep Compact | -XX:+UseSerialGC
Parallel scavenge (PSYoungGen) | Serial Mark Sweep Compact (PSOldGen) | -XX:+UseParallelGC
Parallel scavenge (PSYoungGen) | Parallel Mark Sweep Compact (ParOldGen) | -XX:+UseParallelOldGC
Parallel (ParNew) | Serial Mark Sweep Compact | -XX:+UseParNewGC
Serial (DefNew) | Concurrent Mark Sweep | -XX:-UseParNewGC (*) -XX:+UseConcMarkSweepGC
Parallel (ParNew) | Concurrent Mark Sweep | -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
G1 | G1 |-XX:+UseG1GC

**Note:** -XX:-UseParNewGC explicitly disables parallel mode.

## GC Log
Option | Comments
------ | --------
-verbose:gc/-XX:+PrintGC | Print basic GC info
-XX:+PrintGCDetails | Print detail GC info
-XX:+PrintGCTimeStamps | Print timestamps for each GC event (seconds count from start of JVM) 
-XX:+PrintGCDateStamps | Print timestamps for each GC event
-XX:+PrintReferencGC | Print times for special (weak, JNI, etc) reference processing during STW pause
-XX:+PrintGCCause | Add cause in GC log
-XX:+PrintPromotionFailure | Print additional information for promotion failure
-XX:+PrintGCApplicationStoppedTime |
-XX:+PrintGCApplicationConcurrentTime |

### GC Log File
-Xloggc:\<file\> <br />
-XX:+UseGCLogFileRotation <br />
-XX:GCLogFileSize=10m <br />
-XX:NumberOfGCLogFiles=10 <br />

## Memory Sizing

-Xms256m/-XX:InitialHeapSize=256m (\*)<br />
-Xmx2g/-XX:MaxHeapSize=2g (\*)<br />
-XX:NewSize=64m -XX:MaxNewSize=64m (\*)<br />

-XX:NewRatio=2 <br />
-XX:SurvivorRatio=6 <br />
-XX:PermSize=256m -XX:MaxPermSize=1g

-Xss256k (size in bytes)/-XX:ThreadStackSize=256 (size in Kbytes)
-XX:MaxDirectMemorySize=1g






 

