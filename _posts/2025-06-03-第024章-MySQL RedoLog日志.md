---
layout: post
title: 2025-06-03-第024章-MySQL RedoLog日志.md
categories: [MySQL]
description: 
keywords: MySQL RedoLog日志.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# MySQL RedoLog日志

记录更新时，InnoDB引擎就会先把记录写到RedoLog里面，并更新内存。同时，InnoDB引擎会在空闲时将这个操作记录更新到磁盘里面。如果更新太多RedoLog处理不了的时候，需先将RedoLog部分数据写到磁盘，然后擦除RedoLog部分数据。RedoLog类似转盘。

Redo Log 记录的是新数据的备份（和 Undo Log 相反）。在事务提交前，只要将 Redo Log 持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是 Redo Log 已经持久化。系统可以根据 Redo Log 的内容，将所有数据恢复到崩溃之前的状态。

redo log 是物理日志，记录了某个数据页做了什么修改，对 XXX 表空间中的 YYY 数据页 ZZZ 偏移量的地方做了AAA 更新，每当执行一个事务就会产生这样的一条物理日志。

redo log是innodb引擎级别，用来记录innodb存储引擎的事务日志，不管事务是否提交都会记录下来，用于数据恢复。当数据库发生故障，innoDB存储引擎会使用redo log恢复到发生故障前的时刻，以此来保证数据的完整性。将参数innodb_flush_log_at_trx_commit设置为1，那么在执行commit时会将redo log同步写到磁盘。

**DDL和DML语句都会产生redolog日志。**



<img src="https://oss.xubighead.top/oss/image/202506/1929821630491627521.png" alt="img"  />



## 写入流程

### redo log buffer

redo log 并不是直接写入磁盘的，因为这样会产生大量的 I/O 操作，而且磁盘的运行速度远慢于内存。所以，`redo log` 也有自己的缓存`redo log buffer`，每当产生一条 redo log 时，会先写入到 redo log buffer，后续在持久化到磁盘如下图：

<img src="https://oss.xubighead.top/oss/image/202506/1929823895231238146.png" alt="事务恢复" style="zoom: 33%;" />

`redo log buffer` 默认大小 16 MB，可以通过 `innodb_log_Buffer_size` 参数动态的调整大小，增大它的大小可以让 MySQL 处理大事务时不必写入磁盘，进而提升写 IO 性能。



### 循环顺序写入磁盘

redo log的写入不是直接落到磁盘，而是在内存中设置了一片称之为`redo log buffer`的连续内存空间，也就是`redo 日志缓冲区`。主要有下面几个时机，redo log buffer 的数据会刷入磁盘：

- log buffer 空间不足：当前写入 redo log buffer 的日志量已经占满了 redo log buffer 总容量的一半，就需要把这些日志刷新到磁盘上；
- 事务提交：在事务提交时，为了保证持久性，会把log buffer中的日志全部刷到磁盘；
- 后台线程输入：有一个后台线程，每秒都会刷新一次`redo log buffer`中的`redo log`到磁盘；
- 正常关闭MySQL服务器时；
- **触发checkpoint规则**。



> 事务提交时刷盘场景下，除了本事务的，可能还会刷入其它事务的日志（组提交）；

<img src="https://oss.xubighead.top/oss/image/202506/1929824028484276226.png" alt="img" style="zoom:33%;" />



