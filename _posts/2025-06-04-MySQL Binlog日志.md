---
layout: post
title: 第023章-MySQL Binlog日志
categories: [MySQL]
description: 
keywords: MySQL Binlog日志.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# MySQL BinLog 日志

MySQL 在完成一条更新操作后，Server 层还会生成一条 binlog，等之后事务提交的时候，会将该事物执行过程中产生的所有 binlog 统一写 入 binlog 文件。binlog主要用于恢复数据库和同步数据库。

binlog 文件是记录了所有数据库表结构变更（例如create、alter table）和表数据修改（insert、update、delete）的日志，不会记录查询类的操作，比如 SELECT 和 SHOW 操作。

最开始 MySQL 里并没有 InnoDB 引擎，MySQL 自带的引擎是 MyISAM，但是 MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档。而 InnoDB 是另一个公司以插件形式引入 MySQL 的，既然只依靠 binlog 是没有 crash-safe 能力的，所以 InnoDB 使用 redo log 来实现 crash-safe 能力。

主从数据库同步用到的都是BinLog文件。



> 如果执行更新SQL时数据没有发生变化（MySQL会在更新时做判断），则不会记录binlog日志。



## BinLog模式

BinLog日志文件有三种模式。



### STATEMENT 模式

每一条修改数据的 SQL 都会被记录到 binlog 中（相当于记录了逻辑操作，所以针对这种格式， binlog 可以称为逻辑日志），主从复制中 slave 端再根据 SQL 语句重现。但 STATEMENT 有动态函数的问题，比如你用了 uuid 或者 now 这些函数，你在主库上执行的结果并不是你在从库执行的结果，这种随时在变的函数会导致复制的数据不一致；

`内容`：binlog 只会记录可能引起数据变更的 sql 语句

`优势`：该模式下，因为没有记录实际的数据，所以日志量和 IO 都消耗很低，性能是最优的

`劣势`：但有些操作并不是确定的，比如 uuid() 函数会随机产生唯一标识，当依赖 binlog 回放时，该操作生成的数据与原数据必然是不同的，此时可能造成无法预料的后果。



### ROW 模式

记录行数据最终被修改成什么样了（这种格式的日志，就不能称为逻辑日志了），不会出现 STATEMENT 下动态函数的问题。但 ROW 的缺点是每行数据的变化结果都会被记录，比如执行批量 update 语句，更新多少行数据就会产生多少条记录，使 binlog 文件过大，而在 STATEMENT 格式下只会记录一个 update 语句而已；

在该模式下，binlog 会记录每次操作的更新前和更新后的目标数据，StreamSets就要求该模式。

优点在于可以绝对精准的还原，从而保证了数据的安全与可靠，并且复制和数据恢复过程可以是并发进行的。缺点在于 binlog 体积会非常大，同时，对于修改记录多、字段长度大的操作来说，记录时性能消耗会很严重。阅读的时候也需要特殊指令来进行读取数据。



### MIXED 模式

包含了 STATEMENT 和 ROW 模式，它会根据不同的情况自动使用 ROW 模式和 STATEMENT 模式；

`内容`：是对上述STATEMENT 跟 ROW  两种模式的混合使用。

`细节`：对于绝大部分操作，都使用 STATEMENT 来进行 binlog 的记录，只有以下操作使用 ROW 来实现：表的存储引擎为 NDB，使用了uuid() 等不确定函数，使用了 insert delay 语句，使用了临时表



## binlog刷盘

事务执行过程中，先把日志写到 binlog cache（Server 层的 cache），事务提交的时候，再把 binlog cache 写到 binlog 文件中。

MySQL 给 binlog cache 分配了一片内存，每个线程一个，参数 binlog_cache_size 用于控制单个线程内 binlog cache 所占内存的大小。如果超过了这个参数规定的大小，就要暂存到磁盘。



在事务提交的时候，执行器把 binlog cache 里的完整事务写入到 binlog 文件中，并清空 binlog cache。如下图：

![image-20220725211411126](https://oss.xubighead.top/oss/image/202506/1930204879671889921.png)



## 相关参数

### sql_log_bin

```sql
mysql> show variables like '%sql_log_bin%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sql_log_bin   | ON    |
+---------------+-------+
1 row in set (0.01 sec)
```



`sql_log_bin`表示是否开启binlog日志。



### sync_binlog

```sql
mysql> show variables like '%sync_binlog%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sync_binlog   | 1     |
+---------------+-------+
1 row in set (0.01 sec)
```



MySQL提供一个 sync_binlog 参数来控制数据库的 binlog 刷到磁盘上的频率：

- sync_binlog = 0 的时候，表示每次提交事务都只 write，不 fsync，后续交由操作系统决定何时将数据持久化到磁盘；
- sync_binlog = 1 的时候，表示每次提交事务都会 write，然后马上执行 fsync；
- sync_binlog =N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

在MySQL中系统默认的设置是 sync_binlog = 0，也就是不做任何强制性的磁盘刷新指令，这时候的性能是最好的，但是风险也是最大的。因为一旦主机发生异常重启，在 binlog cache 中的所有 binlog 日志都会被丢失。

而当 sync_binlog 设置为 1 的时候，是最安全但是性能损耗最大的设置。因为当设置为1的时候，即使主机发生异常重启，也最多丢失 binlog cache 中未完成的一个事务，对实际数据没有任何实质性影响，就是对写入性能影响太大。

如果能容少量事务的 binlog 日志丢失的风险，为了提高写入的性能，一般会 sync_binlog 设置为 100~1000 中的某个数值。



### binlog_group_commit_sync_delay

```sql
mysql> show variables like 'binlog_group_commit_sync_delay';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| binlog_group_commit_sync_delay | 0     |
+--------------------------------+-------+
1 row in set (0.00 sec)
```



`binlog_group_commit_sync_delay`参数用于控制binlog日志写入到binlog日志文件后，等待刷盘的时间。超过这个时间后才会进行刷盘的操作，目的是为了组合更多事务的binlog，再一起刷盘，提高数据库性能。



### binlog_group_commit_sync_no_delay_count

```sql
mysql> show variables like 'binlog_group_commit_sync_no_delay_count';
+-----------------------------------------+-------+
| Variable_name                           | Value |
+-----------------------------------------+-------+
| binlog_group_commit_sync_no_delay_count | 0     |
+-----------------------------------------+-------+
1 row in set (0.01 sec)
```



`binlog_group_commit_sync_no_delay_count`参数用于控制binlog日志组提交刷盘的事务数量，在等待刷盘的过程中，如果binlog事务的数量达到设置的数值，就不再等待，马上进行刷盘操作。