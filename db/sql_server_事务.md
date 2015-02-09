# SQL Server 事务

## Transaction

Transaction is complete unit of work. 

* (A) – Atomicity or “all or nothing”. Either all changes are saved or nothing changed. 不允许提交不完整的事务。
* (C) – Consistency. Data remains in consistent stage all the time. 不管事务是成功还是失败，都要保证数据库处于一致的状态。
* (I) – Isolation. Other sessions don’t see the changes until transaction is completed. Well, this is not always true and depend on the implementation. 可以改变隔离性的行为。
* (D) – Durability. Transaction should survive and recover from the system failures. 事务提交后，必须保持提交的状态。(**write-ahead logging**)

a few common myths about transactions:

* There are no transactions if you call insert/update/delete statements without begin tran/commit statements. Not true. In such case SQL Server starts **implicit transaction for every statement**. It’s not only violate consistency rules in a lot of cases, it’s also extremely expensive. Try to run 1,000,000 insert statements within explicit transaction and without it and notice the difference in execution time and log file size. (**TODO** test the difference)
* There is no transactions for select statements. Not true. SQL Server uses (lighter) transactions with select statements.
* There is no transactions when you have (NOLOCK) hint. Not true. (NOLOCK) hint downgrades the reader to read uncommitted isolation level but transactions are still in play.

Each transaction starts in specific transaction isolation level. 

There are 4 “pessimistic” isolation levels: **Read uncommitted, read committed, repeatable read and serializable** and 2 “optimisitic” isolation levels: **Snapshot and read committed snapshot**. 

With pessimistic isolation levels writers always block writers and typically block readers (with exception of read uncommitted isolation level). 

With optimistic isolation level writers don’t block readers and in snapshot isolation level does not block writers (there will be the conflict if 2 sessions are updating the same row).

## 并发访问后果

### 丢失更新 (lost update)

在2个进程读取同一个数据然后试图用一个不同的值更新数据。

可以通过显示地提高隔离级别来避免。

### 脏读 (dirty read)

脏读不管其他进程的锁。当读取未提交的数据时，这个读操作会被标记为脏读。如果未提交的事务失败或者回滚了，就会出现问题。

### 不可重复读 (not-reaptable read)

在同一个事务中，不能保证2次读取的数据是一致的。

根据隔离级别的不同，另一个事务可能在这个事务的2次读之间对数据做了更新。**在较低的隔离级别，数据锁定只发生在读数据的时间段内，而不是整个事务的时间段内。**

### 幻影读 (phantom read)

一个事务对一个范围内的数据插入或删除了行，而这个范围的数据正在被另一个事务读取。

### 双重读 (double read)

在一个查询在某个表上的一个范围内进行扫描的同时另一个事务对行进行了移动操作，导致第一个查询对这行数据读取了2次。

读数据锁在数据成功读取后就立即释放了，可以通过提高隔离级别避免。

## 悲观隔离级别

### 未提交读 (read uncommitted)

SQL Server的最低级别的隔离，允许脏读，不可重复读和幻影读。

### 已提交读 (read committed)

SQL Server的默认隔离级别，允许发生不可重复读和幻影读。读取锁只在读取数据的期间起作用，而不是整个事务的期间。

### 可重复读 (repeatable read)

保证在同一个事务内对数据的多次读取结果是一致的。锁必须持续整个事务期间，可以防止其他用户在事务期间修改读取的数据，但是不能防止插入或删除行。

### 可序列化 (serializable)

最高级别的悲观隔离级别，不允许脏读、不可重复读，幻影读。对并发性能影响最大。
****
'x' 表示不允许出现

|脏读 | 不可重复读 | 幻影读 |
|-----|----------|-------|
|未提交读| | | |
|提交读| x | | |
|可重复读| x | x | |
|可序列化| x | x | x |

## 锁与隔离级别

排他锁总是持有到事务结束，和隔离级别无关。共享锁的处理方式如下表:

|隔离级别|共享锁行为|Table Hint|
|-------|--------|----------|
|未提交读|不请求共享锁| NOLOCK |
|提交读|请求，立即释放|READCOMMITTED|
|可重复读|持有到事务结束|REPEATABLEREAD|
|可序列化|范围锁(Range Lock)持有到事务结束|HOLDLOCK|

## References

http://aboutsqlserver.com/lockingblocking/ <br />
SQL Server 2008 内核剖析与故障排除
