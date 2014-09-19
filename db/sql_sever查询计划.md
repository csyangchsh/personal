# SQL Sever查询计划

## 查看查询计划

* SSMS中的图形化计划
* `set showplan_text[showplan_xml|showplan_all] on`
* `set statistics profile on`
* 计划缓存
	
	```
	select *
	from sys.dm_exec_query_plan(plan_handle) --XML
	
	select *
	from sys.dm_exec_text_query_plan(plan_handle) --text
	```
	* sql_handle 原始T-SQL源码的hash值
	* plan_handle 缓存的计划的hash值
	
## 查询计划操作符

### 1. 联接操作符

#### 嵌套循环联接 (nested loop join)

适用于小型表，内层表的联接键上有索引。

扫描outer table的所有行，对每一行，扫描inner table的所有行。如果outer table的行和inner table的行有匹配，这一行会加入结果。

#### 合并联接 (merge join)

适用于中等大小的表，表还有有序索引，或者输出需要排序。

需要排好序的输入，可以同时遍历两个表的行，从上至下寻找匹配结果，并且尽可能快的在任何一个范围满足的时候结束。

#### 哈希联接 (hash join)

适用于大中型表，对于并行计划运行良好，精确。

* build phase 扫描较小的表，将所有行放入一个哈希表，每一行都进行哈希计算。
* probe phase 将大表的每一行和和哈希表的内容进行比较。

**TODO** 小型，中型，大型量化指标是什么？ 

### 2. spool 操作符

创建输入流中行的临时副本，并传递给输出流。

* 非聚集索引 spool 从子表中读入行，并放入tempdb，在进一步操作前创建非聚集索引。
* 行计数 spool 不带数据，父操作可以判断行存不存在 (exists, not exists)
* 表 spool 从子表中读取行并写入tempdb

### 3. 扫描和查找操作符

#### 扫描操作符 (scan operator) 

全表扫描，当匹配的行多于整个表的20%时，扫描的性能可以超过直接查找(seek)。

* 聚集索引扫描操作符
* 非聚集索引扫描操作符
* 表扫描操作符

#### 搜索操作符 (seek operator)

使用索引找到匹配行 (单行，一组，一个范围)

* 聚集索引查找操作符
* 非聚集索引查找操作符

### 4. 查找操作符 (lookup operator)

查找单独一行数据

#### 书签查找 (bookmark lookup)

利用聚集索引查找行，只存在于SQL Server 2000及更早版本。

#### 键查找 (key lookup)

在表具有聚集索引的时候返回单独行数据时使用，在SQL Server 2005 SP2引人。

#### RID 查找 (RID lookup)

查找堆表中单独一行数据。（row identifier）

## References

SQL Server 2008 内核剖析与故障分析