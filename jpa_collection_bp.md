#JPA集合映射最佳实践

* 使用List时，不要认为没有指定排序就会自动排序。List中元素的顺序可能会受到DB查询结果的影响。无法保证多次查询的顺序是一致的。

When using a List, do not assume that it is ordered automatically if you have not actually specified any ordering. The List order might be affected by the database results, which are only partially deterministic with respect to how they are ordered. There are no guarantees that such an ordering will be the same across multiple executions.


* 通常都可以根据对象的某一属性进行排序。与使用特定的order column来维护persistent List元素顺序相比，使用@OrderBy注解是最好的方式。（更新order column会带来overhead）只有在别无选择情况才使用order column。

It will generally be possible to order the objects by one of their own attributes. Using the @OrderBy annotation will always be the best approach when compared to a persistent List that must maintain the order of the items within it by updating a specific order column. Use the order column only when it is impossible to do otherwise.

* Map很有用，但是相对来说有些复杂，不容易正确配置。到一定阶段，和各种relationship以及elment collection相比，它所提供的模型支持和松散关联支持会使它成为理想的选择。

Map types are very helpful, but they can be relatively complicated to properly configure. Once you reach that stage, however, the modeling capabilities that they offer and the loose association support that can be leveraged makes them ideal candidates for various kinds of relationships and element collections.

* 和List类似，使用Map最高效、最推荐的方式是选用target object的属性作为Map的key。使用基本属性类（basic attribute type，如String，Integer）型作为key最常见，也最有效，能够解决大部分问题。在关联basic object时，使用基于basic key和value的Map是非常有用的配置。

As with the List, the preferred and most efficient use of a Map is to use an attribute of the target object as a key, making a Map of entities keyed by a basic attribute type the most common and useful. It will often solve most of the problems you encounter. A Map of basic keys and values can be a useful configuration for associating one basic object with another.

* 避免在Map中使用embedded object，尤其是作为key，因为通常不会定义它们的identity。只有在确实需要使用embedded object的时候才使用，一般用作值对象。

Avoid using embedded objects in a Map, particularly as keys, because their identity is typically not defined. Embeddables in general should be treated with care and used only when absolutely necessary.

* 即使可以实现也不推荐在集合中支持重复值和null。会导致在集合上的某些操作变慢，并且增加数据库操作。有的时候会导致删除和插入操作，而不是更新操作。

Support for duplicate or null values in collections is not guaranteed, and is not recommended even when possible. They will cause certain types of operations on the collection type to be slower and more database-intensive, sometimes amounting to a combination of record deletion and insertion instead of simple updates.

参考书籍 - Pro JPA 2