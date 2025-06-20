---
layout: post
title: 第002章-ECMAScript 数据类型
categories: [ECMAScript]
description: ECMAScript定义了 ​​7 种原始数据类型（Primitive Types）​​ 和 ​​1 种引用数据类型（Object Type）​​。
keywords: ECMAScript 数据类型.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ECMAScript 数据类型

## 基础类型



## String类型

### 概述

JavaScript内部，字符以UTF-16的格式储存，每个字符固定为2个字节。对于那些需要4个字节储存的字符（Unicode码点大于0xFFFF的字符），JavaScript会认为它们是两个字符。



### 字符的Unicode表示

JavaScript允许采用“\uxxxx”形式表示一个字符，其中“xxxx”表示字符的码点。

```ts
"\u0061" // "a"
```



这种表示法只限于 `\u0000——\uFFFF` 之间的字符。超出这个范围的字符，必须用两个双字节的形式表达。

```ts
"\uD842\uDFB7" // "𠮷"
```



如果直接在“\u”后面跟上超过0xFFFF的数值（比如\u20BB7），JavaScript会理解成“\u20BB+7”。由于\u20BB是一个不可打印字符，所以只会显示一个空格，后面跟着一个7。

```ts
"\u20BB7" // " 7"
```



ES6对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。

```ts
"\u{20BB7}" // "𠮷"
"\u{41}\u{42}\u{43}" // "ABC"
```



### 模板字符串

模板字符串相当于加强版的字符串，用反引号 **`**,除了作为普通字符串，还可以用来定义多行字符串，还可以在字符串中加入变量和表达式。模板字符串中的换行和空格都是会被保留的



#### 基础用法

普通字符串。

```ts
let string = `Hello'\n'world`;
console.log(string); 
```

```
"Hello'
'world"
```



多行字符串。

```ts
let string1 =  `Hey,
can you stop angry now?`;
console.log(string1);
```

```
Hey,
can you stop angry now?
```



如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。

```ts
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`);
```

上面代码中，所有模板字符串的空格和换行，都是被保留的，比如`<ul>`标签前面会有一个换行。如果你不想要这个换行，可以使用`trim`方法消除它。

```ts
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```



字符串插入变量和表达式，变量名写在`${}`中，`${}`中可以放入JavaScript表达式。如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的`toString`方法。如果模板字符串中的变量没有声明，将报错。

```ts
let name = "Mike";
let age = 27;
let info = `My Name is ${name},I am ${age+1} years old next year.`
console.log(info);
```

```
My Name is Mike,I am 28 years old next year.
```



字符串中调用函数。

```ts
function f(){
  return "have fun!";
}
let string2= `Game start,${f()}`;
console.log(string2); 
```

```
Game start,have fun!
```



#### 嵌套模板字符串

模板字符串甚至还能嵌套。

```javascript
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;
```



上面代码中，模板字符串的变量之中，又嵌入了另一个模板字符串，使用方法如下。

```javascript
const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];

console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
```



如果需要引用模板字符串本身，在需要时执行，可以写成函数。

```javascript
let func = (name) => `Hello ${name}!`;
func('Jack') // "Hello Jack!"
```

上面代码中，模板字符串写成了一个函数的返回值。执行这个函数，就相当于执行这个模板字符串了。



#### 标签模板

标签模板是指模板字符串紧跟在函数后面的使用方式，是一种函数调用，模板字符串作为函数参数。

```ts
alert`Hello world!`;
// 等价于
alert('Hello world!');
```



模板字符串里有变量时，会将模板字符串先处理成多个参数，再调用函数。

```ts
let a = 5;
let b = 10;
function tag(stringArr, ...values){
    // ...
}

tag`Hello ${ a + b } world ${ a * b }`;
// 等同于，tag是一个函数
tag(['Hello ', ' world ', ''], 15, 50);
```

`tag`函数的第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，其他参数都是模板字符串各个变量被替换后的值。`tag`函数所有参数的实际值如下：

- 第一个参数：`['Hello ', ' world ', '']`
- 第二个参数：15
- 第三个参数：50



