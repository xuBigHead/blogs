---
layout: post
title: 2025-06-05-第000章-Reactor 发布者 Flux
categories: [Reactor 响应式编程]
description: 
keywords: Reactor 发布者 Flux.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
## Flux

### Static Method

#### combineLatest

构建一个Flux，混合由多个的发布者发布最新事件。

combineLatest 操作符**把所有流中的最新产生的元素合并成一个新的元素，作为返回结果流中的元素**。只要其中任何一个流中产生了新的元素，合并操作就会被执行一次，结果流中就会产生新的元素。



![img](https://oss.xubighead.top/oss/image/202506/1930509456954724354.jpg)

```java
@Test
public void combineLatest() {
    Flux.combineLatest(Arrays::toString, Flux.just(1, 4, 5, 7), Flux.just(2, 3, 6))
        .subscribe(this::info);

    this.info("==============");
    // 流中最新产生的元素会被收集到一个数组中，通过 Arrays.toString 方法来把数组转换成 String
    Flux.combineLatest(
        Arrays::toString,
        Flux.interval(Duration.ofMillis(100)).take(5),
        Flux.interval(Duration.ofMillis(50), Duration.ofMillis(100)).take(5)
    ).toStream().forEach(this::info);
}
```



#### concat

连接两个Flux，连接由源下游发射的迭代转发元素提供的所有源。通过顺序订阅第一个源，然后在订阅下一个源之前等待它完成，等等，直到最后一个源完成，从而实现连接。任何错误立即中断序列并被转发到下游。

与combineLatest不同的是，concat都是在前一个流完成后在连接新的流。而combineLatest，则哪个事件最先到的，哪个先处理。



<img src="https://oss.xubighead.top/oss/image/202506/1930509480073728002.jpg" alt="img" style="zoom:67%;" />



#### concatDelayError

将从父Publisher发出的ONNEXT信号连接到所有源，转发由下游源发出的元素。通过顺序订阅第一个源，然后在订阅下一个源之前等待它完成，等等，直到最后一个源完成，从而实现连接。错误不会中断主序列，但是在其余的源有机会被连接之后被传播。

此操作符在取消时丢弃内部排队的元素以产生背压。



<img src="https://oss.xubighead.top/oss/image/202506/1930509531395231746.jpg" alt="img" style="zoom:67%;" />



#### create

以编程方式创建具有多次发射能力的Flux，元素通过FluxSink API以同步或异步方式进行。

create()方法与 generate()方法的不同之处在于**所使用的是 FluxSink 对象**。FluxSink **支持同步和异步的消息产生**，并且可以在一次调用中产生多个元素。



```java
@Test
public void createTest() {
    Flux.create(t -> t.next(1).next(2).next(3).complete())
        .subscribe(this::info);
}
```



#### defer

每当对得到的Flux进行Subscription时，延迟提供Publisher，因此实际的源实例化被推迟，直到每个订阅和Supplier可以创建订阅者特定的实例。但是，如果供应商没有生成新的实例，这个操作符将有效地从Publisher起作用。

<img src="https://oss.xubighead.top/oss/image/202506/1930509551322370049.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void deferTest() throws InterruptedException {
    Flux<String> flux1 = Flux.just(new Date().toString());

    Flux<String> flux2 = Flux.defer(() -> Flux.just(new Date().toString()));

    flux1.subscribe(x -> this.info("s1: " + x));
    flux2.subscribe(x -> this.info("s2: " + x));

    TimeUnit.SECONDS.sleep(5);
    flux1.subscribe(x -> this.info("s3: " + x));
    flux2.subscribe(x -> this.info("s4: " + x));
}
```



#### empty

创建一个Flux，完成而不发射任何项目。

<img src="https://oss.xubighead.top/oss/image/202506/1930509569781501954.jpg" alt="img" style="zoom: 67%;" />



```java
@Test
public void emptyTest() {
    Flux.empty()
        .subscribe(this::info, this::info, () -> this.info("empty"));
}
```



#### error

创建一个Flux，它在订阅之后立即以指定的错误终止。

<img src="https://oss.xubighead.top/oss/image/202506/1930509586982342657.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void errorTest() {
    Flux.error(new RuntimeException("emitter an error"))
        .subscribe(this::info, this::info, () -> this.info("never"));
}
```



