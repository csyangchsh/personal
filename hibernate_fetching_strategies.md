#Hibernate Fetching Strategies

* 什么时刻加载？(timing)
* 用什么样的方式加载？(style)

###When

####Eager

立即加载，会load大量、很深的关联关系。

####Lazy

* **第一次**访问的时候加载
* 对于集合是默认的加载方式 "persistent collections"
* 简单属性需要字节码增强，没有特别大的好处
* to-one关联
	* 访问的时候加载
	* Proxy
		* 默认方式
		* 访问时加载（不包括ID）
	* Non-Proxy
		* 访问时加载（包括ID）
		* 需要字节码增强

#### Extra Lazy

* 只针对集合
* 访问的时候加载单个元素
* 不会加载整个集合

###Style

####Join

* Left/Outer Join
* to-one关联使用Join很好
* 不要Join多个集合，会产生笛卡尔集。

####Batch

* Fetching up to ‘N’ collections or entities, **Not record**.
* Fetches multiple entries when one is accessed

####Select

* 集合的默认加载方式，避免笛卡尔集。
* N+1

####SubSelect

* Group collection into a sub select statement.
* Fetches all collection entries when accessed for the 1st time.

###Links

http://www.mkyong.com/hibernate/hibernate-fetching-strategies-examples/