---
layout: post
title: JDK FunctionalInterface.md
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
# Consumer\<T>

## 概述

`Consumer<T>`是一个函数式接口，其唯一的抽象方法接收一个参数T且无返回值，一般用于对参数T做消费行为，如输出到控制台、修改参数属性等。JDK中还提供了一些与`Consumer<T>`功能类似的特性化函数式接口，如`BiConsumer<T, U>`、`DoubleConsumer`和`ObjDoubleConsumer<T>`等接口。



## 源码解析

### 方法定义

```java
// @since 8
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    default Consumer<T> andThen(Consumer<? super T> after) {}
}
```



#### andThen

多个Consumer对象依次消费T类型参数。

```java
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```



## 实践应用



# Function\<T, R>

## 概述

`Function<T, R>`是一个函数式接口，其唯一的抽象方法接收一个参数T后返回类型R的值，一般用于对参数T做转换行为，如类型转换等。JDK中还提供了一些与`Function<T, R>`功能类似的特性化函数式接口，如`BiFunction<T, U, R>`、`DoubleToIntFunction`和`DoubleBinaryOperator`等接口。



## 源码解析

### 方法定义

```java
// @since 8
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {}
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {}
    static <T> Function<T, T> identity() {}
}
```



#### compose

接收一个Function实例，将V类型入参转换为当前Function实例所需要的T类型入参。

```java
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```



#### andThen

接收一个Function实例，将当前Function实例的R类型出参继续作为入参进行下一步转换。

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```



#### identity

不做任何转换，直接将入参作为出参返回。

```java
static <T> Function<T, T> identity() {
    return t -> t;
}
```



## 实践应用



# Predicate\<T>

## 概述

`Predicate<T>`是一个函数式接口，其唯一的抽象方法接收一个参数T后返回boolean类型的值，一般用于断言场景。JDK中还提供了一些与`Predicate<T, R>`功能类似的特性化函数式接口，如`BiPredicate<T, U>`、`DoublePredicate`和`IntPredicate`等接口。



## 源码解析

### 方法定义

```java
@FunctionalInterface
public interface Predicate<T> { // @since 8
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) {}
    default Predicate<T> negate() {}
    default Predicate<T> or(Predicate<? super T> other) {}
    static <T> Predicate<T> isEqual(Object targetRef) {}
    static <T> Predicate<T> not(Predicate<? super T> target) {} // @since 11
}
```



#### and



```java
default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) && other.test(t);
}
```



#### negate

```java
default Predicate<T> negate() {
    return (t) -> !test(t);
}
```



#### or

```java
default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) || other.test(t);
}
```



#### isEqual

```java
static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
            ? Objects::isNull
            : object -> targetRef.equals(object);
}
```



#### not

```java
static <T> Predicate<T> not(Predicate<? super T> target) {
    Objects.requireNonNull(target);
    return (Predicate<T>)target.negate();
}
```



## 实践应用



# Supplier\<T>

## 概述

`Supplier<T>`是一个函数式接口，其唯一的抽象方法不接受任何参数，然后返回T类型的值，一般用于获取指定数据。JDK中还提供了一些与`Supplier<T, R>`功能类似的特性化函数式接口，如`DoubleSupplier`、`IntSupplier`和`LongSupplier`等接口。



## 源码解析

### 方法定义

```java
// @since 8
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```



## 实践应用
