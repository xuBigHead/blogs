# ECMAScript Error

## 概述

Error 对象用来表示代码运行时的异常情况，但是从这个对象拿到的上下文信息，有时很难解读，也不够充分。[ES2022](https://github.com/tc39/proposal-error-cause) 为 Error 对象添加了一个`cause`属性，可以在生成错误时，添加报错原因的描述。

它的用法是`new Error()`生成 Error 实例时，给出一个描述对象，该对象可以设置`cause`属性。

```javascript
const actual = new Error('an error!', { cause: 'Error cause' });
actual.cause; // 'Error cause'
```

上面示例中，生成 Error 实例时，使用描述对象给出`cause`属性，写入报错的原因。然后，就可以从实例对象上读取这个属性。

`cause`属性可以放置任意内容，不必一定是字符串。

```javascript
try {
  maybeWorks();
} catch (err) {
  throw new Error('maybeWorks failed!', { cause: err });
}
```

上面示例中，`cause`属性放置的就是一个对象。



## 源码解析

### 方法定义

```ts
// lib.es5.d.ts
interface Error {
    name: string;
    message: string;
    stack?: string;
}

interface ErrorConstructor {
    new (message?: string): Error;
    (message?: string): Error;
    readonly prototype: Error;
}

declare var Error: ErrorConstructor;
```



```ts
// lib.es2022.error.d.ts
interface Error {
    cause?: unknown;
}

interface ErrorConstructor {
    new (message?: string, options?: ErrorOptions): Error;
    (message?: string, options?: ErrorOptions): Error;
}
```

