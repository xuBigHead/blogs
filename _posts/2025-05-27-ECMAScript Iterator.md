# ECMAScript Iterator

## 概述

遍历器（Iterator）是一种接口规格，任何对象只要部署这个接口，就可以完成遍历操作。它的作用有两个，一是为各种数据结构，提供一个统一的、简便的接口，二是使得对象的属性能够按某种次序排列。在ES6中，遍历操作特指for...of循环，即Iterator接口主要供for...of循环使用。

遍历器提供了一个指针，指向当前对象的某个属性，使用next方法，就可以将指针移动到下一个属性。next方法返回一个包含value和done两个属性的对象。其中，value属性是当前遍历位置的值，done属性是一个布尔值，表示遍历是否结束。下面是一个模拟next方法返回值的例子。

```ts
function makeIterator(array){
  var nextIndex = 0;
  return {
    next: function(){
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  }
}

var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }
```



上面代码定义了一个makeIterator函数，它的作用是返回一个遍历器对象，用来遍历参数数组。next方法依次遍历数组的每个成员，请特别注意，next返回值的构造。下面是一个无限运行的遍历器例子。

```ts
function idMaker(){
  var index = 0;

  return {
    next: function(){
      return {value: index++, done: false};
    }
  }
}

var it = idMaker();

it.next().value // '0'
it.next().value // '1'
it.next().value // '2'
// ...
```



上面的例子，只是为了说明next方法返回值的结构。Iterator接口返回的遍历器，原生具备next方法，不用自己部署。所以，真正需要部署的是Iterator接口，让其返回一个遍历器。在ES6中，有三类数据结构原生具备Iterator接口：数组、类似数组的对象（字符串）、Set和Map结构。除此之外，其他数据结构（主要是对象）的Iterator接口都需要自己部署。



## 源码解析

### 方法定义

```ts
// lib.es2015.iterable.d.ts
interface Iterator<T, TReturn = any, TNext = any> {
    // NOTE: 'next' is defined using a tuple to ensure we report the correct assignability errors in all places.
    next(...[value]: [] | [TNext]): IteratorResult<T, TReturn>;
    return?(value?: TReturn): IteratorResult<T, TReturn>;
    throw?(e?: any): IteratorResult<T, TReturn>;
}
```



## 实践应用

### 其他应用

#### 遍历字符串

字符串是一个类似数组的对象，因此也原生具有Iterator接口。下面代码中，调用`Symbol.iterator`方法返回一个遍历器，在这个遍历器上可以调用next方法，实现对于字符串的遍历。

```ts
var someString = "hi";
typeof someString[Symbol.iterator] // "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```



可以覆盖原生的`Symbol.iterator`方法，达到修改遍历器行为的目的。下面代码中，字符串str的`Symbol.iterator`方法被修改了，所以扩展运算符（...）返回的值变成了bye，而字符串本身还是hi。

```ts
var str = new String("hi");

[...str] // ["h", "i"]

str[Symbol.iterator] = function() {
  return { 
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};

console.log([...str]); // ["bye"]  
console.log(str); // "hi"
```



#### 自定义实现Iterator接口

下面代码是一个类部署Iterator接口的写法。`Symbol.iterator`是一个表达式，返回Symbol对象的iterator属性，这是一个预定义好的、类型为Symbol的特殊值，所以要放在方括号内。这里要注意，@@iterator的键名是`Symbol.iterator`，键值是一个方法（函数），该方法执行后，返回一个当前对象的遍历器。

```ts
lass MySpecialTree {
    // ...
    [Symbol.iterator]() { 
        // ...
        return theIterator;
    }
}
```



对象自定义实现Iterator接口：

```ts
let obj = {
    data: [ 'hello', 'world' ],
    [Symbol.iterator]() {
        const self = this;
        let index = 0;
        return {
            next() {
                if (index < self.data.length) {
                    return {
                        value: self.data[index++],
                        done: false
                    };
                } else {
                    return { value: undefined, done: true };
                }
            }
        };
    }
};
```



如果`Symbol.iterator`方法返回的不是遍历器，解释引擎将会报错。下面代码中，变量obj的iterator方法返回的不是遍历器，因此报错。

```ts
var obj = {};
obj[Symbol.iterator] = () => 1;
[...obj] // TypeError: [] is not a function
```



#### 通过Generator函数实现Iterator接口

部署`Symbol.iterator`方法的最简单实现方式是使用下一节要提到的Generator函数。

```ts
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
};
[...myIterable] // [1, 2, 3]

// 或者采用下面的简洁写法
let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x); // hello world
}
```

