# ECMAScript 简介

ECMAScript表示浏览器脚本语言的标准，简称ES，Javascript是实现该标准的一种浏览器脚本语言。


## 版本历史

| 发布实践   | 版本号 | 概述                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| 1997年6月  | 1.0    | 诞生。                                                       |
| 1998年6月  | 2.0    | 包含一些小的更改，用于同步独立的 ISO 国际标准。              |
| 1999年12月 | 3.0    | 奠定了JS的基本语法，被其后版本完全继承。                     |
| 2008年7月  | 4.0    | 当下ES6的前身，但由于这个版本太过激烈，对ES3做了彻底升级，所以暂时被"和谐"了。 |
| 2009年12月 | ES5    | 成为ISO国际标准。                                            |
| 2015年6月  | ES6    | 现代JavaScript的基石。                                       |
| 2016年6月  | ES7    | 指数运算符，`Array.prototype.includes()`。                   |
| 2017年6月  | ES8    | 异步增强。                                                   |
| 2018年6月  | ES9    | 异步迭代与正则改进。                                         |
| 2019年6月  | ES10   | 细节优化。                                                   |
| 2020年6月  | ES11   | 新数据类型BigInt与语法：可选链、空值合并。                   |
| 2021年6月  | ES12   | 实用新增。                                                   |
| 2022年6月  | ES13   | 类与私有化。                                                 |
| 2023年6月  | ES14   | 数组查找从末尾开始。                                         |
| 2024年6月  | ES15   |                                                              |



# ECMAScript 基础

## 变量

ES6 一共有 6 种声明变量的方法：

- var声明变量；
- function声明变量；
- let声明变量；
- const声明变量；
- import声明变量；
- class声明变量。



## 作用域

### 全局作用域

### 函数作用域

### 块级作用域

#### 为什么需要块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。第一种场景，内层变量可能会覆盖外层变量。

```javascript
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}

f(); // undefined
```

上面代码的原意是，`if`代码块的外部使用外层的`tmp`变量，内部使用内层的`tmp`变量。但是，函数`f`执行后，输出结果为`undefined`，原因在于变量提升，导致内层的`tmp`变量覆盖了外层的`tmp`变量。



第二种场景，用来计数的循环变量泄露为全局变量。

```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
```

上面代码中，变量`i`只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。



#### ES6 的块级作用域 

`let`实际上为 JavaScript 新增了块级作用域。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

上面的函数有两个代码块，都声明了变量`n`，运行后输出 5。这表示外层代码块不受内层代码块的影响。如果两次都使用`var`定义变量`n`，最后输出的值才是 10。



ES6 允许块级作用域的任意嵌套。

```javascript
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}};
```

上面代码使用了一个五层的块级作用域，每一层都是一个单独的作用域。第四层作用域无法读取第五层作用域的内部变量。



内层作用域可以定义外层作用域的同名变量。

```javascript
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
```



#### 块级作用域与函数声明

ES6 规定，块级作用域之中，函数声明语句的行为类似于`let`，在块级作用域之外不可引用。



```javascript
function f() { console.log('I am outside!'); }

(function () {
    if (false) {
        // 重复声明一次函数f
        function f() { console.log('I am inside!'); }
    }

    f();
}());
// Uncaught TypeError: f is not a function
```

上面的代码在 ES6 浏览器中会报错。因为如果改变了块级作用域内声明的函数的处理规则，会对老代码产生很大影响，为了减轻因此产生的不兼容问题，ES6 规定浏览器的实现可以不遵守上面的规定：

- 允许在块级作用域内声明函数。
- 函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。
- 同时，函数声明还会提升到所在的块级作用域的头部。



上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作`let`处理。根据这三条规则，浏览器的 ES6 环境中，块级作用域内声明的函数，行为类似于`var`声明的变量。上面的例子实际运行的代码如下。

```javascript
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```



考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

```javascript
// 块级作用域内部的函数声明语句，建议不要使用
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 块级作用域内部，优先使用函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```



另外，还有一个需要注意的地方。ES6 的块级作用域必须有大括号，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。

```javascript
// 第一种写法，报错
if (true) let x = 1;

// 第二种写法，不报错
if (true) {
  let x = 1;
}
```

上面代码中，第一种写法没有大括号，所以不存在块级作用域，而`let`只能出现在当前作用域的顶层，所以报错。第二种写法有大括号，所以块级作用域成立。



