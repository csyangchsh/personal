#JPA/Hibernate 常见问题

###LazyInitialisationException

集合是用惰性加载，到页面的时候session已经关闭，无法访问集合元素的属性。

```
public class Customer {
    @OneToMany(mappedBy = "customer")
    private List<Order> orders = new ArrayList<Order>();
}

public class Order {
    @ManyToOne
    @NotNull
    private Customer customer;
 
    @OneToOne
    @NotNull
    private PaymentMethod paymentMethod;
}

@Query("SELECT DISTINCT c "
    + "FROM Customer c "
    + "ORDER BY c.name")
public List<Customer> findAllCustomers();

```
Service的中的方法

```
@Transactional(readOnly = true)
public List<Customer> findAllCustomers() {
    List<Customer> customers = customerRepository.findAll();
    return customers;
}
```

JSP

```
<c:forEach items="${customer.orders}" var="order">
    <tr>
        <td><fmt:formatNumber value="${order.amount}" maxFractionDigits="2" type="currency" currencySymbol="$"/></td>
        <td><fmt:formatDate value="${order.date}"/></td>
        <td><c:out value="${order.paymentMethod.name}"/></td>
    </tr>
</c:forEach>
```
####解决方法

* 调用findAllCustomers()后，去访问Customer的Orders集合中的每个Order，触发数据库查询。

```
@Transactional(readOnly = true)
public List<Customer> findAllCustomers() {
    List<Customer> customers = customerRepository.findAll();
    for(Customer c: customers) {
        List<Order> orders = c.getOrders();
        for(Order o: orders) {
            o.getAmount();
            o.getPaymentMethod().getName();
        }
    }
    return customers;
}
```

根据实际情况使用Query返回所需要的数据或许更好。

* 使用OpenEntityManagerInViewFilter或者OpenSessionInViewFilter

```
<filter>
    <filter-name>openEntityManagerInViewFilter</filter-name>
    <filter-class>org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>openEntityManagerInViewFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

###N+1

第一个例子也存在N+1问题，产生了一条查询Customer的语句和10条查询Order的语句。

####解决方法

使用Jion Fetch

```
@Query("SELECT DISTINCT c " //
        + "FROM Customer c " //
        + "LEFT JOIN FETCH c.orders")
public List<Customer> findAllCustomersFetchOrders();

@Query("SELECT DISTINCT c " //
        + "FROM Customer c " //
        + "LEFT JOIN FETCH c.orders o " //
        + "LEFT JOIN FETCH o.paymentMethod")
public List<Customer> findAllCustomersFetchOrdersAndPaymentMethods();
```

###分页时，Query中jion了collection

```
@Query("SELECT DISTINCT c "
    + "FROM Customer c "
    + "LEFT JOIN FETCH c.orders "
    + "ORDER BY c.name")
public List<Customer> findCustomers(Pageable page);

@Transactional(readOnly = true)
public List<Customer> findAllCustomersFetchOrders(Pageable pageable) {
    return customerRepository.findAllCustomersFetchOrders(pageable);
}

@RequestMapping("/findCustomers/{page}")
public ModelAndView findAllCustomersFetchOrders(@PathVariable("page") Integer page) {
    Pageable pageable = new PageRequest(page, 5);
    return new ModelAndView(LIST_CUSTOMERS_VIEW, CUSTOMERS, customerService.findAllCustomersFetchOrders(pageable));
}
```

会在log中看到'firstResult/maxResults specified with collection fetch; applying in memory!'的警告。

每个Customer的Order数目都不一样，Hibernate不知道要查询多少条记录。只能查询所有的Customer和Order映射到Entity，取最前面5条返回。

####解决方案

思考一下需求，主要目的是查询Order。可以不使用双向关联。

```
public class Customer {
    // No reference to Unbounded Collection of Orders
}

public class Order {
    @ManyToOne(fetch = FetchType.LAZY)
    private Customer customer;
 
    @OneToOne(fetch = FetchType.LAZY)
    private PaymentMethod paymentMethod;
}

@Query("SELECT DISTINCT o "
    + "FROM Order o "
    + "LEFT JOIN FETCH o.customer c "
    + "ORDER BY c.name, o.date")
public List<Order> findOrders(Pageable page);
```

###在内存中进行Aggregate操作

```
@Transactional(readOnly = true)
public List<Map<String, Object>> customerReport() {
 
    List<Customer> customers = customerReporistory.findAllCustomersFetchOrders();
 
    for(Customer customer : customers) {
        List<Order> orders = customer.getOrders();
        double amount = 0.0D;
        for(Order order : orders) {
            amount += order.getAmount();
        }
        // Rest of implementation
    }
}
```

如果Customer非常多，可能会触发OOM。

####解决方法

放在Named Query中计算

```
@Query("SELECT new Map(c as customer, "
        + "   SUM(o.amount) as total, "
        + "   MIN(o.date) as since) "
        + "FROM Customer c, Order o "
        + "WHERE o.customer = c "
        + "GROUP BY c "
        + "ORDER BY c.name")
public List<Map<String, Object>> customerReport();
```

**使用Select new避免load整个object graph。**

###Links

https://github.com/BrunoChauvet/orm-anti-patterns