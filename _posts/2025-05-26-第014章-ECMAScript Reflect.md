---
layout: post
title: 2025-05-26-第014章-ECMAScript Reflect.md
categories: [ECMAScript]
description: 
keywords: ECMAScript Reflect.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ECMAScript Reflect

## 概述

> @since ES6



ES6 中将 Object 的一些明显属于语言内部的方法移植到了 Reflect 对象上（当前某些方法会同时存在于 Object 和 Reflect 对象上），未来的新方法会只部署在 Reflect 对象上。

Reflect可以用于获取目标对象的行为，它与Object类似，但是更易读，为操作对象提供了一种更优雅的方式。它的方法与Proxy是对应的。



`Reflect`对象与`Proxy`对象一样，也是 ES6 为了操作对象而提供的新 API。`Reflect`对象的设计目的有这样几个。

（1） 将`Object`对象的一些明显属于语言内部的方法（比如`Object.defineProperty`），放到`Reflect`对象上。现阶段，某些方法同时在`Object`和`Reflect`对象上部署，未来的新方法将只部署在`Reflect`对象上。也就是说，从`Reflect`对象上可以拿到语言内部的方法。

（2） 修改某些`Object`方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。

```javascript
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```

（3） 让`Object`操作都变成函数行为。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。

```javascript
// 老写法
'assign' in Object // true

// 新写法
Reflect.has(Object, 'assign') // true
```