#### first

选择第一个Publisher发出任何信号（onNext/onError/onComplete）并重放来自该Publisher的所有信号，有效地表现得像这些竞争源中最快的一个。

返回一个新的Flux，其性能最快。

<img src="https://oss.xubighead.top/oss/image/202506/1930509604250292225.jpg" alt="img" style="zoom:67%;" />



#### from

用Flux API装饰指定的Publisher，通过Publisher创建一个Flux。

<img src="https://oss.xubighead.top/oss/image/202506/1930509618838081537.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void fromTest() {
    Flux.from(Flux.just(1, 2, 3)).subscribe(this::info);

    Flux.from(Mono.just(4)).subscribe(this::info);

    Flux.fromArray(new Integer[]{5, 6, 7}).subscribe(this::info);

    Flux.fromIterable(Lists.newArrayList(8, 9, 10)).subscribe(this::info);

    Flux.fromStream(Stream.of(11, 12, 13)).subscribe(this::info);
}
```



##### fromArray

创建一个Flux，它发出包含在提供的数组中的项。

<img src="https://oss.xubighead.top/oss/image/202506/1930509634809991170.jpg" alt="img" style="zoom:67%;" />



##### fromIterable

创建一个个Flux，它发出所提供的Iterable中包含的项。将为每个subscriber创建一个新的Iterable。

<img src="https://oss.xubighead.top/oss/image/202506/1930509648491810817.jpg" alt="img" style="zoom:67%;" />



##### fromStream

创建一个Flux，它发出所提供的Stream中包含的项。请记住，Stream不能被重新使用，这可能是有问题的。多订阅或重订阅的情况（如repeat或retry)Stream是closed由操作员取消，错误或完成。

<img src="https://oss.xubighead.top/oss/image/202506/1930509669207478274.jpg" alt="img" style="zoom:67%;" />



#### generate

以编程方式创建一个的Flux，通过consumer回调逐一生成信号；generate中next只能调1次，否则会报错 reactor.core.Exceptions$ErrorCallbackNotImplemented。

<img src="https://oss.xubighead.top/oss/image/202506/1930509685359742978.jpg" alt="img" style="zoom:67%;" />



generate() 方法通过同步和逐一地方式来产生 Flux 序列。序列的产生是通过调用所提供的 SynchronousSink 对象的 next()，complete()和 error(Throwable)方法来完成的。逐一生成的含义是在具体的生成逻辑中，next() 方法只能最多被调用一次。在有些情况下，序列的生成可能是有状态的，需要用到某些状态对象。此时可以使用 generate() 方法的另外一种形式 `generate(Callable<S> stateSupplier, BiFunction<S,SynchronousSink<T>,S> generator)`，其中 stateSupplier 用来提供初始的状态对象。在进行序列生成时，状态对象会作为 generator 使用的第一个参数传入，可以在对应的逻辑中对该状态对象进行修改以供下一次生成时使用。


```java
@Test
public void generateTest() {
    Flux.generate(
        () -> 0, // 初始state值
        (state, sink) -> {
            sink.next("3 x " + state + " = " + 3 * state); // 产生数据是同步的，每次产生一个数据
            if (state == 10) {
                sink.complete();
            }
            return state + 1; // 改变状态
        },
        state -> this.info("state: " + state))
        .subscribe(this::info);

    Flux.generate(
        sink -> {
            // next 只能调用一次，否则抛出 More than one call to onNext.
            sink.next("1");
            // 如果不调用 complete()方法，所产生的是一个无限序列。
            sink.complete();
        })
        .subscribe(this::info);
}
```



#### interval

创建一个Flux，它以0开始发射长值并递增。全局计时器上指定的时间间隔。如果需求没有及时产生，一个OnError将用来发出信号。IllegalStateException详细说明无法发出的信息。在正常情况下，Flux将永远不会完成。

<img src="https://oss.xubighead.top/oss/image/202506/1930509703470747649.jpg" alt="img" style="zoom: 67%;" />

```java
@Test
public void intervalTest() throws InterruptedException {
    Flux.interval(Duration.ofSeconds(3))
        .map(input -> {
            if (input < 2) return "tick " + input;
            throw new RuntimeException("boom");
        })
        .onErrorReturn("Uh oh")
        .subscribe(this::info);
    // 防止程序过早停止
    TimeUnit.SECONDS.sleep(10);
}
```





#### just

创建一个Flux，它发出所提供的元素，然后完成。

<img src="https://oss.xubighead.top/oss/image/202506/1930509720134717442.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void justTest() {
    Flux.just(1, 2, 3).subscribe(this::info);
}
```



