# ECMAScript AggregateError

## 概述

ES2021 标准之中，为了配合新增的`Promise.any()`方法（详见《Promise 对象》一章），还引入一个新的错误对象`AggregateError`，也放在这一章介绍。

AggregateError 在一个错误对象里面，封装了多个错误。如果某个单一操作，同时引发了多个错误，需要同时抛出这些错误，那么就可以抛出一个 AggregateError 错误对象，把各种错误都放在这个对象里面。

AggregateError 本身是一个构造函数，用来生成 AggregateError 实例对象。

```javascript
AggregateError(errors[, message])
```

`AggregateError()`构造函数可以接受两个参数。

- errors：数组，它的每个成员都是一个错误对象。该参数是必须的。
- message：字符串，表示 AggregateError 抛出时的提示信息。该参数是可选的。

```javascript
const error = new AggregateError([
  new Error('ERROR_11112'),
  new TypeError('First name must be a string'),
  new RangeError('Transaction value must be at least 1'),
  new URIError('User profile link must be https'),
], 'Transaction cannot be processed')
```

上面示例中，`AggregateError()`的第一个参数数组里面，一共有四个错误实例。第二个参数字符串则是这四个错误的一个整体的提示。

`AggregateError`的实例对象有三个属性。

- name：错误名称，默认为“AggregateError”。
- message：错误的提示信息。
- errors：数组，每个成员都是一个错误对象。

下面是一个示例。

```javascript
try {
  throw new AggregateError([
    new Error("some error"),
  ], 'Hello');
} catch (e) {
  console.log(e instanceof AggregateError); // true
  console.log(e.message);                   // "Hello"
  console.log(e.name);                      // "AggregateError"
  console.log(e.errors);                    // [ Error: "some error" ]
}
```



## 源码解析

### 方法定义

```ts
// lib.es2021.promise.d.ts
interface AggregateError extends Error {
    errors: any[];
}

interface AggregateErrorConstructor {
    new (errors: Iterable<any>, message?: string): AggregateError;
    (errors: Iterable<any>, message?: string): AggregateError;
    readonly prototype: AggregateError;
}

declare var AggregateError: AggregateErrorConstructor;
```



## 实践应用