（4）`Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

```javascript
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target, name, value, receiver);
    if (success) {
      console.log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

上面代码中，`Proxy`方法拦截`target`对象的属性赋值行为。它采用`Reflect.set`方法将值赋值给对象的属性，确保完成原有的行为，然后再部署额外的功能。

下面是另一个例子。

```javascript
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log('has' + name);
    return Reflect.has(target, name);
  }
});
```

上面代码中，每一个`Proxy`对象的拦截操作（`get`、`delete`、`has`），内部都调用对应的`Reflect`方法，保证原生行为能够正常执行。添加的工作，就是将每一个操作输出一行日志。

有了`Reflect`对象以后，很多操作会更易读。

```javascript
// 老写法
Function.prototype.apply.call(Math.floor, undefined, [1.75]) // 1

// 新写法
Reflect.apply(Math.floor, undefined, [1.75]) // 1
```



`Reflect`对象一共有 13 个静态方法。

- Reflect.apply(target, thisArg, args)
- Reflect.construct(target, args)
- Reflect.get(target, name, receiver)
- Reflect.set(target, name, value, receiver)
- Reflect.defineProperty(target, name, desc)
- Reflect.deleteProperty(target, name)
- Reflect.has(target, name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)

上面这些方法的作用，大部分与`Object`对象的同名方法的作用都是相同的，而且它与`Proxy`对象的方法是一一对应的。下面是对它们的解释。



## 源码解析

### 函数定义

> @since ES6

```ts
declare namespace Reflect {

    function apply<T, A extends readonly any[], R>(
        target: (this: T, ...args: A) => R,
        thisArgument: T,
        argumentsList: Readonly<A>,
    ): R;
    function apply(target: Function, thisArgument: any, argumentsList: ArrayLike<any>): any;

    function construct<A extends readonly any[], R>(
        target: new (...args: A) => R,
        argumentsList: Readonly<A>,
        newTarget?: new (...args: any) => any,
    ): R;
    function construct(target: Function, argumentsList: ArrayLike<any>, newTarget?: Function): any;

    function defineProperty(target: object, propertyKey: PropertyKey, attributes: PropertyDescriptor & ThisType<any>): boolean;

    function deleteProperty(target: object, propertyKey: PropertyKey): boolean;

    function get<T extends object, P extends PropertyKey>(
        target: T,
        propertyKey: P,
        receiver?: unknown,
    ): P extends keyof T ? T[P] : any;

    function getOwnPropertyDescriptor<T extends object, P extends PropertyKey>(
        target: T,
        propertyKey: P,
    ): TypedPropertyDescriptor<P extends keyof T ? T[P] : any> | undefined;

    function getPrototypeOf(target: object): object | null;

    function has(target: object, propertyKey: PropertyKey): boolean;

    function isExtensible(target: object): boolean;

    function ownKeys(target: object): (string | symbol)[];

    function preventExtensions(target: object): boolean;

    function set<T extends object, P extends PropertyKey>(
        target: T,
        propertyKey: P,
        value: P extends keyof T ? T[P] : any,
        receiver?: any,
    ): boolean;
    function set(target: object, propertyKey: PropertyKey, value: any, receiver?: any): boolean;

    function setPrototypeOf(target: object, proto: object | null): boolean;
}
```



## 实践应用

### 静态方法

#### apply

`Reflect.apply(func, thisArg, args)` 等同于Function.prototype.apply.call(func, thisArg, args) 。func表示目标函数；thisArg表示目标函数绑定的this对象；args表示目标函数调用时传入的参数列表，可以是数组或类似数组的对象。若目标函数无法调用，会抛出TypeError 。

```ts
Reflect.apply(Math.max, Math, [1, 3, 5, 3, 1]); // 5
```



`Reflect.apply`方法等同于`Function.prototype.apply.call(func, thisArg, args)`，用于绑定`this`对象后执行给定函数。

一般来说，如果要绑定一个函数的`this`对象，可以这样写`fn.apply(obj, args)`，但是如果函数定义了自己的`apply`方法，就只能写成`Function.prototype.apply.call(fn, obj, args)`，采用`Reflect`对象可以简化这种操作。

```javascript
const ages = [11, 33, 12, 54, 18, 96];

// 旧写法
const youngest = Math.min.apply(Math, ages);
const oldest = Math.max.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);

// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
const oldest = Reflect.apply(Math.max, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngest, []);
```



#### construct

`Reflect.construct(obj, args)` 等同于 new target(...args)。

```ts
function person(name){
    this.name = name;
}
Reflect.construct(person, ['Tom']); // person {name: "Tom"}
```



`Reflect.construct`方法等同于`new target(...args)`，这提供了一种不使用`new`，来调用构造函数的方法。

```javascript
function Greeting(name) {
  this.name = name;
}

// new 的写法
const instance = new Greeting('张三');

// Reflect.construct 的写法
const instance = Reflect.construct(Greeting, ['张三']);
```

如果`Reflect.construct()`方法的第一个参数不是函数，会报错。



#### defineProperty

`Reflect.defineProperty(target, propertyKey, attributes)` 用于为目标对象定义属性。如果target不是对象，会抛出错误。

```ts
let myDate= {}
Reflect.defineProperty(myDate, 'now', {
  value: () => Date.now()
}); // true

const student = {};
Reflect.defineProperty(student, "name", {value: "Mike"}); // true
student.name; // "Mike"
```



`Reflect.defineProperty`方法基本等同于`Object.defineProperty`，用来为对象定义属性。未来，后者会被逐渐废除，请从现在开始就使用`Reflect.defineProperty`代替它。

```javascript
function MyDate() {
  /*…*/
}

// 旧写法
Object.defineProperty(MyDate, 'now', {
  value: () => Date.now()
});

// 新写法
Reflect.defineProperty(MyDate, 'now', {
  value: () => Date.now()
});
```

如果`Reflect.defineProperty`的第一个参数不是对象，就会抛出错误，比如`Reflect.defineProperty(1, 'foo')`。

这个方法可以与`Proxy.defineProperty`配合使用。

```javascript
const p = new Proxy({}, {
  defineProperty(target, prop, descriptor) {
    console.log(descriptor);
    return Reflect.defineProperty(target, prop, descriptor);
  }
});

p.foo = 'bar';
// {value: "bar", writable: true, enumerable: true, configurable: true}

p.foo // "bar"
```

上面代码中，`Proxy.defineProperty`对属性赋值设置了拦截，然后使用`Reflect.defineProperty`完成了赋值。



#### deleteProperty

`Reflect.deleteProperty(obj, property)` 是 `delete obj[property]` 的函数化，用于删除obj对象的property属性，返回值类型为boolean。如果obj不是对象则会报错TypeError。

```ts
let exam = {
    name: "Tom",
    age: 24
}
Reflect.deleteProperty(exam , 'name'); // true
exam // {age: 24} 
// property 不存在时，也会返回 true
Reflect.deleteProperty(exam , 'name'); // true
```



`Reflect.deleteProperty`方法等同于`delete obj[name]`，用于删除对象的属性。

```javascript
const myObj = { foo: 'bar' };

// 旧写法
delete myObj.foo;

// 新写法
Reflect.deleteProperty(myObj, 'foo');
```

该方法返回一个布尔值。如果删除成功，或者被删除的属性不存在，返回`true`；删除失败，被删除的属性依然存在，返回`false`。

如果`Reflect.deleteProperty()`方法的第一个参数不是对象，会报错。



#### get

`Reflect.get(target, name, receiver)` 查找并返回target对象的name属性。

```ts
let exam = {
    name: "Tom",
    age: 24,
    get info(){
        return this.name + this.age;
    }
}
let name = Reflect.get(exam, 'name'); // "Tom"
console.log(name);

// 当target对象中存在name属性的getter方法，getter方法的this会绑定receiver
let receiver = {
    name: "Jerry",
    age: 20
}
let info = Reflect.get(exam, 'info', receiver); // Jerry20
console.log(info);

// 当name为不存在于target对象的属性时，返回undefined
let birth = Reflect.get(exam, 'birth'); // undefined
console.log(birth);

// 当target不是对象时，会报错
Reflect.get(1, 'name'); // TypeError
```



`Reflect.get`方法查找并返回`target`对象的`name`属性，如果没有该属性，则返回`undefined`。

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
}

