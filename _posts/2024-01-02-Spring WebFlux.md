---
layout: post
title: Spring WebFlux.md
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Introduction

`Spring 5` 引入的一个基于 `Netty` 而不是 `Servlet` 的高性能的 `Web` 框架 - `Spring WebFlux` ，但是使用方式并没有同传统的基于 `Servlet` 的 `Spring MVC` 有什么大的不同。



```java
@GetMapping("/users/{id}")
public Mono<ReactiveUser> getUser(@PathVariable("id") UUID id) {
    return userService.find(id);
}
```

最大的变化就是返回值从 `ReactiveUser` 所表示的一个对象变为 `Mono<ReactiveUser>` 或 `Flux<ReactiveUser>`。





# References

- [Guide to Spring 5 WebFlux](https://www.baeldung.com/spring-webflux)