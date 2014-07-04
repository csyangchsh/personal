# Better Code - String

## 避免隐式的String字符串

### Bad Example

1. `a = a + b; // a and b are Strings`

2. 
```
String result = foo() + arg;
result += boo();
System.out.println("result = " + result);
```

**TODO**: 编译器会生成怎样的代码？

### 解决方案

使用 StringBuilder/StringBuffer

```
StringBuilder value = new StringBuilder("result = ");
value.append(foo()).append(arg).append(boo());
System.out.println(value);
```
以上代码仅生成一个StringBuilder 对象，和一个存储最终result 的String对象。

## String.intern()

### String pooling

将内容相同但identity不同的字符串用一个shared String object替代。可以使用String.intern()达到这一目的。

### String.intern() in Java 6

String pool存放在PermGen上。PermGen string pool中还包含了字符串常量。(**如果class/method m没有被load/call, 定义的常量不会被load。**)

### String.intern() in Java 7

String pool存放在Heap上。

**String pool中的string是可以被垃圾回收的。**

String pool的实现是一个固定容量的hash map，每个bucket包含hash code相同的string组成的list。

### `-XX:StringTableSize=N`

在Java 6和Java 7u40之前的版本中, 默认值是1009。Java 7u40默认值是60013。

### `-XX:+PrintStringTableStatistics`

### Summary

如果有大量的内容相同的字符串，可以考虑使用internalize strings来减少内存使用。

**在Java 6中, 所有internalized stings都会存放到permanent generation**，确保permanent generation有足够的空间。

确保internalize所有和internalized strings相比较的strings。

确保能够接受internalizing的额外CPU开销，这是一个native method。

**TODO**: 执行测试代码

## References

[减少GC开销的5个编码技巧](http://www.importnew.com/10472.html) <br />
[REDUCING MEMORY USAGE WITH STRING.INTERN()
](https://plumbr.eu/blog/reducing-memory-usage-with-string-intern) <br />
Java Performance: The Definitive Guide <br />
[String.intern in Java 6, 7 and 8 – string pooling](http://java-performance.info/string-intern-in-java-6-7-8/)