下面是一个更复杂的例子。

```ts
let total = 30;
let msg = passthru`The total is ${total} (${total*1.05} with tax)`;

function passthru(literals) {
    let result = '';
    let i = 0;

    while (i < literals.length) {
        result += literals[i++];
        if (i < arguments.length) {
            result += arguments[i];
        }
    }

    return result;
}

```

上面这个例子展示了，如何将各个参数按照原来的位置拼合回去。`passthru`函数采用 rest 参数的写法如下：

```ts
function passthru(literals, ...values) {
    let output = "";
    let index;
    for (index = 0; index < values.length; index++) {
        output += literals[index] + values[index];
    }

    output += literals[index]
    return output;
}
```



标签模板的一个重要应用，就是过滤 HTML 字符串，防止用户输入恶意内容。

```ts
let message =SaferHTML`<p>${sender} has sent you a message.</p>`;
function SaferHTML(templateData) {
    let s = templateData[0];
    for (let i = 1; i < arguments.length; i++) {
        let arg = String(arguments[i]);

        // Escape special characters in the substitution.
        s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

        // Don't escape special characters in the template.
        s += templateData[i];
    }
    return s;
}
```

上面代码中，`sender`变量往往是用户提供的，经过`SaferHTML`函数处理，里面的特殊字符都会被转义。

```ts
let sender = '<script>alert("abc")</script>'; // 恶意代码
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;
```



## 数值类型

### 二进制和八进制表示法

ES6提供了二进制和八进制数值的新的写法：

- 二进制写法：前缀`0b`或`0B`； 
- 八进制写法：前缀`0o`或`0O`；

```ts
0b111110111 === 503 // true
0o767 === 503 // true
```



如果要将`0b`和`0o`前缀的字符串数值转为十进制，要使用`Number`方法。

```ts
Number('0b111')  // 7
Number('0o10')  // 8
```



### 数值分隔符

ES2021，允许 JavaScript 的数值使用下划线（`_`）作为分隔符。

```javascript
let budget = 1_000_000_000_000;
budget === 10 ** 12 // true
```



这个数值分隔符没有指定间隔的位数，也就是说，可以每三位添加一个分隔符，也可以每一位、每两位、每四位添加一个。

```javascript
123_00 === 12_300 // true
12345_00 === 123_4500 // true
12345_00 === 1_234_500 // true
```



小数和科学计数法也可以使用数值分隔符。

```javascript
0.000_001 // 小数
1e10_000 // 科学计数法
```



数值分隔符有几个使用注意点。

- 不能放在数值的最前面或最后面；
- 不能两个或两个以上的分隔符连在一起；
- 小数点的前后不能有分隔符；
- 科学计数法里面，表示指数的`e`或`E`前后不能有分隔符。



下面的写法都会报错。

```javascript
3_.141
3._141
1_e12
1e_12
123__456
_1464301
1464301_
```



除了十进制，其他进制的数值也可以使用分隔符。

```javascript
0b1010_0001_1000_0101 // 二进制
0xA0_B0_C0 // 十六进制
```

可以看到，数值分隔符可以按字节顺序分隔数值，这在操作二进制位时，非常有用。



注意，分隔符不能紧跟着进制的前缀`0b`、`0B`、`0o`、`0O`、`0x`、`0X`。

```javascript
// 报错
0_b111111000
0b_111111000
```



数值分隔符只是一种书写便利，对于 JavaScript 内部数值的存储和输出，并没有影响。

```javascript
let num = 12_345;

num // 12345
num.toString() // 12345
```

上面示例中，变量`num`的值为`12_345`，但是内部存储和输出的时候，都不会有数值分隔符。



下面三个将字符串转成数值的函数，不支持数值分隔符。主要原因是语言的设计者认为，数值分隔符主要是为了编码时书写数值的方便，而不是为了处理外部输入的数据。

- Number()
- parseInt()
- parseFloat()

```javascript
Number('123_456') // NaN
parseInt('123_456') // 123
```



## 对象类型

### 对象字面量

ES6允许直接写入变量和函数，作为对象的属性和方法。

