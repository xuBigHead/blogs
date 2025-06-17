---
layout: post
title: ElementPlus
categories: [ElementPlus]
description: 
keywords: ElementPlus.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# 版本升级

## 找不到./lib/locale/lang/zh-cn

> 2.3.3 -> 2.4.4

```shell
No known conditions for "./lib/locale/lang/zh-cn" entry in "element-plus" package
```



这个问题的原因是因为 Element Plus 版本的升级。在新版本中，"./lib/locale/lang/zh-cn" 的路径发生了变化，导致了编译错误。


### 解决方案

修改引用路径：

```tsx
import zhCn from "element-plus/lib/locale/lang/zh-cn";
// 改为
import zhCn from "element-plus/es/locale/lang/zh-cn";
```