默认情况下， InnoDB 存储引擎有 1 个重做日志文件组( redo log Group），重做日志文件组由有 2 个 redo log 文件组成，这两个 redo 日志的文件名叫 ：`ib_logfile0` 和 `ib_logfile1` 。

在重做日志组中，每个 redo log File 的大小是固定且一致的，假设每个 redo log File 设置的上限是 1 GB，那么总共就可以记录 2GB 的操作。

重做日志文件组是以**循环写**的方式工作的，从头开始写，写到末尾就又回到开头，相当于一个环形。

所以 InnoDB 存储引擎会先写 ib_logfile0 文件，当 ib_logfile0 文件被写满的时候，会切换至 ib_logfile1 文件，当 ib_logfile1 文件也被写满时，会切换回 ib_logfile0 文件。



![重做日志文件组写入过程](https://oss.xubighead.top/oss/image/202506/1929824114421370882.png)



redo log 是为了防止Buffer Pool 中的脏页丢失而设计的，那么如果随着系统运行，Buffer Pool 的脏页刷新到了磁盘中，那么 redo log 对应的记录也就没用了，这时候擦除这些旧记录，以腾出空间记录新的更新操作。

redo log 是循环写的方式，相当于一个环形，InnoDB 用 write pos 表示 redo log 当前记录写到的位置，用 checkpoint 表示当前要擦除的位置，如下图：

<img src="https://oss.xubighead.top/oss/image/202506/1929824287662903297.png" alt="img" style="zoom: 33%;" />



图中的：

- write pos 和 checkpoint 的移动都是顺时针方向；
- write pos ～ checkpoint 之间的部分（图中的红色部分），用来记录新的更新操作；
- check point ～ write pos 之间的部分（图中蓝色部分）：待落盘的脏数据页记录；

如果 write pos 追上了 checkpoint，就意味着 **redo log 文件满了，这时 MySQL 不能再执行新的更新操作，也就是说 MySQL 会被阻塞**（*因此所以针对并发量大的系统，适当设置 redo log 的文件大小非常重要*），此时**会停下来将 Buffer Pool 中的脏页刷新到磁盘中，然后标记 redo log 哪些记录可以被擦除，接着对旧的 redo log 记录进行擦除，等擦除完旧记录腾出了空间，checkpoint 就会往后移动（图中顺时针）**，然后 MySQL 恢复正常运行，继续执行新的更新操作。

所以，一次 checkpoint 的过程就是脏页刷新到磁盘中变成干净页，然后标记 redo log 哪些记录可以被覆盖的过程。

write pos 是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。

write pos 和 checkpoint 之间的可以用来写入新的操作记录。如果 write pos 追上 checkpoint，这时候不能再执行新的更新，需要先擦掉一些记录，把 checkpoint 推进一下。有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 crash-safe。

重做日志缓存、重做日志文件都是以**块（block）**的方式进行保存的，称之为**重做日志块（redo log block）**，块的大小是固定的512字节。redo log它是固定大小的，可以看作是一个逻辑上的 **log group**，由一定数量的**log block** 组成。它的写入方式是从头到尾开始写，写到末尾又回到开头循环写。



> 因为redo log是循环写文件的，所以不能用于数据库删除恢复，此时可能会丢失已被擦除的数据。恢复被删除的数据库只能通过binlog日志恢复。



写入 redo log 的方式使用了追加操作， 所以磁盘操作是**顺序写**，而写入数据需要先找到写入位置，然后才写到磁盘，所以磁盘操作是**随机写**。磁盘的顺序写 比随机写高效的多，因此 redo log 写入磁盘的开销更小。

可以说这是 WAL 技术的另外一个优点：**MySQL 的写操作从磁盘的随机写变成了顺序写**，提升语句的执行性能。这是因为 MySQL 的写操作并不是立刻更新到磁盘上，而是先记录在日志上，然后在合适的时间再更新到磁盘上 。

redolog日志文件设置为循环写可以避免新建、删除文件带来的抖动。



## crash-safe

因为MySQL对数据的修改不会马上写入磁盘，而是存储在内存中，如果此时MySQL宕机了，就会丢失内存中的这部分数据，此时就需要读取redolog日志中的数据进行恢复。

在MySQL宕机重启后，会读取redolog中从checkpoint开始到writepos结束区间的日志数据进行恢复，而不是读取所有的redolog日志文件。



## 相关参数

### innodb_flush_log_at_trx_commit

```sql
mysql> show variables like "innodb_flush_log_at_trx_commit";
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.00 sec)
```



InnoDB 在执行更新语句的过程中，生成的 redo log 会先写入到 redo log buffer 中，然后默认等每次事务提交时，将缓存在 redo log buffer 中的 redo log 按组的方式顺序写到磁盘。除此之外，InnoDB 还提供了另外两种策略，由参数 `innodb_flush_log_at_trx_commit` 参数控制，参数可选值分别代表的策略如下：

- 为 0 时表示每次事务提交时 ，将 redo log 留在 redo log buffer 中，该模式下在事务提交时不会主动触发写入磁盘的操作，而是通过MySQL自身的机制异步执行持久化到磁盘操作。
- 为 1 时（默认）表示每次事务提交时，都将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘，这样可以保证 MySQL 异常重启之后数据不会丢失。
- 为 2 时表示每次事务提交时，都只是缓存在 redo log buffer 里的 redo log 写到 redo log 文件，注意写入到redo log 文件并不意味着写入到了磁盘，因为操作系统的文件系统中有个 Page Cache，Page Cache 是专门用来缓存文件数据的，所以写入redo log文件只是写入到了操作系统的文件缓存。



![img](https://oss.xubighead.top/oss/image/202506/1929824752563752962.png)



InnoDB 的后台线程每隔 1 秒：

- 针对参数 0 ：会把缓存在 redo log buffer 中的 redo log ，通过调用 `write()` 写到操作系统的 Page Cache，然后调用 `fsync()` 持久化到磁盘。所以参数为 0 的策略，MySQL 进程的崩溃会导致上一秒钟所有事务数据的丢失;
- 针对参数 2 ：调用 fsync，将缓存在操作系统中 Page Cache 里的 redo log 持久化到磁盘。所以参数为 2 的策略，较取值为 0 情况下更安全，因为 MySQL 进程的崩溃并不会丢失数据，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。



加入了后台现线程后，innodb_flush_log_at_trx_commit 的刷盘时机如下图：

![img](https://oss.xubighead.top/oss/image/202506/1929824438280359937.png)



这三个参数的数据安全性和写入性能的比较如下：

- 数据安全性：参数 1 > 参数 2 > 参数 0
- 写入性能：参数 0 > 参数 2> 参数 1

所以，数据安全性和写入性能是熊掌不可得兼的，**要不追求数据安全性，牺牲性能；要不追求性能，牺牲数据安全性**。

- 在一些对数据安全性要求比较高的场景中，显然 `innodb_flush_log_at_trx_commit` 参数需要设置为 1。
- 在一些可以容忍数据库崩溃时丢失 1s 数据的场景中，将该值设置为 0，这样可以明显地减少日志同步到磁盘的 I/O 操作。
- 安全性和性能折中的方案就是参数 2，虽然参数 2 没有参数 0 的性能高，但是数据安全性方面比参数 0 强，因为参数 2 只要操作系统不宕机，即使数据库崩溃了，也不会丢失数据，同时性能方便比参数 1 高。



### innodb_flush_log_at_timeout

```sql
mysql> show variables like "innodb_flush_log_at_timeout";
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| innodb_flush_log_at_timeout | 1     |
+-----------------------------+-------+
1 row in set (0.02 sec)
```



innodb_flush_log_at_timeout 变量控制日志刷新频率(1s由log buffer刷盘一次）。



### innodb_log_buffer_size

```sql
mysql> show variables like "innodb_log_buffer_size";
+------------------------+----------+
| Variable_name          | Value    |
+------------------------+----------+
| innodb_log_buffer_size | 16777216 |
+------------------------+----------+
1 row in set (0.01 sec)
```



`innodb_log_buffer_size`参数控制redo log buffer缓存空间的大小，默认值是16M。



### innodb_log_file_size

```sql
mysql> show variables like '%innodb_log_file_size%';
+----------------------+----------+
| Variable_name        | Value    |
+----------------------+----------+
| innodb_log_file_size | 50331648 |
+----------------------+----------+
1 row in set (0.00 sec)
```



`innodb_log_file_size`参数用于设置redo log单文件大小，默认大小是48M。最大值为512G，注意最大值 指的是整个 redo log 系列文件之和，
即(`innodb_log_files_in_group` * `innodb_log_file_size` )不能大 于最大值512G。



### innodb_log_files_in_group

```sql
mysql> show variables like 'innodb_log_files_in_group';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| innodb_log_files_in_group | 2     |
+---------------------------+-------+
1 row in set (0.00 sec)
```



`innodb_log_files_in_group`参数用于设置redo log文件数量，默认2个，最大100个。



### innodb_log_group_home_dir

```sql
mysql> show variables like 'innodb_log_group_home_dir';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| innodb_log_group_home_dir | ./    |
+---------------------------+-------+
1 row in set (0.01 sec)
```



指定redo log文件组所在的路径，默认值为 ./ ，表示在数据库 的数据目录下。MySQL的默认数据目录(var/lib/mysql)下默认有两个名为ib_logfile0和ib_logfile1的文件，redo log buffer中的日志默认情况下就是刷新到这两个磁盘文件中。



## 总结

### redo log 实现功能

- **实现事务的持久性，让 MySQL 有 crash-safe 的能力**，能够保证 MySQL 在任何时间段突然崩溃，重启后之前已提交的记录都不会丢失；
- **将写操作从随机写变成了顺序写，单次事务提交变成组提交**，提升 MySQL 写入磁盘的性能。



### redo log 和 binlog 对比

| <span style="display:inline-block;width:70px"> </span> | binlog                                                       | redo log                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 适用对象                                               | binlog 是 MySQL 的 Server 层实现的日志，所有存储引擎都可以使用 | redo log 是 Innodb 存储引擎实现的日志                        |
| 日志内容                                               | bin log是逻辑日志，记录的是SQL语句的原始逻辑（逻辑日志可以理解为不管哪一台服务器，都能通过同样的逻辑恢复数据） | redo log是物理日志，记录的是在某个数据页上做了什么修改（物理日志可以理解为只能对当前服务器有效，其它服务器无法通过相同日志恢复数据） |
| 写入方式                                               | binlog 是追加写，写满一个文件，就创建一个新的文件继续写，不会覆盖以前的日志，保存的是全量的日志 | redo log 是循环写，日志空间大小是固定，全部写满就从头开始，保存未被刷入磁盘的脏页日志 |
| 用途                                                   | binlog 用于备份恢复、主从复制                                | redo log 用于掉电等故障恢复                                  |



binlog和redo log虽然都是记录了同一份数据变化，但是两者都是必要的。

**binlog必要性：**

- redolog只有InnoDB有，别的引擎没有；
- redolog是循环写的，不持久保存，binlog的归档这个功能，redolog是不具备的。



**redolog必要性：**

- 内存中脏页会在MySQL宕机时丢失，此时需要redolog进行恢复数据。



## 扩展

### 被修改 Undo 页面，需要记录对应 redo log 吗？

需要的。

开启事务后，InnoDB 层更新记录前，首先要记录相应的 undo log，如果是更新操作，需要把被更新的列的旧值记下来，也就是要生成一条 undo log，undo log 会写入 Buffer Pool 中的 Undo 页面。

不过，在修改该 Undo 页面前需要先记录对应的 redo log，所以**先记录修改 Undo 页面的 redo log ，然后再真正的修改 Undo 页面**。



### redo log 和 undo log 区别在哪？

这两种日志是属于 InnoDB 存储引擎的日志，它们的区别在于：

- redo log 记录了此次事务**「完成后」**的数据状态，记录的是更新之**「后」**的值；
- undo log 记录了此次事务**「开始前」**的数据状态，记录的是更新之**「前」**的值；

事务提交之前发生了崩溃，重启后会通过 undo log 回滚事务，事务提交之后发生了崩溃，重启后会通过 redo log 恢复事务，如下图：

![事务恢复](https://oss.xubighead.top/oss/image/202506/1929824517540122625.png)





所以有了 redo log，再通过 WAL 技术，InnoDB 就可以保证即使数据库发生异常重启，之前已提交的记录都不会丢失，这个能力称为 **crash-safe**（崩溃恢复）。可以看出来， **redo log 保证了事务四大特性中的持久性**。