```ts
let birth = "1994-09-01"
var person = {
    name: '张三',
    //等同于birth: birth
    birth,
    // 等同于hello: function ()
    hello() { console.log('我的名字是', this.name); },
    sayHi: function(){
        console.log("Hi");
    }
};
person.sayHi()
```



如果是Generator函数，则要在前面加一个星号：

```ts
const obj = {
    * myGenerator() {
        yield 'hello world';
    }
};
//等同于
const obj = {
    myGenerator: function* () {
        yield 'hello world';
    }
};
```



ES6 允许在大括号里面，直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

```javascript
const foo = 'bar';
const baz = {foo};
baz // {foo: "bar"}

// 等同于
const baz = {foo: foo};
```

上面代码中，变量`foo`直接写在大括号里面。这时，属性名就是变量名, 属性值就是变量值。下面是另一个例子。

```javascript
function f(x, y) {
  return {x, y};
}

// 等同于

function f(x, y) {
  return {x: x, y: y};
}

f(1, 2) // Object {x: 1, y: 2}
```

除了属性简写，方法也可以简写。

```javascript
const o = {
  method() {
    return "Hello!";
  }
};

// 等同于

const o = {
  method: function() {
    return "Hello!";
  }
};
```

下面是一个实际的例子。

```javascript
let birth = '2000/01/01';

const Person = {

  name: '张三',

  //等同于birth: birth
  birth,

  // 等同于hello: function ()...
  hello() { console.log('我的名字是', this.name); }

};
```

这种写法用于函数的返回值，将会非常方便。

```javascript
function getPoint() {
  const x = 1;
  const y = 10;
  return {x, y};
}

getPoint()
// {x:1, y:10}
```

CommonJS 模块输出一组变量，就非常合适使用简洁写法。

```javascript
let ms = {};

function getItem (key) {
  return key in ms ? ms[key] : null;
}

function setItem (key, value) {
  ms[key] = value;
}

function clear () {
  ms = {};
}

module.exports = { getItem, setItem, clear };
// 等同于
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```

属性的赋值器（setter）和取值器（getter），事实上也是采用这种写法。

```javascript
const cart = {
  _wheels: 4,

  get wheels () {
    return this._wheels;
  },

  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('数值太小了！');
    }
    this._wheels = value;
  }
}
```

简洁写法在打印对象时也很有用。

```javascript
let user = {
  name: 'test'
};

let foo = {
  bar: 'baz'
};

console.log(user, foo)
// {name: "test"} {bar: "baz"}
console.log({user, foo})
// {user: {name: "test"}, foo: {bar: "baz"}}
```

上面代码中，`console.log`直接输出`user`和`foo`两个对象时，就是两组键值对，可能会混淆。把它们放在大括号里面输出，就变成了对象的简洁表示法，每组键值对前面会打印对象名，这样就比较清晰了。

注意，简写的对象方法不能用作构造函数，会报错。

```javascript
const obj = {
  f() {
    this.foo = 'bar';
  }
};

new obj.f() // 报错
```

上面代码中，`f`是一个简写的对象方法，所以`obj.f`不能当作构造函数使用。



### 属性名表达式

ES6允许用表达式作为属性名或方法名，但是一定要将表达式放在方括号内。

```ts
let propKey = "name"
const obj = {
    [propKey]: "bighead", // 使用变量作为属性名
    ["he"+"llo"](){ // 使用表达式作为方法名
        return "Hi";
    }
}
obj.hello();  //"Hi"
obj.world = "world"; 
obj['a'+'bc'] = 123;
console.log(obj);
```



注意点：属性的简洁表示法和属性名表达式不能同时使用，否则会报错。

```ts
const hello = "Hello";
const obj = {
    [hello]
};
console.log(obj);
obj  //SyntaxError: Unexpected token }

const hello = "Hello";
const obj = {
    [hello+"2"]:"world"
};
obj  //{Hello2: "world"}
```



JavaScript 定义对象的属性，有两种方法。

```javascript
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```

上面代码的方法一是直接用标识符作为属性名，方法二是用表达式作为属性名，这时要将表达式放在方括号之内。

