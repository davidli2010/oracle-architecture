# 数据库表

## 表类型

在Oracle中主要有以下9种表类型：
- 堆组织表（heap organized table）：这就是普通的标准数据库表。数据在其中以堆的方式管理。增加数据时，会找到并使用段中第一个能放下此数据的自由空间。从表中删除数据时，则允许以后的INSERT和UPDATE重用这部分空间。这就是这种表类型中“堆”这个名字的由来。“堆”是指一组空间，被以某种随机的方式来使用。
- 索引组织表（index organized table）：这些表按索引结构存储。这就强制要求行本身有某种物理顺序。在堆表中，只要空间放得下，数据可以放在其中任何位置；而索引组织表（IOT）则不同，数据要根据主键有序地存储在IOT中。
- 索引聚簇表（index clustered table）：聚簇（cluster）是指一个或多个表组成的组，这些表中的数据物理地存储在相同的数据库块上，有相同聚簇键值的所有行会物理存储在相邻的位置。这种结构可以实现两个目标。首先，多个表可以被物理地存储在一起。一般而言，你可能认为一个数据库块中只包含一个表的数据，但是对于聚簇表，可能把多个表的数据存储在同一个块上。其次，包含相同聚簇键值的所有数据会物理地存储在一起。因此数据会按聚簇键值聚簇在一起，而聚簇键是使用B*Tree索引建立的。索引聚簇表的优点是当频繁访问聚簇键连接的多个表时，能减少磁盘I/O并提高查询性能。
- 散列聚簇表（hash clustered table）：这些表类似于索引聚簇表，但不是使用B*Tree索引按聚簇键来定位数据。散列聚簇是将键值散列到聚簇上，从而直接查找到数据应该在哪个数据库块上。在散列聚簇中，数据就是索引。如果需要频繁的通过键的相等性比较来读取数据，散列聚簇表就很适用。
- 有序散列聚簇表（sorted hash clustered table）：这种表类型是Oracle10g新增的。它结合了散列聚簇表和IOT的一些特点。其概念如下：如果你的行按某个键值进行散列，而与该键相关的一系列记录会以某种有序顺序被写入表，并被按这种顺序进行处理。
- 嵌套表（nested table）：嵌套表是Oracle对象关系扩展的一部分。它们实际上就是系统生成和维护的父/子关系中的子表。
- 临时表（temporary table）：这些表存储的是事务期间或会话期间的“草稿”数据。临时表要根据需要从当前用户的临时表空间分配临时区段。每个会话只能看到本会话分配的区段，而不会看到其他任何会话创建的任何数据。可以使用临时表来存储一些临时性的数据，这样做的好处是：与常规的堆表相比，临时表上redo的生成量大幅降低。
- 对象表(object table)：对象表是基于某种对象类型创建的。它们拥有非对象表所没有的特殊属性。对象表是堆组织表、索引组织表和临时表的一种特殊类型，Oracle会用这些种类的表甚至嵌套表来构建对象表。另外，嵌套表也是对象表结构中的一种。
- 外部表（external table）：这些表中的数据并不存储在数据库中，而是放在数据库之外，也就是说它们被存放为普通的操作系统文件。在Oracle9i及以上版本中，可以利用外部表查询数据库以外的文件，就好像这个文件也是数据库中的一个普通的表一样。

不论哪种类型的表，都会有以下基本信息：
- 一个表最多可以有1000列。如果一行数据包含超过254列，Oracle就会在内部把它存储成多个单独的行段，这些行段相互指向，而在使用这个行时，则必须把它们重新组装为完整的行映像。
- 一个表的行数理论上是无限的，不过因为存在着另外某些限制，使得它实际上无法达到“无限”。
- 表中的列有多少种排列，表就可以有多少个索引。
- 即使在一个数据库中也可以有无限多个表。

## 术语

### 段