Reflect.get(myObject, 'foo') // 1
Reflect.get(myObject, 'bar') // 2
Reflect.get(myObject, 'baz') // 3
```

如果`name`属性部署了读取函数（getter），则读取函数的`this`绑定`receiver`。

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
};

var myReceiverObject = {
  foo: 4,
  bar: 4,
};

Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

如果第一个参数不是对象，`Reflect.get`方法会报错。

```javascript
Reflect.get(1, 'foo') // 报错
Reflect.get(false, 'foo') // 报错
```



#### getOwnPropertyDescriptor

`Reflect.getOwnPropertyDescriptor(target, propertyKey)` 用于得到target对象的propertyKey属性的描述对象。在target不是对象时，会抛出错误表示参数非法，不会将非对象转换为对象。

```ts
var person = {}
Reflect.defineProperty(person, 'name', {
  value: true,
  enumerable: false,
})
Reflect.getOwnPropertyDescriptor(person, 'name') // { configurable: false, enumerable: false, value: true, writable: false}

// propertyKey 属性在 target 对象中不存在时，返回 undefined
Reflect.getOwnPropertyDescriptor(person, 'age') // undefined
```



`Reflect.getOwnPropertyDescriptor`基本等同于`Object.getOwnPropertyDescriptor`，用于得到指定属性的描述对象，将来会替代掉后者。

```javascript
var myObject = {};
Object.defineProperty(myObject, 'hidden', {
  value: true,
  enumerable: false,
});

// 旧写法
var theDescriptor = Object.getOwnPropertyDescriptor(myObject, 'hidden');

// 新写法
var theDescriptor = Reflect.getOwnPropertyDescriptor(myObject, 'hidden');
```

`Reflect.getOwnPropertyDescriptor`和`Object.getOwnPropertyDescriptor`的一个区别是，如果第一个参数不是对象，`Object.getOwnPropertyDescriptor(1, 'foo')`不报错，返回`undefined`，而`Reflect.getOwnPropertyDescriptor(1, 'foo')`会抛出错误，表示参数非法。



#### getPrototypeOf

`Reflect.getPrototypeOf(obj)` 用于读取obj的\_proto_属性。在obj不是对象时不会像Object一样把obj转为对象，而是会报错。

```ts
class Exam{}
let obj = new Exam()
Reflect.getPrototypeOf(obj) === Exam.prototype // true
```



`Reflect.getPrototypeOf`方法用于读取对象的`__proto__`属性，对应`Object.getPrototypeOf(obj)`。

```javascript
const myObj = new FancyThing();

// 旧写法
Object.getPrototypeOf(myObj) === FancyThing.prototype;