但是，如果使用字面量方式定义对象（使用大括号），在 ES5 中只能使用方法一（标识符）定义属性。

```javascript
var obj = {
  foo: true,
  abc: 123
};
```

ES6 允许字面量定义对象时，用方法二（表达式）作为对象的属性名，即把表达式放在方括号内。

```javascript
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};
```

下面是另一个例子。

```javascript
let lastWord = 'last word';

const a = {
  'first word': 'hello',
  [lastWord]: 'world'
};

a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"
```

表达式还可以用于定义方法名。

```javascript
let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};

obj.hello() // hi
```

注意，属性名表达式与简洁表示法，不能同时使用，会报错。

```javascript
// 报错
const foo = 'bar';
const bar = 'abc';
const baz = { [foo] };

// 正确
const foo = 'bar';
const baz = { [foo]: 'abc'};
```

注意，属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串`[object Object]`，这一点要特别小心。

```javascript
const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
```

上面代码中，`[keyA]`和`[keyB]`得到的都是`[object Object]`，所以`[keyB]`会把`[keyA]`覆盖掉，而`myObject`最后只有一个`[object Object]`属性。



### 方法的name属性

函数的`name`属性，返回函数名。对象方法也是函数，因此也有`name`属性。

```javascript
const person = {
  sayName() {
    console.log('hello!');
  },
};

person.sayName.name   // "sayName"
```

上面代码中，方法的`name`属性返回函数名（即方法名）。

如果对象的方法使用了取值函数（`getter`）和存值函数（`setter`），则`name`属性不是在该方法上面，而是该方法的属性的描述对象的`get`和`set`属性上面，返回值是方法名前加上`get`和`set`。

```javascript
const obj = {
  get foo() {},
  set foo(x) {}
};

obj.foo.name
// TypeError: Cannot read property 'name' of undefined

const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');

descriptor.get.name // "get foo"
descriptor.set.name // "set foo"
```

有两种特殊情况：`bind`方法创造的函数，`name`属性返回`bound`加上原函数的名字；`Function`构造函数创造的函数，`name`属性返回`anonymous`。

```javascript
(new Function()).name // "anonymous"

var doSomething = function() {
  // ...
};
doSomething.bind().name // "bound doSomething"
```

如果对象的方法是一个 Symbol 值，那么`name`属性返回的是这个 Symbol 值的描述。

```javascript
const key1 = Symbol('description');
const key2 = Symbol();
let obj = {
  [key1]() {},
  [key2]() {},
};
obj[key1].name // "[description]"
obj[key2].name // ""
```

上面代码中，`key1`对应的 Symbol 值有描述，`key2`没有。



### 属性的可枚举性和遍历

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。`Object.getOwnPropertyDescriptor`方法可以获取该属性的描述对象。

```javascript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```

描述对象的`enumerable`属性，称为“可枚举性”，如果该属性为`false`，就表示某些操作会忽略当前属性。

目前，有四个操作会忽略`enumerable`为`false`的属性。

- `for...in`循环：只遍历对象自身的和继承的可枚举的属性。
- `Object.keys()`：返回对象自身的所有可枚举的属性的键名。
- `JSON.stringify()`：只串行化对象自身的可枚举的属性。
- `Object.assign()`： 忽略`enumerable`为`false`的属性，只拷贝对象自身的可枚举的属性。

这四个操作之中，前三个是 ES5 就有的，最后一个`Object.assign()`是 ES6 新增的。其中，只有`for...in`会返回继承的属性，其他三个方法都会忽略继承的属性，只处理对象自身的属性。实际上，引入“可枚举”（`enumerable`）这个概念的最初目的，就是让某些属性可以规避掉`for...in`操作，不然所有内部属性和方法都会被遍历到。比如，对象原型的`toString`方法，以及数组的`length`属性，就通过“可枚举性”，从而避免被`for...in`遍历到。

```javascript
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable
// false

Object.getOwnPropertyDescriptor([], 'length').enumerable
// false
```

上面代码中，`toString`和`length`属性的`enumerable`都是`false`，因此`for...in`不会遍历到这两个继承自原型的属性。