#### merge

将由Publisher发出的Publisher序列的数据合并为交织合并序列。与concat不同的是，内部source踊跃竞争。和combineLatest类似，但它要求是同类型的流合并，combineLatest需要提供合并方式。

<img src="https://oss.xubighead.top/oss/image/202506/1930509734802198529.jpg" alt="img" style="zoom:67%;" />

merge 和 mergeSequential 操作符**用来把多个流合并成一个 Flux 序列**。不同之处在于 **merge 按照所有流中元素的实际产生顺序来合并，而 mergeSequential 则按照所有流被订阅的顺序，以流为单位进行合并**

```java
@Test
public void merge() {
    Flux.merge(Flux.interval(Duration.ofMillis(0), Duration.ofMillis(100)).take(5),
               Flux.interval(Duration.ofMillis(50), Duration.ofMillis(100)).take(5))
        .toStream()
        .forEach(this::info);
}
```



#### mergeOrdered

通过从每个源（由它们的自然顺序定义）中选择最小值，将来自提供的Publisher序列的数据合并成有序的合并序列。这不是一个SORT，因为它不考虑整个序列。相反，该操作符只考虑来自每个源的一个值，并选择所有这些值中最小的值，然后为选择的源补充槽。

返回 一个合并Flux，但保持原始排序的合并Flux。



<img src="https://oss.xubighead.top/oss/image/202506/1930509751310979074.jpg" alt="img" style="zoom:67%;" />

#### mergeSequential

```java
@Test
public void mergeSequential(){
    Flux.mergeSequential(Flux.interval(Duration.ofMillis(0), Duration.ofMillis(100)).take(5),
        Flux.interval(Duration.ofMillis(50), Duration.ofMillis(100)).take(5))
        .toStream()
        .forEach(this::info);
}
```

代码中分别使用了 merge 和 mergeSequential 操作符。进行合并的流都是每隔 100 毫秒产生一个元素，不过第二个流中的每个元素的产生都比第一个流要延迟 50 毫秒。在使用 merge 的结果流中，来自两个流的元素是按照时间顺序交织在一起；而使用 mergeSequential 的结果流则是首先产生第一个流中的全部元素，再产生第二个流中的全部元素。




#### never

创建一个Flux，它永远不会发出任何数据、错误或完成信号。

<img src="https://oss.xubighead.top/oss/image/202506/1930509771909206018.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void neverTest() {
    Flux.never()
        .subscribe(this::info, this::info, () -> this.info("never"));
}
```



#### range

建立一个Flux，它只会发出一个count递增整数的序列，从start开始。也就是说，在start（包含）和start + count（排除）之间发出整数，然后完成。

<img src="https://oss.xubighead.top/oss/image/202506/1930509787717537794.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void rangeTest() {
    // 在start和start + count之间发出整数，然后完成
    Flux.range(3, 4).subscribe(this::info);
}
```



#### switchOnNext

创建一个Flux，该镜像反映最近发布的Publisher，转发其数据直到源代码中的新Publisher进入。 一旦源中没有新的Publisher（源已完成），并且最后一个镜像Publisher也已完成，则生成的Flux将完成。

从最新的发布者那里获取事件，如果有新的发布者加入，则改用新的发布者。当最后一个发布者完成所有发布事件，并且没有发布者加入，则flux完成。

<img src="https://oss.xubighead.top/oss/image/202506/1930509802611511298.jpg" alt="img" style="zoom:67%;" />



#### using

<img src="https://oss.xubighead.top/oss/image/202506/1930509817392238593.jpg" alt="img" style="zoom:67%;" />



##### usingWhen



#### zip

