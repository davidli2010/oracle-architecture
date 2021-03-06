# 并发与多版本控制

## 什么是并发控制

并发控制（concurrency control）是数据库提供的一系列功能，它允许许多人同时访问和修改数据。锁（lock）是Oracle管理共享数据库资源的并发访问并防止并发数据库事务之间”相互干涉“的核心机制之一。

Oracle使用了多种锁，在此总结为如下几种类型：
- TX（事务处理）锁：修改数据的事务在执行期间会获得这种锁。
- TM（DML队列）锁和DDL锁：在你修改一个对象的内容（对应TM锁）或对象本身（对应DDL锁）时，这些锁可以确保对象的结构不被其它用户修改。
- 闩（latch）和互斥锁（mutext）：这是Oracle的内部锁，用来协调对其共享数据结构的访问。

Oracle对并发的支持不只是高效的锁定，它还实现了一种多版本控制（multiversioning）体系结构，这种体系结构提供了一种受控但高度并发的数据访问。多版本控制是指，Oracle能同时物化多个版本的数据，这也是Oracle提供数据读一致的基础。

默认情况下，Oracle的读一致性多版本适用于语句级，我们也可以将其调整为事务级。

## 事务隔离级别

ANSI/ISO SQL标准定义了4种事务隔离级别，对于相同的事务，采用不同的隔离级别分别有不同的结果。也就是说，如果事务的隔离级别不同，就算是事务的输入相同，而且采用同样的方式来完成同样的工作，也可能得到完全不同的答案。这些隔离级别是根据3种”现象“定义的，以下就是隔离级别可能允许或不允许的3种现象。
- 脏读（dirty read）：能读取未提交的数据，也就是脏数据。脏读会影响数据的完整性，另外外键约束会遭到破坏，而且会忽略唯一性约束。
- 不可重复读（nonrepeatable read）：如果你在T1时间读取某一行，在T2时间重新读取这一行时，这一行可能已经有所修改，也许它已经消失，也可能被更新了，等等。此时得到的结果也会与之前不同。
- 幻读（phantom read）：如果你在T1时间执行一个查询，而在T2时间再执行这个查询，此时可能数据库中新增了一些行而影响你的结果。与不可重复读的区别在于：在幻读中，已经读取的数据没发生变化，T2比T1有更多的数据满足你的查询条件。

SQL隔离级别是根据是否允许上述各个现象而定义的。
|隔离级别|脏读|不可重复读|幻读|
|-------|----|---------|---|
|READ UNCOMMITTED|允许|允许|允许|
|READ COMMITTED|-|允许|允许|
|REPEATABLE READ|-|-|允许|
|SERIALIZABLE|-|-|-|

Oracle明确地支持READ COMMITTED和SERIALIZABLE隔离级别。

Oracle没有使用脏读这样的机制，它甚至不允许脏读。

## 读一致性

Oracle可以使用undo信息来提供非阻塞的查询和一致的读。查询时，Oracle会从缓冲区缓存中读出块，它能保证这个块版本足够”旧“，能够被该查询看到。

## 写一致性

不能修改块的老版本，修改一行时，必须修改该块的当前版本。

Oracle处理修改语句时会进行两种读取：
- 一致读（consistent read）：”发现“要修改的行。
- 当前读（current read）：取得数据块来进行真正的修改。

要修改Y=5的行，如果开始时Y=5的行现在包含值Y=10，Oracle会悄悄地回滚这条UPDATE语句（仅回滚更新，不会回滚事务的任何其它部分），然后再运行这个UPDATE语句（假设使用的是READ COMMITTED隔离级别）。如果你使用了SERIALIZABLE隔离级别，此时这个事务就会收到一个ORA-08177: can't serialize access for this transaction错误。而在READ COMMITTED模式下，事务回滚更新后，数据库会重启更新操作（也就是说，修改UPDATE开始的时间点），但是它不是直接运行这个UPDATE语句，而是先进入SELECT FOR UPDATE模式，并试图为会话锁住所有WHERE Y=5的行。当所有数据锁定之后，它才会真正地重新运行UPDATE语句，这样可以确保这一次就能完成而不必（再次）重启动。