// 新写法
Reflect.getPrototypeOf(myObj) === FancyThing.prototype;
```

`Reflect.getPrototypeOf`和`Object.getPrototypeOf`的一个区别是，如果参数不是对象，`Object.getPrototypeOf`会将这个参数转为对象，然后再运行，而`Reflect.getPrototypeOf`会报错。

```javascript
Object.getPrototypeOf(1) // Number {[[PrimitiveValue]]: 0}
Reflect.getPrototypeOf(1) // 报错
```



#### has

`Reflect.has(obj, name)` 是 `name in obj` 指令的函数化，用于查找name属性在obj对象中是否存在，返回值类型为boolean。如果obj不是对象则会报错TypeError。

```ts
let exam = {
    name: "Tom",
    age: 24
}
Reflect.has(exam, 'name'); // true
```



`Reflect.has`方法对应`name in obj`里面的`in`运算符。

```javascript
var myObject = {
  foo: 1,
};

// 旧写法
'foo' in myObject // true

// 新写法
Reflect.has(myObject, 'foo') // true
```

如果`Reflect.has()`方法的第一个参数不是对象，会报错。



#### isExtensible

`Reflect.isExtensible(target)` 用于判断target对象是否可扩展，返回值为boolean。如果target参数不是对象，会抛出错误。

```ts
let exam = {}
Reflect.isExtensible(exam) // true
```



`Reflect.isExtensible`方法对应`Object.isExtensible`，返回一个布尔值，表示当前对象是否可扩展。

```javascript
const myObject = {};

// 旧写法
Object.isExtensible(myObject) // true

// 新写法
Reflect.isExtensible(myObject) // true
```

如果参数不是对象，`Object.isExtensible`会返回`false`，因为非对象本来就是不可扩展的，而`Reflect.isExtensible`会报错。

```javascript
Object.isExtensible(1) // false
Reflect.isExtensible(1) // 报错
```



#### ownKeys

`Reflect.ownKeys(target)` 用于返回target对象的所有属性，等同于Object.getOwnPropertyNames与Object.getOwnPropertySymbols之和。

```ts
var exam = {
  name: 1,
  [Symbol.for('age')]: 4
}
Reflect.ownKeys(exam) // ["name", Symbol(age)]
```



`Reflect.ownKeys`方法用于返回对象的所有属性，基本等同于`Object.getOwnPropertyNames`与`Object.getOwnPropertySymbols`之和。

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  [Symbol.for('baz')]: 3,
  [Symbol.for('bing')]: 4,
};

// 旧写法
Object.getOwnPropertyNames(myObject)
// ['foo', 'bar']

Object.getOwnPropertySymbols(myObject)
//[Symbol(baz), Symbol(bing)]

// 新写法
Reflect.ownKeys(myObject)
// ['foo', 'bar', Symbol(baz), Symbol(bing)]
```

如果`Reflect.ownKeys()`方法的第一个参数不是对象，会报错。



#### preventExtensions

`Reflect.preventExtensions(target)` 用于让target对象变为不可扩展。如果target参数不是对象，会抛出错误。

```ts
let exam = {}
Reflect.preventExtensions(exam) // true
```



`Reflect.preventExtensions`对应`Object.preventExtensions`方法，用于让一个对象变为不可扩展。它返回一个布尔值，表示是否操作成功。

```javascript
var myObject = {};

// 旧写法
Object.preventExtensions(myObject) // Object {}

// 新写法
Reflect.preventExtensions(myObject) // true
```

如果参数不是对象，`Object.preventExtensions`在 ES5 环境报错，在 ES6 环境返回传入的参数，而`Reflect.preventExtensions`会报错。

```javascript
// ES5 环境
Object.preventExtensions(1) // 报错

// ES6 环境
Object.preventExtensions(1) // 1

// 新写法
Reflect.preventExtensions(1) // 报错
```



#### set

`Reflect.set(target, name, value, receiver)` 将target的name属性设置为value。返回值为boolean，true表示修改成功，false表示失败。当target为不存在的对象时，会报错。

```ts
let exam = {
    name: "Tom",
    age: 24,
    set info(value){
        return this.age = value;
    }
}
console.log(exam.age); // 24
Reflect.set(exam, 'age', 25); // true
console.log(exam.age); // 25
 
// value 为空时会将 name 属性清除
Reflect.set(exam, 'age', ); // true
console.log(exam.age); // undefined
 
// 当target对象中存在name属性setter方法时，setter方法中的this会绑定receiver, 所以修改的实际上是receiver的属性,
let receiver = {
    age: 18
}
Reflect.set(exam, 'info', 1, receiver); // true
console.log(receiver.age); // 1
 
let receiver1 = {
    name: 'oppps'
}
Reflect.set(exam, 'info', 1, receiver1);
console.log(receiver1.age); // 1
```



