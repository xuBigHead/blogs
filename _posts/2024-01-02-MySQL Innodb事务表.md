---
layout: post
title: 第0010章-MySQL Innodb事务表
categories: [MySQL]
description: 
keywords: MySQL Innodb事务表.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# 查看事务表

可以在 information_schema 库的 innodb_trx 这个表中查询事务信息。



## 事务表字段说明

```sql
mysql> select * from information_schema.innodb_trx\G;
*************************** 1. row ***************************
                    trx_id: 562948418071768
                 trx_state: RUNNING
               trx_started: 2023-11-01 07:31:36
     trx_requested_lock_id: NULL
          trx_wait_started: NULL
                trx_weight: 0
       trx_mysql_thread_id: 5799
                 trx_query: select * from information_schema.innodb_trx
       trx_operation_state: NULL
         trx_tables_in_use: 0
         trx_tables_locked: 0
          trx_lock_structs: 0
     trx_lock_memory_bytes: 1128
           trx_rows_locked: 0
         trx_rows_modified: 0
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
 trx_adaptive_hash_latched: 0
 trx_adaptive_hash_timeout: 0
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
       trx_schedule_weight: NULL
1 row in set (0.00 sec)
```



## 使用示例

查找持续时间超过 60s 的事务

```sql
mysql> select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60\G;
*************************** 1. row ***************************
                    trx_id: 562948418071768
                 trx_state: RUNNING
               trx_started: 2023-11-01 07:31:36
     trx_requested_lock_id: NULL
          trx_wait_started: NULL
                trx_weight: 0
       trx_mysql_thread_id: 5799
                 trx_query: select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
       trx_operation_state: NULL
         trx_tables_in_use: 0
         trx_tables_locked: 0
          trx_lock_structs: 0
     trx_lock_memory_bytes: 1128
           trx_rows_locked: 0
         trx_rows_modified: 0
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
 trx_adaptive_hash_latched: 0
 trx_adaptive_hash_timeout: 0
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
       trx_schedule_weight: NULL
1 row in set (0.01 sec)
```
