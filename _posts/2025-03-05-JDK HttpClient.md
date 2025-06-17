---
layout: post
title: 第021章-JDK HttpClient
categories: [Java]
description: 
keywords: JDK HttpClient.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# JDK类 HttpClient

## 概述

Java 11对Java 9中引入并在Java 10中进行了更新的Http Client API进行了标准化，在前两个版本中进行孵化的同时，Http Client几乎被完全重写，并且现在完全支持异步非阻塞。并且，Java 11中，Http Client的包名由`jdk.incubator.http`改为`java.net.http`，该API通过`CompleteableFuture`提供非阻塞请求和响应语义。



## 源码解析

### 方法

#### send

```

```



## 实践应用

### 发送Get请求

```java
@Test
public void testSend() {
    var request = HttpRequest.newBuilder()
        .uri(URI.create("https://www.baidu.com".indent()))
        .GET()
        .build();
    try(var client = HttpClient.newHttpClient()) {
        // 同步
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());

        // 异步
        client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
            .thenApply(HttpResponse::body)
            .thenAccept(System.out::println);
    } catch (IOException | InterruptedException e) {
        throw new RuntimeException(e);
    }
}
```
