---
layout: post
title: Go 标准库 strings.md
categories: [Go]
description: Go
keywords: Go
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Go 标准库 strings

## 概述

## 类型

### Builder

`strings.Builder`类型值不推荐被复制，`strings.Builder`的实现会在运行时刻探测到非法的`strings.Builder`值复制，一旦这样的复制被发现，就会产生恐慌。例如：

```go
func TestForbidDuplicateBuilderValue(t *testing.T) {
    var b strings.Builder
    b.WriteString("hello ")
    var b2 = b
    b2.WriteString("world!") // 一个恐慌将在这里产生
}
```



其他如`bytes.Buffer`类型和`sync`标准库包里的类型的值也不推荐被复制，复制标准库包`sync`中类型的值会被Go官方工具链提供的`go vet`命令检测到并被警告，复制`bytes.Buffer`的值不会在运行时被检查到，也不会被`go vet`命令所检测到，千万要小心不要随意这样做。



## 函数

### Trim

标准包`strings`和`bytes`里有多个修剪（trim）函数，这些函数可以被分类为两组：

1. `Trim`、`TrimLeft`、`TrimRight`、`TrimSpace`、`TrimFunc`、`TrimLeftFunc`和`TrimRightFunc`：修剪首尾所有满足指定（或隐含）条件的utf-8编码的Unicode码点(即rune)。`TrimSpace`隐含了修剪各种空格符。这些函数将检查每个开头或结尾的rune值，直到遇到一个不满足条件的rune值为止。
2. `TrimPrefix`和`TrimSuffix`：会把指定前缀或后缀的子字符串（或子切片）作为一个整体进行修剪。

```go
func TestTrim(t *testing.T) {
	var s = "abaay森z众xbbab"
	fmt.Println(strings.TrimPrefix(s, "ab")) // aay森z众xbbab
	fmt.Println(strings.TrimSuffix(s, "ab")) // abaay森z众xbb
	fmt.Println(strings.TrimLeft(s, "ab"))   // y森z众xbbab
	fmt.Println(strings.TrimRight(s, "ab"))  // abaay森z众x
	fmt.Println(strings.Trim(s, "ab"))       // y森z众x
	fmt.Println(strings.TrimFunc(s, func(r rune) bool {
		return r < 128 // trim all ascii chars
	})) // 森z众
}
```
