---
layout: post
title: 2024-01-02-第008章-MySQL InnoDB存储引擎
categories: [MySQL]
description: 
keywords: MySQL InnoDB存储引擎.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# MySQL Innodb存储引擎

## 相关参数

### innodb_page_size

```sql
mysql> show variables like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.01 sec)
```



innodb_page_size用于配置每一页的大小。