`Reflect.set`方法设置`target`对象的`name`属性等于`value`。

```javascript
var myObject = {
  foo: 1,
  set bar(value) {
    return this.foo = value;
  },
}

myObject.foo // 1

Reflect.set(myObject, 'foo', 2);
myObject.foo // 2

Reflect.set(myObject, 'bar', 3)
myObject.foo // 3
```

如果`name`属性设置了赋值函数，则赋值函数的`this`绑定`receiver`。

```javascript
var myObject = {
  foo: 4,
  set bar(value) {
    return this.foo = value;
  },
};

var myReceiverObject = {
  foo: 0,
};

Reflect.set(myObject, 'bar', 1, myReceiverObject);
myObject.foo // 4
myReceiverObject.foo // 1
```

注意，如果 `Proxy`对象和 `Reflect`对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，而且传入了`receiver`，那么`Reflect.set`会触发`Proxy.defineProperty`拦截。

```javascript
let p = {
  a: 'a'
};

let handler = {
  set(target, key, value, receiver) {
    console.log('set');
    Reflect.set(target, key, value, receiver)
  },
  defineProperty(target, key, attribute) {
    console.log('defineProperty');
    Reflect.defineProperty(target, key, attribute);
  }
};

let obj = new Proxy(p, handler);
obj.a = 'A';
// set
// defineProperty
```

上面代码中，`Proxy.set`拦截里面使用了`Reflect.set`，而且传入了`receiver`，导致触发`Proxy.defineProperty`拦截。这是因为`Proxy.set`的`receiver`参数总是指向当前的 `Proxy`实例（即上例的`obj`），而`Reflect.set`一旦传入`receiver`，就会将属性赋值到`receiver`上面（即`obj`），导致触发`defineProperty`拦截。如果`Reflect.set`没有传入`receiver`，那么就不会触发`defineProperty`拦截。

```javascript
let p = {
  a: 'a'
};

let handler = {
  set(target, key, value, receiver) {
    console.log('set');
    Reflect.set(target, key, value)
  },
  defineProperty(target, key, attribute) {
    console.log('defineProperty');
    Reflect.defineProperty(target, key, attribute);
  }
};

let obj = new Proxy(p, handler);
obj.a = 'A';
// set
```

如果第一个参数不是对象，`Reflect.set`会报错。

```javascript
Reflect.set(1, 'foo', {}) // 报错
Reflect.set(false, 'foo', {}) // 报错
```



#### setPrototypeOf

`Reflect.setPrototypeOf(obj, newProto)` 用于设置目标对象的 prototype。

```ts
let obj ={}
Reflect.setPrototypeOf(obj, Array.prototype); // true
```



`Reflect.setPrototypeOf`方法用于设置目标对象的原型（prototype），对应`Object.setPrototypeOf(obj, newProto)`方法。它返回一个布尔值，表示是否设置成功。

```javascript
const myObj = {};

// 旧写法
Object.setPrototypeOf(myObj, Array.prototype);

// 新写法
Reflect.setPrototypeOf(myObj, Array.prototype);

myObj.length // 0
```

如果无法设置目标对象的原型（比如，目标对象禁止扩展），`Reflect.setPrototypeOf`方法返回`false`。

```javascript
Reflect.setPrototypeOf({}, null)
// true
Reflect.setPrototypeOf(Object.freeze({}), null)
// false
```

如果第一个参数不是对象，`Object.setPrototypeOf`会返回第一个参数本身，而`Reflect.setPrototypeOf`会报错。

```javascript
Object.setPrototypeOf(1, {})
// 1

Reflect.setPrototypeOf(1, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

如果第一个参数是`undefined`或`null`，`Object.setPrototypeOf`和`Reflect.setPrototypeOf`都会报错。

```javascript
Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Reflect.setPrototypeOf(null, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```



## 参考资料

- [Reflect - ECMAScript 6入门](https://es6.ruanyifeng.com/#docs/reflect)