---
layout: post
title: 2025-03-18-第024章-JDK Optional
categories: [Java]
description: 
keywords: JDK Optional.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Optional

## 概述

> @since 8



Optional容器类用于避免空指针异常。



## 源码解析

```java
public final class Optional<T> {
    private static final Optional<?> EMPTY = new Optional<>();
    private final T value;
}
```



### 方法定义

```java
// @since 8
public final class Optional<T> {
    // 将内部T类型的值转换为另一个类型U的值
    public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {}
    // 将当前对象转换为另一个包含指定类型U的值的Optional对象，用于将嵌套Optional展平
    public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper) {}
    // @since 9 
	public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) {}
    // @since 9
    public static <T> Optional<T> of(T value) {}
    // @since 9
    public Stream<T> stream() {}
    // @since 10 值不存在时抛出默认异常NoSuchElementException("No value present")；
    public T orElseThrow() {}
    // @since 11 判断值是否为null，为null时返回true；
    public boolean isEmpty() {}
}
```



### 方法说明

#### constructor

```java
private Optional() {
    this.value = null;
}
```



```java
// 私有有参构造器，参数不能为null
private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}
```



由于Optional私有化了空构造器，因此只能调用其提供的方法来实例化Optional类。


```java
    // 获取空数据的Optional容器类实例
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    // of方法生成Optional实例，参数不能为null
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
    
    // ofNullable方法生成Optional实例，参数允许为null
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```





#### get

获取封装对象，如果为null则抛出异常

```java
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```



#### isPresent

判断封装对象是否为null

```java
    public boolean isPresent() {
        return value != null;
    }
```

```java
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

```java
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```

```java
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

```java
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```

```java
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

```java
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

```java
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```



#### ifPresentOrElse

> @since 9



`ifPresentOrElse()`方法接受两个参数`Consumer`和`Runnable`，如果`Optional`不为空调用`Consumer`参数，为空则调用`Runnable`参数。

```java
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) {
    if (value != null) {
        action.accept(value);
    } else {
        emptyAction.run();
    }
}
```



#### map

```java
public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (isEmpty()) {
        return empty();
    } else {
        return Optional.ofNullable(mapper.apply(value));
    }
}
```



#### flatMap

```java
public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper) {
    Objects.requireNonNull(mapper);
    if (isEmpty()) {
        return empty();
    } else {
        @SuppressWarnings("unchecked")
        Optional<U> r = (Optional<U>) mapper.apply(value);
        return Objects.requireNonNull(r);
    }
}
```



#### or

> @since 9



`or()`方法接受一个`Supplier`参数，如果`Optional`为空则返回`Supplier`参数指定的`Optional`值。

```java
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier) {
    Objects.requireNonNull(supplier);
    if (isPresent()) {
        return this;
    } else {
        @SuppressWarnings("unchecked")
        Optional<T> r = (Optional<T>) supplier.get();
        return Objects.requireNonNull(r);
    }
}
```



#### stream

> @since 9

```java
public Stream<T> stream() {
    if (!isPresent()) {
        return Stream.empty();
    } else {
        return Stream.of(value);
    }
}
```



#### orElseThrow

> @since 10

```java
public T orElseThrow() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```



## 实践应用