将两个源压缩在一起，也就是说，等待所有源发出一个元素，并将这些元素组合成一个输出值（由提供的组合器构造）。操作将继续这样做，直到任何来源完成。错误将立即被转发。这种“Step-Merge”处理在分散聚集场景中特别有用。

<img src="https://oss.xubighead.top/oss/image/202506/1930509831766118402.jpg" alt="img" style="zoom:67%;" />



**并发执行** 是常见的一个需求。`Reactive Programming` 虽然是一种 **异步编程** 方式，但是 **异步** 不代表就是 **并发并行** 的。在使用 `Reactor` 开发有 **并发** 执行场景的 **反应式代码** 时，应该使用 `Mono` 和 `Flux` 中的 `zip()` 方法：

```java
@Test
public void zip(){
    Flux<Integer> flux1 = Flux.just(1);
    Flux<Integer> flux2 = Flux.just(2);
    Flux.zip(flux1, flux2).map(a -> a.getT1() + a.getT2()).subscribe(this::info);
}
```



flux1和flux2是并行的。

对于两个元素的并发执行，也可以通过 `zip(Mono<? extends T1> p1, Mono<? extends T2> p2, BiFunction<? super T1, ? super T2, ? extends O> combinator)` 方法直接将结果合并。方法是传递 `BiFunction` 实现 **合并算法**。



### Non-Static Method

#### all

如果这个序列的所有值与谓词匹配，则发出一个布尔布尔值true 。该实现使用短路逻辑，如果谓词与值不匹配，则用FALSE完成。

<img src="https://oss.xubighead.top/oss/image/202506/1930509850753732609.jpg" alt="img" style="zoom:67%;" />



#### any

如果这个Flux序列的任何值与谓词匹配，则发出一个布尔布尔值true。该实现使用短路逻辑，如果任何值与谓词不匹配，则完成FALSE。

<img src="https://oss.xubighead.top/oss/image/202506/1930509877458866178.jpg" alt="img" style="zoom:67%;" />

#### as

将此Flux转换为目标类型。



#### blockFirst

阻塞至第一个值处理完成。



#### blockLast

阻塞至最后一个值处理完成。



#### buffer

将所有传入的值收集到一个列表缓冲器中，一旦该Flux完成，将由返回的Flux发出。该操作在数据信号触发的取消或错误时丢弃缓冲器。

<img src="https://oss.xubighead.top/oss/image/202506/1930509893481107457.jpg" alt="img" style="zoom:67%;" />

```java
@Test
public void bufferTest(){
    Flux.range(1, 100).buffer(30).subscribe(this::info);
}
```



#### bufferTimeout

这两个操作符的作用是把当前流中的元素收集到集合中，并把集合对象作为流中的新元素。在进行收集时可以指定不同的条件：所包含的元素的最大数量或收集的时间间隔。方法 buffer()仅使用一个条件，而 bufferTimeout()可以同时指定两个条件。指定时间间隔时可以使用 Duration 对象或毫秒数，即使用 bufferMillis()或 bufferTimeoutMillis()两个方法

```java
@Test
public void bufferTimeout(){
    Flux.interval(Duration.ofMillis(100))
        .bufferTimeout(20, Duration.ofMillis(300))
        .take(10)
        .toStream().forEach(this::info);
}
```



通过 toStream()方法把 Flux 序列转换成 Java 8 中的 Stream 对象，再通过 forEach()方法来进行输出。这是因为**序列的生成是异步的，而转换成 Stream 对象可以保证主线程在序列生成完成之前不会退出，从而可以正确地输出序列中的所有元素**。



#### bufferUtil

除了元素数量和时间间隔之外，还可以通过 bufferUntil 和 bufferWhile 操作符来进行收集。这两个操作符的参数是表示每个集合中的元素所要满足的条件的 Predicate 对象。bufferUntil 会一直收集直到 Predicate 返回为 true。使得 Predicate 返回 true 的那个元素可以选择添加到当前集合或下一个集合中；bufferWhile 则只有当 Predicate 返回 true 时才会收集。一旦值为 false，会立即开始下一次收集。



#### bufferWhen



#### cache

将此Flux量转换为热源，并为进一步的用户缓存最后发射的信号。将保留一个无限量的OnNeXT信号。完成和错误也将被重放。