Oracle中的段（segment）是占用磁盘上存储空间的一个对象。段有很多种类型，下面列出的是最常见的段类型：
- 聚簇（cluster）：这种段类型能存储多个表。聚簇分为两种：B*Tree聚簇和散列聚簇。聚簇通常用于存储多个表上的相关数据，将其预先连接存储到同一个数据库块上；还可以用于存储一个表的相关信息。“聚簇”这个词是指这个段能把相关的信息物理地聚在一起。
- 表（table）：一个表段保存一个数据库表的数据，这可能是最常用的段类型，通常与索引段联合使用。
- 表分区（table partition）或子分区（subpartition）：这种段类型用于分区，与表段很相似。一个表分区或子分区中存储了一个表中的一部分数据。一个分区表由一个或多个表分区段（table partition segment）组成，组合分区表则由一个或多个表子分区段（table subpartition segment）组成。
-索引（index）：这种段类型可以保存索引结构。
- 索引分区（index partition）：类似于表分区，这种段类型包含一个索引的一部分。分区索引由一个或多个索引分区段（index partition segment）组成。
- LOB分区（lob partition）、LOB子分区（lob subpartition）、LOB索引（lobindex）和LOB段(lobsegment)：LOB索引和LOB段类型可以保存大对象（large object， LOB）的结构。对包含LOB的表分区时，LOB段也会被分区，LOB分区段（lob partition segment）正是用于此。
- 嵌套表（nested table）：这是为嵌套表指定的段类型，它是父/子关系中一种特殊类型的子表。
- 回滚（rollback）段和“Type2 undo”段：被撤销的数据就存储在这里。回滚段是DBA手动创建的段。Type2 undo段则是由Oracle自动创建和管理的。

### 段空间管理

从Oracle9i开始，管理段空间有下面两种方法：
- 手动段空间管理（Manual Segment Space Management, MSSM）：由你设置FREELISTS、FREELIST GROUPS、PCTUSED和其他参数来控制如何分配、使用和重用段中的空间。
- 自动段空间管理（Automatic Segment Space Management, ASSM）：只需控制与空间使用相关的一个参数PCTFREE。

### 高水位线

如果把表想象成一个平面结构，或者想象成从左到右依次排开的一系列块，高水位线（High-Water Mark， HWM）就是曾经包含过数据的最右边的块。

HWM很重要，因为Oracle在全面扫描段时会扫描HWM之下的所有块，即使其中不包含任何数据。它会影响全面扫描的性能，特别是当HWM之下的大多数块都为空时。

### FREELIST

使用MSSM表空间时，针对那些有自由空间的对象，Oracle会使用自由列表（FREELIST）来维护它们的HWM之下的块。

每个对象都至少有一个相关的FREELIST，使用块时，可能会根据需要把块放在FREELIST上或者从FREELIST上去除。需要说明的重要一点是，只有位于HWM以下的对象块才会出现在FREELIST中。仅当FREELIST为空时才会使用HWM之上的块，此时Oracle会推进HWM，并把这些块增加到FREELIST中。采用这种方式，Oracle会延迟到不得已时才推进对象的HWM。

一个对象可以有多个FREELIST。如果预料到会有很多并发用户在一个对象上执行大量的INSERT或UPDATE，就可以配置多个FREELIST，这对性能提升很有好处。

### PCTFREE和PCTUSED

一般而言，PCTFREE参数用来告诉Oracle应该在块上保留多少空间来应对将来的更新。默认情况下，这个值是10%。如果一个块上的自由空间的比例高于PCTFREE中指定的值，那这个块就被认为是“自由的”。PCTUSED则是用来告诉Oracle，当前一个不“自由”的块上的自由空间比例重新达到多大时，才能使这个块再次变为自由的。默认值是40%。

行最初很小，而现在需要将行的大小扩大一倍。这种情况下，倘若PCTFREE设置的太小，更新行时就会导致行迁移。

#### 行迁移

行迁移（row migration）是指由于某一行变得太大，无法再与块中其余的行放在创建这一行的块中，这就会要求这一行离开原来的块。Oracle不能简单地移动这一行，它必须留下一个转发地址。因为可能有一些索引物理地指向原来的地址，而简单的更新并不会同时修改这些索引。因此，Oracle迁移这一行时，她会留下一个指针，标识这一行实际是在什么位置。

### INITRANS和MAXTRANS

段中每个块都有一个块首部。这个块首部中有一个事务表。事务表中会建立一些条目来描述哪些事务将块上的哪些行/元素锁定。这个事务表的初始大小由对象的INITRANS设置指定（表和索引上的默认值为2）。事务表会根据需要动态扩展，最大达到MAXTRANS个条目（假设块上有足够的自由空间）。所分配的每个事务条目需要占用块首部中的23~24字节的存储空间。

## 堆组织表

应用中99%的情况下使用的可能都是堆组织表（heap organized table）。执行CREATE TABLE语句时，默认得到的表类型就是堆组织表。

堆（heap）是计算机科学领域得到深入研究的一种经典数据结构。它实际上就是一个很大的空间、磁盘或内存区，并被以一种很随机的方式管理。数据会放在适合放下它的地方，而不是以某种特定顺序来放置。