函数声明也是如此，严格模式下，函数只能声明在当前作用域的顶层。

```javascript
// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错
'use strict';
if (true)
  function f() {}
```







# ECMAScript 流程控制



## if-else 条件分支



## for 循环

for循环设置循环变量的部分是一个父作用域，循环体内部是一个单独的子作用域。

```ts
for (let i = 0; i < 3; i++) {
    let i = 'abc';
    console.log(i);
}
```

```
abc
abc
abc
```



上面代码正确运行，输出了3次abc 。这表明函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域。



## for...of 循环

ES6中，一个对象只要定义了@@iterator方法，就被视为具有iterator接口，就可以用for...of循环遍历它的值。也就是说，for...of循环内部调用是原对象的`Symbol.iterator`方法。



### 迭代数组

数组原生具备iterator接口。

```ts
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
    console.log(v); // red green blue
}
```



### 迭代Set

set也具备iterator接口，迭代set示例如下：

```ts
var engines = Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
    console.log(e);
}
```



### 迭代Map

map也具备iterator接口，迭代map示例如下：

```ts
var myMap = new Map();
myMap.set(0, "zero");
myMap.set(1, "one");
 
for (var [key, value] of myMap) {
  console.log(key + " = " + value);
}
for (var [key, value] of myMap.entries()) {
  console.log(key + " = " + value);
}

for (var key of myMap.keys()) {
  console.log(key);
}

for (var value of myMap.values()) {
  console.log(value);
}
```



### 迭代Generator函数

for...of循环可以自动遍历Generator函数，且此时不再需要调用next方法。

```ts
function *foo() {
    yield 1;
    yield 2;
    return 3;
}

for (let v of foo()) {
    console.log(v); // 1 2 3
}
```



下面是一个利用generator函数和for...of循环，实现斐波那契数列的例子。

```ts
function* fibonacci() {
    let [prev, curr] = [0, 1];
    for (;;) {
        [prev, curr] = [curr, prev + curr];
        yield curr;
    }
}

for (let n of fibonacci()) {
    if (n > 1000) break;
    console.log(n);
}
```



### 总结

对于普通的对象，for...of结构不能直接使用，会报错，必须部署了iterator接口后才能使用。但是，这样情况下，for...in循环依然可以用来遍历键名。

总结一下，for...of循环可以使用的范围包括数组、类似数组的对象（比如arguments对象、DOM NodeList对象）、Set和Map结构、后文的Generator对象，以及字符串。



## for...in循环

JavaScript原有的for...in循环，只能获得对象的键名，不能直接获取键值。for...in用来遍历对象中的属性：

```ts
let nicknames = ['di', 'boo', 'punkeye'];
nicknames.size = 3;
for (let nickname in nicknames) {
  console.log(nickname);
}
// 0, 1, 2, size
```



# ECMAScript Class类

## 概述

ES5通过构造函数，定义并生成新对象。下面是一个例子。

```ts
function Point(x,y){
    this.x = x;
    this.y = y;
}

Point.prototype.toString = function () {
    return '('+this.x+', '+this.y+')';
}
```



在ES6中，class作为对象的模板被引入，可以通过class关键字定义类。class的本质是function，它可以看作一个语法糖，让对象原型的写法更加清晰、更像面向对象编程的语法。



## 类声明

下面代码定义了一个“类”，可以看到里面有一个constructor函数，这就是构造函数，而this关键字则代表实例对象。这个类除了构造方法，还定义了一个toString方法。注意，定义方法的时候，前面不需要加上function这个保留字，直接把函数定义放进去了就可以了。

```ts
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

    toString() {
        return '('+this.x+', '+this.y+')';
    }

}

var point = new Point(2,3);
point.toString() // (2, 3)
```



类表达式可以为匿名或命名。注意：类不可重复声明。

```ts
// 一般声明类
class Example {
    constructor(a) {
        this.a = a;
    }
}
// 匿名声明类
let Example = class {
    constructor(a) {
        this.a = a;
    }
}
// 命名声明类
let Example = class Example {
    constructor(a) {
        this.a = a;
    }
}
```



类的声明不会提升（hoisting)，如果要使用某个Class，必须在使用之前定义它，否则会抛出一个ReferenceError的错误

```ts
new Example(); // Uncaught ReferenceError: Example is not defined
class Example {}
```