另外，ES6 规定，所有 Class 的原型的方法都是不可枚举的。

```javascript
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype, 'foo').enumerable
// false
```

总的来说，操作中引入继承的属性会让问题复杂化，大多数时候，我们只关心对象自身的属性。所以，尽量不要用`for...in`循环，而用`Object.keys()`代替。

### 属性的遍历

ES6 一共有 5 种方法可以遍历对象的属性。

- `for...in`循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。
- `Object.keys`返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。
- `Object.getOwnPropertyNames`返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
- `Object.getOwnPropertySymbols`返回一个数组，包含对象自身的所有 Symbol 属性的键名。
- `Reflect.ownKeys`返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。



以上的 5 种方法遍历对象的键名，都遵守同样的属性遍历的次序规则。

- 首先遍历所有数值键，按照数值升序排列。
- 其次遍历所有字符串键，按照加入时间升序排列。
- 最后遍历所有 Symbol 键，按照加入时间升序排列。

```javascript
Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
// ['2', '10', 'b', 'a', Symbol()]
```

上面代码中，`Reflect.ownKeys`方法返回一个数组，包含了参数对象的所有属性。这个数组的属性次序是这样的，首先是数值属性`2`和`10`，其次是字符串属性`b`和`a`，最后是 Symbol 属性。



### 类数组对象

一个类数组对象（array-like object）必须含有 length 属性，且元素属性名必须是数值或者可转换为数值的字符。类数组对象可以由Array.from()函数转换为数组。

```ts
let arrayLike = {
  0: '1',
  1: '2',
  2: 3,
  length: 3
}
```



## 数组类型

### 数组声明

```ts
var mySet = new Set(["value1", "value2", "value3"]);
var myArray = [...mySet]; // 用...操作符，将 Set 转 Array

let map = new Map([[1, 'one'], [2, 'two'], [3, 'three']]);
[...map.keys()] // [1, 2, 3]
[...map.values()] // ['one', 'two', 'three']
[...map.entries()] // [[1,'one'], [2, 'two'], [3, 'three']]
[...map] // [[1,'one'], [2, 'two'], [3, 'three']]
```



### 数组的空位

数组的空位指的是，数组的某一个位置没有任何值，比如`Array()`构造函数返回的数组都是空位。

```javascript
Array(3) // [, , ,]
```

上面代码中，`Array(3)`返回一个具有 3 个空位的数组。

注意，空位不是`undefined`，某一个位置的值等于`undefined`，依然是有值的。空位是没有任何值，`in`运算符可以说明这一点。

```javascript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```

上面代码说明，第一个数组的 0 号位置是有值的，第二个数组的 0 号位置没有值。

ES5 对空位的处理，已经很不一致了，大多数情况下会忽略空位。

- `forEach()`, `filter()`, `reduce()`, `every()` 和`some()`都会跳过空位。
- `map()`会跳过空位，但会保留这个值
- `join()`和`toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串。

```javascript
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```

ES6 则是明确将空位转为`undefined`。

`Array.from()`方法会将数组的空位，转为`undefined`，也就是说，这个方法不会忽略空位。

```javascript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]
```

扩展运算符（`...`）也会将空位转为`undefined`。

```javascript
[...['a',,'b']]
// [ "a", undefined, "b" ]
```

`copyWithin()`会连空位一起拷贝。

```javascript
[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]
```

`fill()`会将空位视为正常的数组位置。

```javascript
new Array(3).fill('a') // ["a","a","a"]
```

`for...of`循环也会遍历空位。

```javascript
let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
```

上面代码中，数组`arr`有两个空位，`for...of`并没有忽略它们。如果改成`map()`方法遍历，空位是会跳过的。

`entries()`、`keys()`、`values()`、`find()`和`findIndex()`会将空位处理成`undefined`。

```javascript
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

由于空位的处理规则非常不统一，所以建议避免出现空位。



## 参考资料

- [字符串的扩展 | ECMAScript 6入门](https://wohugb.gitbooks.io/ecmascript-6/content/docs/string.html)

  
