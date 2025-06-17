---
layout: post
title: 2025-05-13-第030章-Go 标准库 runtime
categories: [Go]
description: 
keywords: Go 标准库 runtime.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Go 标准库 runtime

## 概述

## 函数

### Goexit

可以通过调用`runtime.Goexit`函数退出一个goroutine，`runtime.Goexit`函数没有参数。

```go
func TestGoexit(t *testing.T) {
	c := make(chan int)
	go func() {
		defer func() { c <- 1 }()
		defer fmt.Println("Go")
		func() {
			defer fmt.Println("C")
			runtime.Goexit()
		}()
		fmt.Println("Java")
	}()
	<-c
}
```



### SetFinalizer

可以调用`runtime.SetFinalizer`函数来给一个对象设置一个终结器函数，一般说来，此终结器函数将在此对象被垃圾回收之前调用。但是终结器并非被设计为对象的析构函数，通过`runtime.SetFinalizer`函数设置的终结器函数并不保证总会被运行，因此不应该依赖于终结器来保证程序的正确性。

终结器的主要用途是为了库包的维护者能够尽可能地避免因为库包使用者不正确地使用库包而带来的危害。例如，当在程序中使用完某个文件后，应该将其关闭。但是有时候因为种种原因，比如经验不足或者粗心大意，导致一些文件在使用完成后并未被关闭，那么和这些文件相关的很多资源只有在此程序退出之后才能得到释放，这属于资源泄漏。为了尽可能地避免防止资源泄露，`os`库包的维护者将会在一个`os.File`对象被被创建的时候为之设置一个终结器，此终结器函数将关闭此`os.File`对象，当此`os.File`对象因为不再被使用而被垃圾回收的时候，此终结器函数将被调用。

有一些终结器函数永远不会被调用，并且有时候不当的设置终结器函数将会阻止对象被垃圾回收。