如其名缓存，相当于复制一份用于接下来的操作，而当前的流将会被缓存起来，用于之后的操作。

<img src="https://oss.xubighead.top/oss/image/202506/1930509912267395073.jpg" alt="img" style="zoom: 80%;" />



#### cancelOn

取消。



#### cast

将产生的Flux类型转换为目标产生类型。

<img src="https://oss.xubighead.top/oss/image/202506/1930509929631813633.jpg" alt="img" style="zoom:67%;" />



#### checkpoint

用于检测当前节点，流中是否存在错误。



#### collect

该系列实例方法，用于收集所有的元素到特定类型，如list、map等。处理完成时返回Mono。

通过应用收集器BiConsumer获取容器和每个元素，将此Flux发出的所有元素收集到用户定义的容器中。收集的结果将在这个序列完成时发出。

<img src="https://oss.xubighead.top/oss/image/202506/1930509943980527618.jpg" alt="img" style="zoom:67%;" />



#### collectList

将此Flux所发射的所有元素收集到序列完成时由结果Mono发出的列表。

<img src="https://oss.xubighead.top/oss/image/202506/1930509958350213121.jpg" alt="img" style="zoom:67%;" />



#### collectMap



#### collectMultimap



#### collectSortedList



#### concatMap

concatMap 操作符的作用也是把流中的每个元素转换成一个流，再把所有流进行合并。与 flatMap 不同的是，concatMap 会根据原始流中的元素顺序依次把转换之后的流进行合并；与 flatMapSequential 不同的是，concatMap 对转换之后的流的订阅是动态进行的，而 flatMapSequential 在合并之前就已经订阅了所有的流。

<img src="https://oss.xubighead.top/oss/image/202506/1930509977300078594.jpg" alt="img" style="zoom:67%;" />



```java
@Test
public void concatMap(){
    Flux.just(5, 10)
        .concatMap(x -> Flux.interval(Duration.ofMillis(x * 10), Duration.ofMillis(100)).take(x))
        .toStream()
        .forEach(System.out::println);
}
```

结果流中依次包含了第一个流和第二个流中的全部元素。



#### concatMapDelayError



#### concatWith

与concatMap不同，这是相加。



#### concatWithValues

将值连接到Flux的末尾。

<img src="https://oss.xubighead.top/oss/image/202506/1930509994446393345.jpg" alt="img" style="zoom:67%;" />



#### defaultIfEmpty

如果没有任何数据完成此序列，则提供默认的唯一值。

<img src="https://oss.xubighead.top/oss/image/202506/1930510009369726977.jpg" alt="img" style="zoom:67%;" />



#### distinct

对于每一个Subscriber，跟踪已经从这个Flux 跟踪元素和过滤出重复。值本身被记录到一个用于检测的哈希集中。如果希望使用distinct(Object::hashcode)更轻量级的方法，该方法不保留所有对象，但是更容易由于hashcode冲突而错误地认为两个元素是不同的。

<img src="https://oss.xubighead.top/oss/image/202506/1930510023781355522.jpg" alt="img" style="zoom:67%;" />



#### doOnCancel

当Flux被取消时触发附加行为（side-effect）。

<img src="https://oss.xubighead.top/oss/image/202506/1930510039379972098.jpg" alt="img" style="zoom:67%;" />



#### doOnComplete

当Flux完成时触发附加行为（side-effect）。

<img src="https://oss.xubighead.top/oss/image/202506/1930510053263118338.jpg" alt="img" style="zoom:67%;" />



#### doOnNext

当Flux发射一个项目时附加行为（side-effect）。

<img src="https://oss.xubighead.top/oss/image/202506/1930510068081594370.jpg" alt="img" style="zoom:67%;" />



#### elementAt

返回某一位置的值，类型为Mono<T>，可以设置默认值。



#### filter

根据给定谓词评估每个源值。如果谓词测试成功，则发出该值。如果谓词测试失败，则忽略该值，并在上游生成请求1。

<img src="https://oss.xubighead.top/oss/image/202506/1930510086616223745.jpg" alt="img" style="zoom:67%;" />



#### flatMap