## 类的实例化

class的实例化必须通过new关键字。

```ts
class Example {}
let exam = Example(); // Uncaught TypeError: Class constructor Example cannot be invoked without 'new'
```



共享原型对象：

```ts
class Example {
    constructor(a, b) {
        this.a = a;
        this.b = b;
        console.log('Example');
    }
    sum() {
        return this.a + this.b;
    }
}
let exam1 = new Example(2, 1);
let exam2 = new Example(3, 1);
 
// __proto__ 已废弃，不建议使用
// console.log(exam1.__proto__ == exam2.__proto__); 
 
console.log(Object.getPrototypeOf(exam1) === Object.getPrototypeOf(exam2));// true
 
Object.getPrototypeOf(exam1).sub = function() {
    return this.a - this.b;
}
console.log(exam1.sub()); // 1
console.log(exam2.sub()); // 2
```



## 类的属性

### prototype

ES6 中，prototype仍旧存在，虽然可以直接自类中定义方法，但是其实方法还是定义在prototype上的。

```ts
Example.prototype={
    //methods
}
// 或使用Object.assign函数来添加方法
Object.assign(Example.prototype,{
    //methods
})
```



### 静态属性

class本身的属性，即直接定义在类内部的属性，不需要实例化。

```ts
class Example {
    static a = 2;
}
Example.b = 2;
```



### 公共属性

```ts
class Example{}
Example.prototype.a = 2;
```



### 实例属性

定义在实例对象this上的属性。

```ts
class Example {
    a = 2;
    constructor () {
        console.log(this.a);
    }
}
new Example()
```



### name属性

```ts
let Example=class Exam {
    constructor(a) {
        this.a = a;
    }
}
console.log(Example.name); // Exam
 
let Example=class {
    constructor(a) {
        this.a = a;
    }
}
console.log(Example.name); // Example
```



## 类的函数

### constructor

constructor方法是类的默认方法，创建类的实例化对象时被调用。

```ts
class Example{
    constructor(){
      console.log('我是constructor');
    }
}
new Example(); // 我是constructor
```



constructor方法默认返回对象this，也可指定返回对象。

```ts
class Test {
    constructor(){
        // 默认返回实例对象 this
    }
}
console.log(new Test() instanceof Test); // true
 
class Example {
    constructor(){
        // 指定返回对象
        return new Test();
    }
}
console.log(new Example() instanceof Example); // false
```



### getter和setter

通过set方法赋值时，不能直接通过属性名的方式赋值，会导致自身递归调用。必须通过`_属性名`的方式来给属性赋值。

```ts
class Example{
    constructor(a, b) {
        this.a = a; // 实例化时调用 set 方法
        this.b = b;
    }
    get a(){
        console.log('getter');
        return this.a;
    }
    set a(a){
        console.log('setter');
        this.a = a; // 自身递归调用
    }
}
let exam = new Example(1,2); // 不断输出 setter ，最终导致 RangeError
```



```ts
class Example1{
    constructor(a, b) {
        this.a = a;
        this.b = b;
    }
    get a(){
        console.log('getter');
        return this._a;
    }
    set a(a){
        console.log('setter');
        this._a = a;
    }
}
let exam1 = new Example1(1,2); // 只输出 setter , 不会调用 getter 方法
console.log(exam1._a); // 1, 可以直接访问
```



getter 与 setter 必须同级出现。

```ts
class Example {
    constructor(a) {
        this.a = a; 
    }
    get a() {
        return this.a;
    }
}
let exam = new Example(1); // Uncaught TypeError: Cannot set property // a of #<Example> which has only a getter
```

```ts
class Father {
    constructor(){}
    get a() {
        return this._a;
    }
}
class Child extends Father {
    constructor(){
        super();
    }
    set a(a) {
        this._a = a;
    }
}
let test = new Child();
test.a = 2;
console.log(test.a); // undefined

class Father1 {
    constructor(){}
    // 或者都放在子类中
    get a() {
        return this._a;
    }
    set a(a) {
        this._a = a;
    }
}
class Child1 extends Father1 {
    constructor(){
        super();
    }
}
let test1 = new Child1();
test1.a = 2;
console.log(test1.a); // 2
```



### 静态方法

```ts
class Example{
    static sum(a, b) {
        console.log(a+b);
    }
}
Example.sum(1, 2); // 3
```



