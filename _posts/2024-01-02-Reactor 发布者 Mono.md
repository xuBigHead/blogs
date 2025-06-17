---
layout: post
title: Reactor 发布者 Mono.md
categories: [Reactor 响应式编程]
description: 
keywords: Reactor 发布者 Mono.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
## Mono

### Static Method

#### create

如果这个 **异步调用** 不会返回 `CompletableFuture`，是有自己的 **回调方法**，那怎么创建 `Mono` 呢？可以使用 `static <T> Mono<T> create(Consumer<MonoSink<T>> callback)` 方法：



#### delay

创建一个 Mono 序列，在指定的延迟时间之后，产生数字 0 作为唯一值。



#### fromCallable

分别从 Callable、CompletionStage、CompletableFuture、Runnable 和 Supplier 中创建 Mono。

```java
@Test
public void create() {
    Mono.create(sink -> sink.success("create")).subscribe(this::info);

    Mono.create(sink -> {
        ListenableFuture<ResponseEntity<String>> entity =
            new ListenableFutureTask<>(() -> new ResponseEntity<>("body", HttpStatus.OK));
        entity.addCallback(new ListenableFutureCallback<ResponseEntity<String>>() {
            @Override
            public void onSuccess(ResponseEntity<String> result) {
                sink.success(result.getBody());
            }

            @Override
            public void onFailure(Throwable ex) {
                sink.error(ex);
            }
        });
    }).subscribe(this::info);
}
```



#### fromCompletionStage

#### fromFuture

如果 **异步方法** 返回一个 `CompletableFuture`，那可以基于这个 `CompletableFuture` 创建一个 `Mono`：



```java
@Test
public void fromFuture(){
    ListenableFuture<ResponseEntity<String>> entity =
        new ListenableFutureTask<>(() -> new ResponseEntity<>("body", HttpStatus.OK));
    CompletableFuture<ResponseEntity<String>> completableFuture = entity.completable();
    Mono.fromFuture(completableFuture).subscribe(this::info);
}
```



#### fromRunnable

#### fromSupplier



#### ignoreElements

创建一个 Mono 序列，忽略作为源的 Publisher 中的所有元素，只产生结束消息。



#### just

#### justOrEmpty

从一个 Optional 对象或可能为 null 的对象中创建 Mono。只有 Optional 对象中包含值或对象不为 null 时，Mono 序列才产生对应的元素。



### Non-Static Method

#### ignoreElement