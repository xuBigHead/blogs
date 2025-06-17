---
layout: post
title: 2024-01-02-linux-tool-awk.md
categories: [Linux]
description: 
keywords: linux-tool-awk.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Awk_Programming

awk指令是一个强大的文本分析工具，网上资料很多，这里就不展开说了；本例子简单解释一下就是从access.log日志中取第一个参数（$1）,因为日志中第一个参数就是我们需要的ip信息



```bash
# 统计nginx配置文件中访问次数最多的10个ip
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr -k1 | head -n 10
```

```
[root@izuf6e56zt7ebjhby2hxboz dir]# awk '{print $1}' /opt/nginx/log/access.log | sort | uniq -c | sort -nr -k1 | head -n 10
   2071 38.54.101.133
   1752 8.219.71.118
   1590 8.219.119.144
   1266 83.97.20.34
   1115 60.191.17.221
    910 114.84.240.243
    877 221.225.31.88
    877 180.171.144.209
    812 45.155.205.127
    721 118.25.62.49
```