### 原型方法

在类中定义函数不需要使用function关键词。

```ts
class Example {
    sum(a, b) {
        console.log(a + b);
    }
}
let exam = new Example();
exam.sum(1, 2); // 3
```



### 实例方法

```ts
class Example {
    constructor() {
        this.sum = (a, b) => {
            console.log(a + b);
        }
    }
}
```



## 类的封装和继承

通过extends实现类的继承，子类constructor方法中必须有super，且必须出现在this之前。super关键字调用的方法指代父类的同名方法。在constructor方法内，super指代父类的constructor方法；在toString方法内，super指代父类的toString方法。

```ts
class Father {
    constructor() {}
}
class Child extends Father {
    constructor() {}
    // or 
    // constructor(a) {
        // this.a = a;
        // super();
    // }
}
let test = new Child(); // Uncaught ReferenceError: Must call super 
// constructor in derived class before accessing 'this' or returning 
// from derived constructor
```



调用父类构造函数，只能出现在子类的构造函数内。

```ts
class Father {
    test(){
        return 0;
    }
    static test1(){
        return 1;
    }
}
class Child extends Father {
    constructor(){
        super();
    }
}
class Child1 extends Father {
    test2() {
        super(); // Uncaught SyntaxError: 'super' keyword unexpected     
        // here
    }
}
```



调用父类方法，super作为对象，在普通方法中，指向父类的原型对象，在静态方法中，指向父类。

```ts
class Child2 extends Father {
    constructor(){
        super();
        // 调用父类普通方法
        console.log(super.test()); // 0
    }
    static test3(){
        // 调用父类静态方法
        return super.test1();
    }
}
Child2.test3(); // 3
```



不可继承常规对象。

```ts
var Father = {
    // ...
}
class Child extends Father {
     // ...
}
// Uncaught TypeError: Class extends value #<Object> is not a constructor or null
 
// 解决方案
Object.setPrototypeOf(Child.prototype, Father);
```



# ECMAScript 模块

ES6引入了模块化，其设计思想是在编译时就能确定模块的依赖关系，以及输入和输出的变量。ES6 的模块化分为导出（export）与导入（import）两个模块。

ES6的模块自动开启严格模式，不管有没有在模块头部加上**use strict;**。模块中可以导入和导出各种类型的变量，如函数，对象，字符串，数字，布尔值，类等。每个模块都有自己的上下文，每一个模块内声明的变量都是局部变量，不会污染全局作用域。每一个模块只加载一次（是单例的），若再去加载同目录下同文件，会直接从内存中读取。



## export与import

模块导入导出各种类型的变量，如字符串，数值，方法，类。

- 导出的函数声明与类声明必须要有名称（export default 命令另外考虑）；
- 不仅能导出声明还能导出引用（例如函数）；
- export 命令可以出现在模块的任何位置，但必需处于模块顶层；
- import 命令会提升到整个模块的头部，首先执行。



```ts
/*-----export [test.js]-----*/
export let myName = "Tom"; // export单个变量
let myAge = 20;
let myfn = function(){
    return "My name is" + myName + "! I'm '" + myAge + "years old."
}
let myClass =  class myClass {
    static a = "yeah!";
}
export { myAge, myfn, myClass } // export多个变量
 
/*-----import [xxx.js]-----*/
import { myName, myAge, myfn, myClass } from "./test.js";
console.log(myfn());// My name is Tom! I'm 20 years old.
console.log(myAge);// 20
console.log(myName);// Tom
console.log(myClass.a );// yeah!
```



建议使用大括号指定所要输出的一组变量写在文档尾部，明确导出的接口。函数与类都需要有对应的名称，导出文档尾部也避免了无对应名称。



### 复合使用

export 与 import 可以在同一模块使用，使用特点：

- 可以将导出接口改名，包括 default。
- 复合使用 export 与 import ，也可以导出全部，当前模块导出的接口会覆盖继承导出的。



```ts
export { foo, bar } from "methods";
 
// 约等于下面两段语句，不过上面导入导出方式该模块没有导入 foo 与 bar
import { foo, bar } from "methods";
export { foo, bar };
 
/* ------- 特点 1 --------*/
// 普通改名
export { foo as bar } from "methods";
// 将 foo 转导成 default
export { foo as default } from "methods";
// 将 default 转导成 foo
export { default as foo } from "methods";
 
/* ------- 特点 2 --------*/
export * from "methods";
```



