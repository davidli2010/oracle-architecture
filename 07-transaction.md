# 事务

事务（transactoin）是数据库区别于文件系统的特性之一。事务会把数据库从一种一致状态转变为另一种一致状态，这就是设计事务的目的。当事务提交时，数据库可以确保要么所有修改都已经保存，要么所有修改都不保存。此外，数据库还能保证提交的事务符合保护数据完整性的各种规则和检查。

Oracle中的事务体现了ACID特性：
- 原子性（Atomicity）：事务中的所有动作要么都发生，要么都不发生。
- 一致性（Consistency）：事务将数据库从一种一致状态转变为下一种一致状态。
- 隔离性（Isolation）：一个事务的影响在该事务提交前对其它事务是不可见的。
- 持久性（Durability）：事务一旦提交，其结果就是永久性的。

## 事务控制语句

- COMMIT：结束事务，并使得已做的所有修改持久地保存在数据库中。
- ROLLBACK：结束事务，并且撤销这个事务所做的修改。
- SAVEPOINT：在事务中创建一个标记点，一个事务中可以有多个SAVEPOINT。
- ROLLBACK TO <SAVEPOINT>：把事务回滚到指定的标记点，但是她不回滚在此标记点之前的工作。
- SET TRANSACTION：设置不同的事务属性，如事务的隔离级别以及事务是只读的还是可读写的。