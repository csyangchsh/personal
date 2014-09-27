#JPA Tips

* setFirstResult(), setMaxResults()

The setFirstResult() and setMaxResults() methods should not be used with queries that join across collection relationships (one-to-many and many-to-many) because these queries may return duplicate values. The duplicate values in the result set make it impossible to use a logical result position.

* Cascade

Cascade settings are unidirectional. This means that they must be explicitly set on both sides of a relationship if the same behavior is intended for both situations. 

通常不应该在关联两端都加Cascade。

* 优先使用单向关联

* 使用Hibernate时，对于one-to-many，many一侧作为relationship owner会避免hibernate产生多余的update语句。（mapedBy, inverse）

* 实现equals()/hashcode()方法

* 使用version property减少数据库查询

* 对于eager fetching的集合，默认使用的是select fetch。测试使用join fetch是否能提高性能。


###Links

http://www.javacodegeeks.com/2011/02/hibernate-mapped-collections.html