---
layout: post
title: JDK Random.md
categories: [Java]
description: Java
keywords: Java
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# JDK类 Random



# JDK类 RandomGenerator

## 概述

Java 17之前可以借助`Random`、`ThreadLocalRandom`和`SplittableRandom`来生成随机数，不过，这3个类都各有缺陷，且缺少常见的伪随机算法支持。Java 17为伪随机数生成器（pseudorandom number generator，PRNG，又称为确定性随机位生成器）增加了新的接口类型和实现，使得开发者更容易在应用程序中互换使用各种 PRNG 算法。

> PRNG用来生成接近于绝对随机数序列的数字序列，一般来说，PRNG会依赖于一个初始值，也称为种子，来生成对应的伪随机数序列。只要种子确定了，PRNG所生成的随机数就是完全确定的，因此其生成的随机数序列并不是真正随机的。



## 源码解析

### 方法定义



## 实践应用

```java
RandomGeneratorFactory<RandomGenerator> l128X256MixRandom = RandomGeneratorFactory.of("L128X256MixRandom");
// 使用时间戳作为随机数种子
RandomGenerator randomGenerator = l128X256MixRandom.create(System.currentTimeMillis());
// 生成随机数
randomGenerator.nextInt(10);
```
