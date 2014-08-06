# JVM OOM Errors

## java.lang.OutOfMemoryError: Java heap space

在JVM Heap里加入新的数据时，数据的大小超过了Heap可以提供的空间大小。

### 原因

最常见原因是应用太大而JVM Heap设置小了，应用需要更多的Heap space来运行。

### 编程错误

* 数据量突然增长，超过了应用的设计预期。
* Memory leak

### 解决方法

* -Xmx 扩大JVM Heap space

**扩大JVM Heap space 也会导致JVM GC pause time变长。**

* 修复代码中的Memory leak问题。

## java.lang.OutOfMemoryError: Permgen space

Permgen的空间被耗尽了。Permgen用来存放Class declaretions，包括：

* Names of the classes
* Fields of the class
* Methods of a class with the bytecode of the methods
* Constant pool information
* Object arrays and type arrays associated with a class
* Just In Time compiler optimizations

### 原因

过多的Class或过大的Class被加载到Permgen了。常常在redeploy应用的时候发生这个错误。有些lib导致无法丢弃旧的class loader，使得旧的Class仍然在Permgen中。

### 解决方法

-XX:MaxPermSize

-XX:+CMSClassUnloadingEnabled （必须启用-XX:+UseConcMarkSweepGC）

## Out of memory: Kill process or sacrifice child

Virtual memory (including swap) is consumed to the extent where the overall operating system stability is put into risk.

### 原因

Overcommitting configuration （/proc/sys/vm/overcommit_memory） allows to allocate more and more memory for this rogue process which can eventually trigger the “Out of memory killer”.

### 解决方法

* 硬件升级
* Scaling horizontally

**不建议增加swap space**

## java.lang.OutOfMemoryError: GC overhead limit exceeded

“java.lang.OutOfMemoryError: GC overhead limit exceeded” error will be displayed **when your application has exhausted pretty much all the available memory and GC has repeatedly failed to clean it.**

### 原因

When failing with the “java.lang.OutOfMemoryError: GC overhead limit exceeded” error JVM is signaling that your application is spending too much time in garbage collection with little to show for it. By default the JVM is configured to throw this error if you are spending more than 98% of the total time in GC and after the GC **less than 2% of the heap is recovered**.

### 解决方法

* -XX:-UseGCOverheadLimit (强烈不建议使用)
* -Xmx
* Profile the application, find root cause.

## java.lang.OutOfMemoryError: Out of swap space?

The swap space is also exhausted and the new allocation is failing due to the lack of both physical memory and swap space.

### 原因

The java.lang.OutOfmemoryError: request bytes for . Out of swap space? is thrown by JVM when an allocation from the native heap failed and the native heap is close to exhaustion. The message indicates the size (in bytes) of the allocation which failed and the reason for the memory request.

java.lang.OutOfMemoryError: Out of swap space? is often caused by the operation system level issues, such as:

* The operating system is configured with insufficient swap space.
* Another process on the system is consuming all memory resources.

It is also possible that the application failed due to a native leak, for example, if application or library code is continuously allocating memory but is not releasing it to the operating system.

### 解决方法

First and often the easiest workaround is to increase swap space.

If your application is deployed next to a “noisy neighbour” with whom the JVM needs to compete for resources, you should isolate the services to separate (virtual) machines.

**In many cases, your only truly viable alternative is to either upgrade the machine to contain more memory or optimize the application to reduce the memory footprint.**

## java.lang.OutOfMemoryError: Unable to create new native thread

Java application has hit the limit on how many Threads it can launch.

### 原因

You have a chance to face the “java.lang.OutOfMemoryError: Unable to create new native thread” whenever JVM is asking a new thread from the OS. Whenever the underlying OS cannot allocate a new native thread, this OutOfMemoryError will be thrown.

### 解决方法

* Increasing the limits on the OS level. 
* Programming error: 创建太多线程。

## java.lang.OutOfMemoryError: Requested array size exceeds VM limit

The application at hand is trying to allocate an array larger than your Java Virtual Machine can support.

### 原因

The error is thrown by the native code within the JVM. It happens before allocating memory for an array when JVM performs a platform-specific check: whether the allocated data structure is addressable in this platform.

### 解决方案

* Check your code base to see whether you really need arrays that large, using smaller bulk.
* sun.misc.Unsafe

**TODO** testing

----
## References

* [https://plumbr.eu/outofmemoryerror](https://plumbr.eu/outofmemoryerror)
* [http://blog.ragozin.info/2012/05/four-flavours-of-out-of-memory-in.html](http://blog.ragozin.info/2012/05/four-flavours-of-out-of-memory-in.html)