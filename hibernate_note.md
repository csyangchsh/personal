# Hibernate Note

## Fetching Strategies

* When (Timing)
* How (style)

### Fetching timing:

#### Eager

* Immediate
* Enormous amount of unnecessary fetches for deep association tree

#### Lazy

* Fetched when first accessed
* Collections
	* LAZY by default
	* Utilizes Hibernate's internal concept of "persistent collections"
* Single attribute
	* Requires bytecode enhancement
	* Not typically used nor beneficial
* Single (ToOne) associations
	* Fetched when accessed
	* Proxy		* Default		* Fetched when accessed (except ID) 		* Handled internally by Hibernate	* No-proxy		* Fetched when accessed (including ID)		* No visible proxy		* Requires buildtime bytecode enhancement

#### Extra Lazy

* Collections only* Fetch individual elements as accessed
* Does not fetch entire collection

### Fetching style:

#### Join

* Join fetch (left/outer join)* Great for ToOne associations* Multiple collection joins	* Possible for non-bags	* Warning: Cartesian product! SELECT is normally faster

#### Select

* Follow-up selects
* Default style for collections (avoid cartesian product)
* Can result in 1+N selects (fetch per collection entry)

#### Batch

* Fetches multiple entries when one is accessed
* Simple select and list of keys
* Determines # of entries to fetch, based on # of provided keys
* Legacy - pre-determined sizes, rounded down
* Padded - same as Legacy, size rounded up
* Dynamic - build SQL for the # of keys, limited by "batch-size"

#### SUBSELECT

* Follow-up select
* Fetches all collection entries when accessed for the 1st time
* Original root entry select used as subselect
* Performance depends on the DB itself

----
## Cache
Hibernate中有三种缓存：

* first level cache - session scope
* second level cache - shared cross session
* query cache - result set of query against db

### Fisrt level cache

一级缓存跟踪session已经加载、访问过的实体的状态。一级缓存防止过多的数据库查询、更新操作，而是在session结束前，以批处理方式执行。一级缓存在session整个生命周期里有效。

如果一个实体在同一个session中修改了多次，Hibernate在session结束前执行包含所有修改的更新操作。

### Second level cache
二级缓存的是**实体数据，不是实体本身，** 类似于hash map，key是实体的identifier，value是原始值的list。

------------------------------------------<br />
| Person Data Cache                       <br />
|-----------------------------------------<br />
| 1 -> [ "John" , "Q" , "Public" , null ] <br />
| 2 -> [ "Joey" , "D" , "Public" ,  1   ] <br />
| 3 -> [ "Sara" , "N" , "Public" ,  1   ] <br />
------------------------------------------<br />

从数据库根据Id加载实体时，会装载二级缓存。例如`entityManager.find()`，遍历lazy initialized 关系。

二级缓存的作用域在SessionFactory内，同一个SessionFactory创建的session可以重用二级缓存。

二级缓存默认不开启。二级缓存采用plugin机制，使用第三方cache实现。

### Query cache
Query cache只是缓存实体的identifier。Query cache概念上像hashmap, key是query string和参数值，value是实体Id的list。

----------------------------------------------------------------<br />
| Query Cache                                             
|---------------------------------------------------------------<br />
| ["from Person where firstName=?", ["Joey"] ] -> [1, 2] ] <br/>
----------------------------------------------------------------<br />

执行cacheable JPQL/HQL query时会装载query cache。

Query cache默认不开启，最好和二级缓存一起使用。

#### Query cache和二级缓存如何一起工作

If a query under execution has previously cached results, then no SQL statement is sent to the database. Instead the query results are retrieved from the query cache, and then the cached entity identifiers are used to access the second level cache.

If the second level cache contains data for a given Id, it re-hydrates the entity and returns it. If the second level cache does not contain the results for that particular Id, then an SQL query is issued to load the entity from the database.

#### 陷阱

* 二级缓存miss可能会导致更多数据库查询；如果不开启二级缓存，开启Query cache也是类似。会对每个ID产生一条查询语句。

* Cache limitations when used in conjunction with `@Inheritance`

* Cache settings get ignored when using a singleton based cache

----
## Reference
Hibernate ORM Tips, Tricks, and Performance Techniques <br />
http://blog.jhades.org/setup-and-gotchas-of-the-hibernate-second-level-and-query-caches/


