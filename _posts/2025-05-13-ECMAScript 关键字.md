---
layout: post
title: 第003章-ECMAScript 关键字
categories: [ECMAScript]
description: 
keywords: ECMAScript 关键字.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ECMAScript 关键字

## var关键字

### 概述

var关键字具备如下特性：

- var声明的变量是在全局有效；



### 特性

#### 全局有效

```ts
{
  var b = 1;
}
b 	// 1
```



注意：在for循环中声明的var变量是在全局有效的，每次循环时setTimeout定时器里面的i指的是全局变量i，而循环里的十个setTimeout是在循环结束后才执行，所以此时的 i都是10。

```ts
for (var i = 0; i < 10; i++) {
  setTimeout(function(){
    console.log(i); // 输出十个 10
  })
}
```



#### 允许重复声明

var声明的变量可以多次声明；

```ts
var b = 3;
var b = 4;
b  // 4
```



#### 具备变量提升

var声明的变量会变量提升。变量b用var声明存在变量提升，所以当脚本开始运行的时候，b已经存在了，但是还没有赋值，所以会输出undefined。

```ts
console.log(b);  //undefined
var b = "banana";
```



## let关键字

> @since ES6



### 概述

let关键字具备如下特性：

- `let`声明的变量只在`let`命令所在的代码块内有效；
- let声明的变量不能重复声明；



### 特性

#### 所在代码块内有效

`let`声明的变量只在`let`命令所在的代码块内有效。相比于var关键字更适合用于for循环。

```ts
{
    let a = 0;
    a   // 0
}
a   // 报错 ReferenceError: a is not defined
```



#### 不允许重复声明

let声明的变量不能重复声明；

```ts
let a = 1;
let a = 2;
a  // Identifier 'a' has already been declared
```



#### 不具备变量提升

let关键词声明的变量不具备变量提升（hoisting）特性，在声明变量a之前，a不存在，所以会报错。

```ts
console.log(a);  //ReferenceError: a is not defined
let a = "apple";
```



#### 暂时性死区

只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。下面代码中，存在全局变量`tmp`，但是块级作用域内`let`又声明了一个局部变量`tmp`，导致后者绑定这个块级作用域，所以在`let`声明变量前，对`tmp`赋值会报错。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```



“暂时性死区”也意味着`typeof`不再是一个百分之百安全的操作。下面代码中，变量`x`使用`let`命令声明，所以在声明之前，都属于`x`的“死区”，只要用到该变量就会报错。因此，`typeof`运行时就会抛出一个`ReferenceError`。

```javascript
typeof x; // ReferenceError
let x;
```



如果一个变量根本没有被声明，使用`typeof`反而不会报错。下面代码中，`undeclared_variable`是一个不存在的变量名，结果返回“undefined”。所以，在没有`let`之前，`typeof`运算符是百分之百安全的，永远不会报错。

```javascript
typeof undeclared_variable // "undefined"
```



有些“死区”比较隐蔽，不太容易发现。下面代码中，调用`bar`函数之所以报错，是因为参数`x`默认值等于另一个参数`y`，而此时`y`还没有声明，属于“死区”。

```javascript
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 报错
```



如果`y`的默认值是`x`，就不会报错，因为此时`x`已经声明了。

```javascript
function bar(x = 2, y = x) {
  return [x, y];
}
bar(); // [2, 2]
```



另外，下面的代码也会报错，与`var`的行为不同。下面代码报错，也是因为暂时性死区。使用`let`声明变量时，只要变量在还没有声明完成前使用，就会报错。上面这行就属于这个情况，在变量`x`的声明语句还没有执行完成前，就去取`x`的值，导致报错”x 未定义“。

```javascript
// 不报错
var x = x;

// 报错
let x = x;
// ReferenceError: x is not defined
```



ES6 规定暂时性死区和`let`、`const`语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在 ES5 是很常见的，现在有了这种规定，避免此类错误就很容易了。

总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。



## const关键字

> @since ES6



### 概述

 const保证的不是变量的值不变，而是保证变量指向的内存地址所保存的数据不允许改动。简单类型和复合类型保存值的方式是不同的，对于简单类型（number、string 、boolean）,值就保存在变量指向的那个内存地址，因此const声明的简单类型变量等同于常量。而复杂类型（object，array，function），变量指向的内存地址其实是保存了一个指向实际数据的指针，所以const只能保证指针是固定的，至于指针指向的数据结构变不变就无法控制了，所以使用const声明复杂类型对象时要慎重。如果真的想将对象冻结，应该使用`Object.freeze`方法。



const关键字具备如下特性：

- const声明一个只读的常量，一旦声明必须初始化值。



### 特性

```ts
const PI = "3.1415926";
PI  // 3.1415926

const MY_AGE;  // SyntaxError: Missing initializer in const declaration
```



#### 所在代码块内有效

`const`的作用域与`let`命令相同：只在声明所在的块级作用域内有效。



#### 不具备常量提升

`const`命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。



#### 暂时性死区

const关键字声明的常量和let关键字声明的变量都具备暂时性死区的特性。



## typeof关键字



## try-catch关键字

JavaScript 语言的`try...catch`结构，以前明确要求`catch`命令后面必须跟参数，接受`try`代码块抛出的错误对象。

```javascript
try {
    // ...
} catch (err) {
    // 处理错误
}
```

上面代码中，`catch`命令后面带有参数`err`。



很多时候，`catch`代码块可能用不到这个参数。但是，为了保证语法正确，还是必须写。ES2019 做出了改变，允许`catch`语句省略参数。

```javascript
try {
    // ...
} catch {
    // ...
}
```



## this关键字

## super关键字

我们知道，`this`关键字总是指向函数所在的当前对象，ES6 又新增了另一个类似的关键字`super`，指向当前对象的原型对象。

```javascript
const proto = {
  foo: 'hello'
};

const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find() // "hello"
```

上面代码中，对象`obj.find()`方法之中，通过`super.foo`引用了原型对象`proto`的`foo`属性。

注意，`super`关键字表示原型对象时，只能用在对象的方法之中，用在其他地方都会报错。

```javascript
// 报错
const obj = {
  foo: super.foo
}

// 报错
const obj = {
  foo: () => super.foo
}

// 报错
const obj = {
  foo: function () {
    return super.foo
  }
}
```

上面三种`super`的用法都会报错，因为对于 JavaScript 引擎来说，这里的`super`都没有用在对象的方法之中。第一种写法是`super`用在属性里面，第二种和第三种写法是`super`用在一个函数里面，然后赋值给`foo`属性。目前，只有对象方法的简写法可以让 JavaScript 引擎确认，定义的是对象的方法。

JavaScript 引擎内部，`super.foo`等同于`Object.getPrototypeOf(this).foo`（属性）或`Object.getPrototypeOf(this).foo.call(this)`（方法）。

```javascript
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};

const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}

Object.setPrototypeOf(obj, proto);

obj.foo() // "world"
```

上面代码中，`super.foo`指向原型对象`proto`的`foo`方法，但是绑定的`this`却还是当前对象`obj`，因此输出的就是`world`。