应该把堆组织表看作一个很大的无序行集合。这些行会以一种看来随机的顺序被取出，而且取出的顺序还取决于所用的其他选项（并行查询、不同的优化器模式，等等），同一个查询可能会以不同的顺序取出数据。不要过分依赖查询得到的行顺序，除非查询中有一个ORDER BY子句。

## 索引组织表

索引组织表（Index Organized Table， IOT）就是存储在一个索引结构中的表。存储在堆表中的数据是无组织的，IOT中的数据则是按主键顺序来存储和排序的。

## 索引聚簇表

聚簇（cluster）是指：如果一组表有一些共同的列，则将这样一组表存储在相同的数据库块中；聚簇还会把相关的数据存储在同一个块上。利用聚簇，一个块可能包含多个表的数据。从概念上讲，这是将数据“预连接”地存储。聚簇还可以用于单个表，可以按某个列将数据分组存储。

## 散列聚簇表

散列聚簇表（hash clustered table）在概念上与索引聚簇表非常相似，只有一个主要区别：聚簇键索引被一个散列函数所取代。数据就是索引。聚簇键被散列到一个块地址上，而数据应该就在那个位置上。
- 散列聚簇一开始就要分配空间。
- 散列聚簇中的HASHKEY数是固定的。除非重建聚簇，否则不能改变散列表的大小。这并不会限制聚簇中能存储的数据量，它只是限制了能为这个聚簇生成的唯一散列键的数量。
- 不能在聚簇键上进行区间扫描。

## 有序散列聚簇表

有序散列聚簇（sorted hash cluster）不仅有前面所述的散列聚簇的有关性质，还结合了IOT的一些性质。

要按某个键获取数据，但要求这些数据按另外某个列排过序，使用类似的查询来获取数据，有序散列聚簇就最为合适。通过使用有序散列聚簇，Oracle可以返回数据而根本不用执行排序。这是通过插入时按键的有序顺序物理存储数据做到的。

## 嵌套表

嵌套表（nested table）是Oracle对象关系扩展的一部分。嵌套表是Oracle中的两种集合类型之一，它与关系模型中传统的“父子表对”里的子表很相似。嵌套表是数据元素的无序集合，集合中所有数据元素的数据类型都相同，可以是一个内置数据类型，也可以是一个对象数据类型。

## 临时表

可以使用临时表（temporary table）来保存事务或会话内的临时结果集。临时表中保存的数据只对当前会话可见，所有会话都看不到其它会话的临时表中的数据；即使当前会话提交了事务，别的会话也看不到临时表中的数据。临时表不存在多用户并发问题，因为一个会话不会因为使用临时表而阻塞另一个会话。即使“锁住”了临时表，也不会妨碍其它会话使用它们自己的临时表。

临时表会从当前登录用户的临时表空间分配存储空间。如果存储过程使用了临时表，而且这个存储过程为定义者权限的，那么存储过程在执行过程中会使用存储过程所有者的临时表空间。

Oracle的临时表是“静态”定义的。在数据库中只需要创建一次就可以了，无需在存储过程中让它每次运行的时候都创建一次临时表。临时表在Oracle数据库中是在使用前预先创建好的，其定义放在数据字典中，如果会话不往临时表中存放数据，那么它在会话中就一直是个空表。由于临时表是静态定义的，所以我们可以在创建视图时直接使用临时表，而且在存储过程中的静态SQL也能用到临时表。

临时表可以是基于会话的（临时表中的数据在提交之后不会被清空，但是在断开连接后再连接时部分数据就没有了），也可以是基于事务的（提交之后数据就消失）。

临时表与永久表有许多共同之处，它们都可以有触发器、检查约束、索引等。但永久表的某些特性在临时表中并不支持。例如下面的特性：
- 不能有引用完整性约束。临时表既不能作为外键关系中的父表，也不能在临时表中定义外键。
- 不能有NESTED TABLE类型的列。
- 不能是IOT。
- 不能在任何类型的聚簇中。
- 不能分区。
- 不能通过ANALYZE表命令生成统计信息。

## 对象表

对象表（object table）是一种基于Oracle类型（TYPE）的表，其定义不是基于列来创建的。一般的表其CREATE TABLE语句类似这样：
```
create table t(x int, y date, z varchar2(25));
```
而创建对象表则稍有不同：
```
create table t of Some_Type;
```
表T的属性（列）是由SOME_TYPE定义的。
