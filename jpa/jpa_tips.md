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

* Extra Lazy Fetching

```
Customer.java
@OneToMany(cascade=CascadeType.ALL, fetch = FetchType.LAZY)
@LazyCollection(LazyCollectionOption.EXTRA)
@IndexColumn(name = "order_index")
private List<Order> orders = new ArrayList<Order>();
```
JSP

```
<c:forEach items="${customers}" var="customer">
    <c:set var="lastOrderIndex" value="${fn:length(customer.orders) - 1}"/>
    <tr>
        <td><c:out value="${customer.name}" /></td>
        <td><fmt:formatNumber value="${customer.orders[lastOrderIndex].amount}" maxFractionDigits="2" type="currency" currencySymbol="$"/></td>
        <td><fmt:formatDate value="${customer.orders[lastOrderIndex].date}"/></td>
        <td><c:out value="${customer.orders[lastOrderIndex].paymentMethod.name}"/></td>
    </tr>
</c:forEach>
```

	1. Customer.getOrders() not fetching the Orders
	2. Customer.getOrders().size() issues a COUNT query
	3. Customer.getOrders().get(index) loads the specific Entity

###Links

http://www.javacodegeeks.com/2011/02/hibernate-mapped-collections.html<br />
https://github.com/BrunoChauvet/orm-anti-patterns