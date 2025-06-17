---
layout: post
title: MySQL UndoLog日志.md
categories: [MySQL]
description: MySQL
keywords: MySQL
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# MySQL UndoLog日志

在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为 Undo Log），然后进行数据的修改。如果出现了错误或者用户执行了 Rollback 语句，系统可以利用 Undo Log 中的备份将数据恢复到事务开始之前的状态。**通俗易懂地说，它只关心过去的数据。**

除了记录redo log外，当进行数据修改时还会记录undo log，undo log用于数据的撤回操作，它保留了记录修改前的内容。通过undo log可以实现事务回滚，并且可以根据undo log回溯到某个特定的版本的数据，实现MVCC。



- Undo log是InnoDB MVCC事务特性的重要组成部分。当我们对记录做了变更操作时就会产生undo记录，Undo记录默认被记录到系统表空间(ibdata)中，但从5.6开始，也可以使用独立的Undo 表空间。

- Undo记录中存储的是老版本数据，当一个旧的事务需要读取数据时，为了能读取到老版本的数据，需要顺着undo链找到满足其可见性的记录。当版本链很长时，通常可以认为这是个比较耗时的操作（例如bug#69812）。

- 大多数对数据的变更操作包括INSERT/DELETE/UPDATE，其中INSERT操作在事务提交前只对当前事务可见，因此产生的Undo日志可以在事务提交后直接删除（谁会对刚插入的数据有可见性需求呢！！），而对于UPDATE/DELETE则需要维护多版本信息，在InnoDB里，UPDATE和DELETE操作产生的Undo日志被归成一类，即update_undo

- 另外, 在回滚段中的undo logs分为: `insert undo log` 和 `update undo log`

  - insert undo log : 事务对insert新记录时产生的undolog, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。
  - update undo log : 事务对记录进行delete和update操作时产生的undo log, 不仅在事务回滚时需要, 一致性读也需要，所以不能随便删除，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被purge线程删除




我们在执行执行一条“增删改”语句的时候，虽然没有输入 begin 开启事务和 commit 提交事务，但是 MySQL 会**隐式开启事务**来执行“增删改”语句的，执行完就自动提交事务的，这样就保证了执行完“增删改”语句后，我们可以及时在数据库表看到“增删改”的结果了。

执行一条语句是否自动提交事务，是由 `autocommit` 参数决定的，默认是开启。所以，执行一条 update 语句也是会使用事务的。

那么，考虑一个问题。一个事务在执行过程中，在还没有提交事务之前，如果MySQL 发生了崩溃，要怎么回滚到事务之前的数据呢？

如果我们每次在事务执行过程中，都记录下回滚时需要的信息到一个日志里，那么在事务执行中途发生了 MySQL 崩溃后，就不用担心无法回滚到事务之前的数据，我们可以通过这个日志回滚到事务之前的数据。

实现这一机制就是 undo log（回滚日志），它保证了事务的 ACID 特性中的原子性（Atomicity）。

undo log 是一种用于撤销回退的日志。在事务没提交之前，MySQL 会先记录更新前的数据到 undo log 日志文件里面，当事务回滚时，可以利用 undo log 来进行回滚。如下图：