## as用法

export命令导出的接口名称，须和模块内部的变量有一一对应关系。导入的变量名须和导出的接口名称相同，即顺序可以不一致。使用as重新定义导入的接口名称，隐藏模块内部的变量：

```ts
/*-----export [test.js]-----*/
let myName = "Tom";
export { myName as exportName }
 
/*-----import [xxx.js]-----*/
import { exportName } from "./test.js";
console.log(exportName);// Tom
```



不同模块导出接口名称命名重复，使用as重新定义变量名。

```ts
/*-----export [test1.js]-----*/
let myName = "Tom";
export { myName }
/*-----export [test2.js]-----*/
let myName = "Jerry";
export { myName }
/*-----import [xxx.js]-----*/
import { myName as name1 } from "./test1.js";
import { myName as name2 } from "./test2.js";
console.log(name1);// Tom
console.log(name2);// Jerry
```



## import特性

**只读属性**：不允许在加载模块的脚本里面，改写接口的引用指向，即可以改写 import 变量类型为对象的属性值，不能改写 import 变量类型为基本类型的值。

```ts
import {a} from "./xxx.js"
a = {}; // error
 
import {a} from "./xxx.js"
a.foo = "hello"; // a = { foo : 'hello' }
```



**单例模式**：多次重复执行同一句 import 语句，那么只会执行一次，而不会执行多次。import 同一模块，声明不同接口引用，会声明对应变量，但只执行一次 import 。

```ts
import { a } "./xxx.js";
import { a } "./xxx.js";
// 相当于 import { a } "./xxx.js";
 
import { a } from "./xxx.js";
import { b } from "./xxx.js";
// 相当于 import { a, b } from "./xxx.js";
```



**静态执行特性**：import 是静态执行，所以不能使用表达式和变量。

```ts
import { "f" + "oo" } from "methods";
// error
let module = "methods";
import { foo } from module;
// error
if (true) {
  import { foo } from "method1";
} else {
  import { foo } from "method2";
}
// error
```



整体引入：除了上述的逐一引入方式，另一种写法是整体输入。

```ts
// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

// main.js
import * as circle from 'circle';

console.log("圆面积：" + circle.area(4));
console.log("圆周长：" + circle.circumference(14));
```



module命令可以取代import语句，达到整体输入模块的作用。module命令后面跟一个变量，表示输入的模块定义在该变量上。

```ts
// main.js
module circle from 'circle';

console.log("圆面积：" + circle.area(4));
console.log("圆周长：" + circle.circumference(14));
```



## export default

export default命令用于指定模块的默认输出。如果模块加载时，只能输出一个值或方法，那就是export default所指定的那个值或方法。所以，import命令后面才不用加大括号。显然，一个模块只能有一个默认输出，因此export deault命令只能使用一次。

export default功能具备如下特性：

- export default可以输出函数、变量、类和默认值等；
- 在一个文件或模块中，export、import 可以有多个，export default 仅有一个；
- export default 中的 default 是对应的导出接口变量；
- 通过 export 方式导出，在导入时要加{ }，export default 则不需要；
- export default 向外暴露的成员，可以使用任意变量来接收。

```ts
var a = "My name is Tom!";
export default a; // 仅有一个
export default var c = "error"; 
// error，default 已经是对应的导出变量，不能跟着变量声明语句
 
import b from "./xxx.js"; // 不需要加{}，使用任意变量接收，指向export default变量
```



如果想在一条import语句中，同时输入默认方法和其他变量，可以写成下面这样。

```ts
import customName, { otherMethod } from './export-default';
```



## 模块的继承

模块之间也可以继承。假设有一个circleplus模块，继承了circle模块。下面代码中的“export *”，表示输出circle模块的所有属性和方法，export default命令定义模块的默认方法。

```javascript
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}
```



这时，也可以将circle的属性或方法，改名后再输出。下面代码表示，只输出circle模块的area方法，且将其改名为circleArea。

```javascript
// circleplus.js
export { area as circleArea } from 'circle';
```



加载上面模块的写法如下。下面代码中的"import exp"表示，将circleplus模块的默认方法加载为exp方法。

```javascript
// main.js
module math from "circleplus";
import exp from "circleplus";
console.log(exp(math.pi));
```