flatMap 和 flatMapSequential 操作符**把流中的每个元素转换成一个流**，**再把所有流中的元素进行合并**。flatMapSequential 和 flatMap 之间的区别与 mergeSequential 和 merge 之间的区别是一样的。

```java
@Test
public void flatMapTest() throws InterruptedException {
    Function<Integer, Publisher<String>> mapper = i -> Flux.just(i * 2 + "");
    Flux.just(1, 2, 3, 4)
        .flatMap(mapper, 2)
        .subscribe(this::info);

    this.info("===================================================");
    Flux.just(5, 10)
        .flatMap(x -> Flux.interval(Duration.ofMillis(x * 10), Duration.ofMillis(100)).take(x))
        .toStream()
        .forEach(System.out::println);
}
```



流中的元素被转换成每隔 100 毫秒产生的数量不同的流，再进行合并。由于第一个流中包含的元素数量较少，所以在结果流中一开始是两个流的元素交织在一起，然后就只有第二个流中的元素。

`flatMap()` 和 `map()` 的区别在于，`flatMap()` 中的入参 `Function` 的返回值要求是一个 `Mono` 对象，而 `map` 的入参 `Function` 只要求返回一个 **普通对象**。在业务处理中常需要调用 `WebClient` 或 `ReactiveXxxRepository` 中的方法，这些方法的 **返回值** 都是 `Mono`（或 `Flux`）。所以要将这些调用串联为一个整体 **链式调用**，就必须使用 `flatMap()`，而不是 `map()`。



#### flatMapSequential



#### groupBy

<img src="https://oss.xubighead.top/oss/image/202506/1930510105113104385.jpg" alt="img" style="zoom:67%;" />

#### hasElement

表示是否存在该值。



#### hasElements

表示是否拥有一个或多个元素。



#### log

观察所有 Reactive Streams信号并使用记录器支持跟踪它们。默认值将使用Level.INFO和 java.util.logging日志记录。如果SLF4J是可用的，它将被用来代替。

<img src="https://oss.xubighead.top/oss/image/202506/1930510120980156417.jpg" alt="img" style="zoom:67%;" />



#### map

通过对每个项目应用同步功能来转换由该Flux发出的项目。

<img src="https://oss.xubighead.top/oss/image/202506/1930510137069506562.jpg" alt="img" style="zoom:67%;" />



#### reduce

reduce 和 reduceWith 操作符**对流中包含的所有元素进行累积操作，得到一个包含计算结果的 Mono 序列**。累积操作是通过一个 BiFunction 来表示的。在操作时可以指定一个初始值。如果没有初始值，则序列的第一个元素作为初始值。

```java
@Test
public void reduce() {
    // 对流中的元素进行相加操作，结果为 5050
    Flux.range(1, 100).reduce(Integer::sum).subscribe(this::info);

    Flux.range(1, 100).reduce(505, Integer::sum).subscribe(this::info);
}
```

```java
Data initData = ...;
List<SubData> list = ...;
Flux.fromIterable(list)
    .reduce(initData, (data, itemInList) -> {
        // Do something on data and itemInList
        return data;
    });
```



`<A> Mono<A> reduce(A initial, BiFunction<A, ? super T, A> accumulator);`可以看出，`reduce()` 方法的功能是将一个 `Flux` **聚合** 成一个 `Mono`。

- **第一个参数**: 返回值 `Mono` 中元素的 **初始值**。
- **第二个参数**: 是一个 `BiFunction`，用来实现 **聚合操作** 的逻辑。对于泛型参数 `<A, ? super T, A>` 中：
	- 第一个 `A`: 表示每次 **聚合操作** 之后的 **结果的类型**，它作为 `BiFunction.apply()` 方法的 **第一个入参**；
	- 第二个 `? super T`: 表示集合中的每个元素的类型，它作为 `BiFunction.apply()` 方法的 **第二个入参**；
	- 第三个 `A`: 表示聚合操作的 **结果**，它作为 `BiFunction.apply()` 方法的 **返回值**。



#### reduceWith

```java
@Test
public void reduceWithTest(){
    // 对流中的元素进行相加操作，不过通过一个 Supplier 给出了初始值为 100，所以结果为 5150
    Flux.range(1, 100).reduceWith(() -> 100, Integer::sum).subscribe(System.out::println);
}
```