![回滚事务](https://oss.xubighead.top/oss/image/202506/1930158049529532417.png)

每当 InnoDB 引擎对一条记录进行操作（修改、删除、新增）时，要把回滚时需要的信息都记录到 undo log 里，比如：

- 在**插入**一条记录时，要把这条记录的主键值记下来，这样之后回滚时只需要把这个主键值对应的记录**删掉**就好了；
- 在**删除**一条记录时，要把这条记录中的内容都记下来，这样之后回滚时再把由这些内容组成的记录**插入**到表中就好了；
- 在**更新**一条记录时，要把被更新的列的旧值记下来，这样之后回滚时再把这些列**更新为旧值**就好了。

在发生回滚时，就读取 undo log 里的数据，然后做原先相反操作。比如当 delete 一条记录时，undo log 中会把记录中的内容都记下来，然后执行回滚操作的时候，就读取 undo log 里的数据，然后进行 insert 操作。

不同的操作，需要记录的内容也是不同的，所以不同类型的操作（修改、删除、新增）产生的 undo log 的格式也是不同的，具体的每一个操作的 undo log 的格式我就不详细介绍了，感兴趣的可以自己去查查。

一条记录的每一次更新操作产生的 undo log 格式都有一个 roll_pointer 指针和一个 trx_id 事务id：

- 通过 trx_id 可以知道该记录是被哪个事务修改的；
- 通过 roll_pointer 指针可以将这些 undo log 串成一个链表，这个链表就被称为版本链；

版本链如下图：

![版本链](https://oss.xubighead.top/oss/image/202506/1930158113807241218.png)

另外，**undo log 还有一个作用，通过 ReadView + undo log 实现 MVCC（多版本并发控制）**。

对于「读提交」和「可重复读」隔离级别的事务来说，它们的快照读（普通 select 语句）是通过 Read View + undo log 来实现的，它们的区别在于创建 Read View 的时机不同：

- 「读提交」隔离级别是在每个 select 都会生成一个新的 Read View，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
- 「可重复读」隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View，这样就保证了在事务期间读到的数据都是事务启动前的记录。

这两个隔离级别实现是通过「事务的 Read View 里的字段」和「记录中的两个隐藏列（trx_id 和 roll_pointer）」的比对，如果不满足可见行，就会顺着 undo log 版本链里找到满足其可见性的记录，从而控制并发事务访问同一个记录时的行为，这就叫 MVCC（多版本并发控制）。

因此，undo log 两大作用：

- **实现事务回滚，保障事务的原子性**。事务处理过程中，如果出现了错误或者用户执 行了 ROLLBACK 语句，MySQL 可以利用 undo log 中的历史数据将数据恢复到事务开始之前的状态。
- **实现 MVCC（多版本并发控制）关键因素之一**。MVCC 是通过 ReadView + undo log 实现的。undo log 为每条记录保存多份历史数据，MySQL 在执行快照读（普通 select 语句）的时候，会根据事务的 Read View 里的信息，顺着 undo log 的版本链找到满足其可见性的记录。



## undolog日志版本链

MySQL 事务开始后会生成一个全局唯一的事务 ID，并且每一行数据其实都有两个隐藏字段 trx_id（最近更新这条数据的事务 ID）和 roll_pointer（undo log 编号），所以如果一个事务更新某行数据时，会把自己的事务 ID 写入这行数据的 trx_id，这行数据的 roll_pointer 会指向最近的 undo log，如此一来，这行数据在多事务场景下修改时，它的 undo log 就会形成一个 undo log 版本链作为这行数据在每个事务中的快照。

比如首先事务 A 插入一行数据值 A，同时生成 insert undo log，在事务 A 提交后，刚才生成的 undo log 就会直接被删除。

![图片](https://oss.xubighead.top/oss/image/202506/1930158174159081474.jpg)



紧接着事务 B 去修改这行数据为值 B，同时生成 update undo log，在事务 B 提交后，刚才生成的 undo log 不会立即删除。

![图片](https://oss.xubighead.top/oss/image/202506/1930158188608458753.jpg)



然后事务 C 把这行数据删除了，但是删除操作并不会直接删除数据行，而是改变数据行中 delete 标记位，标记这行数据是删除状态。

![图片](https://oss.xubighead.top/oss/image/202506/1930158206174203906.jpg)



等待 purge 线程在适当时机删除这行数据的 undo log 版本链。

![图片](https://oss.xubighead.top/oss/image/202506/1930158220841684993.jpg)



## undolog日志删除

没有其它事物线程还在使用当前版本的undo时候，purge进程进行回收。即系统里没有比这个回滚日志更早的 read-view 的时候，这个数据不会再有谁驱使它回滚，就可以删除了。

长事务意味着系统里面会存在很老的事务视图。由于这些事务随时可能访问数据库里面的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间。因此尽量不要使用长事务。



## 相关参数

### innodb_undo_tablespaces

```sql
mysql> show variables like 'innodb_undo_tablespaces';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_undo_tablespaces | 2     |
+-------------------------+-------+
1 row in set (0.01 sec)
```



innodb_undo_tablespaces是控制undo是否开启独立的表空间的参数，为0表示：undo使用系统表空间，即ibdata1，不为0表示：使用独立的表空间，一般名称为 undo001 undo002。一般innodb_undo_tablespaces 默认配置为0。



### innodb_undo_directory

```sql
mysql> show variables like 'innodb_undo_directory';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_undo_directory | ./    |
+-----------------------+-------+
1 row in set (0.01 sec)
```



`innodb_undo_directory` 存放地址的配置项，默认配置为当前数据目录。