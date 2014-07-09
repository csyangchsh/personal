# ArrayList

**TODO** source code, test code

## 添加元素

* `add(E)`
* `add(int, E)`
* `addAll(List<E>)`
* `addAll(int, List<E>)`

只有一个参数的方法是将新元素加到ArrayList的尾部，速度最快。<br />
指定插入位置的方法需要使用`System.arrayCopy()`方法复制插入位置处右侧的元素，所以复杂度是O(n)。<br />
**不要在ArrayList头部插入过多元素。** <br />

如果没有足够空间容纳新元素，需要扩展ArrayList。<br />
**Java 7扩展逻辑**：oldSize * 3 (Java 6 oldSize * 3 / 2 + 1)

## 删除元素

### `remove(int)`

删除指定位置的元素，右侧的元素通过`System.arrayCopy()`方法左移，复杂度是O(n)。<br />

#### 问题

1. 对于`ArrayList<Integer>`，注意`remove(int)`是调用`remove(int)`还是`remove(Object)`。
需要手工转型到`remove(int)`或`remove(Object)`。
2. buffer implementation problem

### `remove(Object)`

删除第一个出现的指定元素，支持null，需要迭代所有元素，复杂度 O(n)。<br />
**如果知道元素位置不要调用此方法。** <br />

删除元素不会shrink数组大小，只有`trimToSize()`会。

### `removeAll(Collection)`

复杂度 O(n*n)，需要迭代所有元素，对每个元素调用参数Collection的`contains`方法。
****

### `retainAll(Collection)`

复杂度 O(n*n)

### `subList(int, int)`

创建部分元素的view。

Java 6: 定义在AbstractList中。每个sublist持有parent list的引用，使用parent list的方法，添加offset作为索引。<br />

Java 7: `ArrayList.subList()`创建的sublist持有original list的引用，并且直接访问elementData数组。 <br />

There are 3 obvious use cases for this method:

* Fast clearing parts of the list
* Using it in for-each iteration over the part of the list
* Using it in the recursive algorithms

### `get(int)`

This method is still extremely fast and you will see any difference only on hundreds of millions accesses.

### `contains(Object)`, `indexOf(Object)`

复杂度 O(n)

## Summary

* Add elements to the end of the list
* Remove elements from the end too
* Avoid contains, indexOf and remove(Object) methods
* Even more avoid removeAll and retainAll methods
* Use subList(int, int).clear() idiom to quickly clean a part of the list

## Reference

http://java-performance.info/arraylist-performance/