#### subscribe

当需要处理 Flux 或 Mono 中的消息时，如之前的代码所示，**可以通过 subscribe 方法来添加相应的订阅逻辑**。在调用 subscribe 方法时可以指定需要处理的消息类型。可以**只处理其中包含的正常消息，也可以同时处理错误消息和完成消息**。

```jaava
@Test
public void subscribe(){
    // 通过 subscribe()方法处理正常和错误消息
    Flux.just(1, 2)
        .concatWith(Mono.error(new IllegalStateException("concatWith error")))
        .concatWith(Mono.just(3))
        .subscribe(this::info, e -> this.error("4", e), () -> this.info("success"));
}
```



#### take

take 系列操作符**用来从当前流中提取元素**。提取的方式可以有很多种。

```java
@Test
public void takeTest(){
    Flux.range(1, 1000).take(10).subscribe(this::info);
}
```



#### takeLast

提取流中的最后 N 个元素。

```java
@Test
public void takeLastTest(){
    Flux.range(1, 1000).takeLast(10).subscribe(this::info);
}
```



#### takeUtil

提取元素直到 Predicate 返回 true。

```java
@Test
public void takeUtilTest(){
    Flux.range(1, 1000).takeUntil(i -> i == 10).subscribe(this::info);
}
```



#### takeUtilOther

提取元素直到另外一个流开始产生元素。



#### takeWhile

当 Predicate 返回 true 时才进行提取。

```java
@Test
public void takeWhileTest(){
    Flux.range(1, 1000).takeWhile(i -> i < 10).subscribe(this::info);
}
```



#### then

让这个Flux完成，然后播放信号提供的Mono。换句话说，忽略这个Flux和转换完成信号的发射和提供Mono完成信号。在得到的Mono中重放错误信号。

<img src="https://oss.xubighead.top/oss/image/202506/1930510163174854658.jpg" alt="img" style="zoom:67%;" />



`then()` 看上去是下一步的意思，但它只表示执行顺序的下一步，不表示下一步依赖于上一步。`then()` 方法的参数只是一个 `Mono`，无从接受上一步的执行结果。而 `flatMap()` 和 `map()` 的参数都是一个 `Function`，入参是上一步的执行结果。

```java
@Test
public void then(){
    Flux.just(1).then(Mono.just(2)).subscribe(this::info);
}
```



#### window

window 操作符的作用类似于 buffer，所不同的是 window 操作符是把当前流中的元素收集到另外的 Flux 序列中，因此返回值类型是 Flux<flux>。



```java
@Test
public void windowTest(){
    Flux.range(1, 100).window(20).subscribe(this::info);
    Flux.interval(Duration.ofMillis(100))
        .window(Duration.ofMillis(1001))
        .take(2)
        .toStream().forEach(this::info);
}
```



在代码中，两行语句的输出结果分别是 5 个和 2 个 UnicastProcessor 字符。这是因为 window 操作符所产生的流中包含的是 UnicastProcessor 类的对象，而 UnicastProcessor 类的 toString 方法输出的就是 UnicastProcessor 字符。



#### zipWith

zipWith 操作符**把当前流中的元素与另外一个流中的元素按照一对一的方式进行合并**。在合并时可以不做任何处理，由此得到的是一个元素类型为 **Tuple2** 的流；也可以通过一个 **BiFunction** 函数对合并的元素进行处理，所得到的流的元素类型为该函数的返回值。



```java
@Test
public void zipWithTest(){
    Flux.just("a", "b")
        .zipWith(Flux.just("c", "d"))
        .subscribe(this::info);
    Flux.just("a", "b")
        .zipWith(Flux.just("c", "d"), (s1, s2) -> String.format("%s-%s", s1, s2))
        .subscribe(this::info);
}
```

```
[a,c]
[b,d]
a-c
b-d
```



在代码中，两个流中包含的元素分别是 a，b 和 c，d。第一个 zipWith 操作符**没有使用合并函数**，因此结果流中的元素类型为 **Tuple2**；第二个 zipWith 操作通过合并函数把元素类型变为 String。