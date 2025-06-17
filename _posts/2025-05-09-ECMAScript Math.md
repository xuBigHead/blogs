---
layout: post
title: 第009章-ECMAScript Math
categories: [ECMAScript]
description: 
keywords: ECMAScript Math.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ECMAScript Math

## 源码解析

### 方法定义

> @since ES6

```ts
interface Math {
    // 用于返回数字的32位无符号整数形式的前导0的个数
    clz32(x: number): number;
    // 两个数以32位带符号整数形式相乘的结果，返回的也是一个32位的带符号整数
    imul(x: number, y: number): number;
    // 判断数字的符号（正、负、0）
    sign(x: number): number;
    // 用于计算以10为底的x的对数
    log10(x: number): number;
    // 用于计算2为底的x的对数
    log2(x: number): number;
    // 用于计算1 + x的自然对数，即Math.log(1 + x) 
    log1p(x: number): number;
    // 用于计算e的x次方减1的结果，即Math.exp(x) - 1
    expm1(x: number): number;
    // 用于计算双曲余弦
    cosh(x: number): number;
    // 用于计算双曲正弦
    sinh(x: number): number;
    // 用于计算双曲正切
    tanh(x: number): number;
    // 用于计算反双曲余弦
    acosh(x: number): number;
    // 用于计算反双曲正弦
    asinh(x: number): number;
    // 用于计算反双曲正切
    atanh(x: number): number;
    // 用于计算所有参数的平方和的平方根
    hypot(...values: number[]): number;
    // 用于返回数字的整数部分
    trunc(x: number): number;
    // 用于获取数字的32位单精度浮点数形式
    fround(x: number): number;
    // 用于计算一个数的立方根
    cbrt(x: number): number;
}
```



## 实践应用

### 实例方法

#### trunc方法

trunc方法用于去除一个数的小数部分，返回整数部分。

```ts
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
```



## 参考资料

- [数值的扩展 - ECMAScript 6入门](https://es6.ruanyifeng.com/#docs/number#Math-%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%89%A9%E5%B1%95)