---
layout: post
title: 2025-05-13-第0010章-ECMAScript Array
categories: [ECMAScript]
description: 
keywords: ECMAScript Array.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ECMAScript Array

## 功能特性

## 源码解析

### 方法定义

> @since ES5

```ts
// lib.es5.d.ts
interface Array<T> {

    length: number;

    toString(): string;

    toLocaleString(): string;

    pop(): T | undefined;

    push(...items: T[]): number;

    concat(...items: ConcatArray<T>[]): T[];

    concat(...items: (T | ConcatArray<T>)[]): T[];

    join(separator?: string): string;

    reverse(): T[];

    shift(): T | undefined;

    slice(start?: number, end?: number): T[];

    sort(compareFn?: (a: T, b: T) => number): this;

    splice(start: number, deleteCount?: number): T[];

    splice(start: number, deleteCount: number, ...items: T[]): T[];

    unshift(...items: T[]): number;

    indexOf(searchElement: T, fromIndex?: number): number;

    lastIndexOf(searchElement: T, fromIndex?: number): number;

    every<S extends T>(predicate: (value: T, index: number, array: T[]) => value is S, thisArg?: any): this is S[];

    every(predicate: (value: T, index: number, array: T[]) => unknown, thisArg?: any): boolean;

    some(predicate: (value: T, index: number, array: T[]) => unknown, thisArg?: any): boolean;

    forEach(callbackfn: (value: T, index: number, array: T[]) => void, thisArg?: any): void;

    map<U>(callbackfn: (value: T, index: number, array: T[]) => U, thisArg?: any): U[];

    filter<S extends T>(predicate: (value: T, index: number, array: T[]) => value is S, thisArg?: any): S[];

    filter(predicate: (value: T, index: number, array: T[]) => unknown, thisArg?: any): T[];

    reduce(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: T[]) => T): T;
    reduce(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: T[]) => T, initialValue: T): T;

    reduce<U>(callbackfn: (previousValue: U, currentValue: T, currentIndex: number, array: T[]) => U, initialValue: U): U;

    reduceRight(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: T[]) => T): T;
    reduceRight(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: T[]) => T, initialValue: T): T;

    reduceRight<U>(callbackfn: (previousValue: U, currentValue: T, currentIndex: number, array: T[]) => U, initialValue: U): U;

    [n: number]: T;
}
```



> @since ES6

```ts
// lib.es2015.core.d.ts
interface Array<T> {
    /**
     * 查找数组中符合条件的元素，若有多个符合条件的元素，则返回第一个元素
     *
     * @param predicate 回调函数
     * @param thisArg 回调函数中的this值
     */
    find<S extends T>(predicate: (value: T, index: number, obj: T[]) => value is S, thisArg?: any): S | undefined;
    find(predicate: (value: T, index: number, obj: T[]) => unknown, thisArg?: any): T | undefined;
    
    /**
     * 查找数组中符合条件的元素索引，若有多个符合条件的元素，则返回第一个元素索引
     * 
     * @param predicate 回调函数
     * @param thisArg 可选，回调函数中的this值
     */
    findIndex(predicate: (value: T, index: number, obj: T[]) => unknown, thisArg?: any): number;
    
    /**
     * 将一定范围索引的数组元素内容填充为单个指定的值
     * 
     * @param valuey 用来填充的值
     * @param start 可选，被填充的起始索引
     * @param end 可选，被填充的结束索引，默认为数组末尾
     */
    fill(value: T, start?: number, end?: number): this;
    /**
     * 将一定范围索引的数组元素修改为此数组另一指定范围索引的元素
     * 
     * @param target 被修改的起始索引，为负数时表示倒数
     * @param start 被用来覆盖的数据的起始索引
     * @param end 可选，被用来覆盖的数据的结束索引，默认为数组末尾
     */
    copyWithin(target: number, start: number, end?: number): this;

    toLocaleString(locales: string | string[], options?: Intl.NumberFormatOptions & Intl.DateTimeFormatOptions): string;
}
```

```ts
// lib.es2015.iterable.d.ts
interface Array<T> {
    [Symbol.iterator](): ArrayIterator<T>;
    /**
     * 遍历键值对
     */
    entries(): ArrayIterator<[number, T]>;

    /**
     * 遍历键名
     */
    keys(): ArrayIterator<number>;
    
    /**
     * 遍历键值
     */
    values(): ArrayIterator<T>;
}
```



> @since ES7

```ts
// lib.es2016.array.include.d.ts
interface Array<T> {
    /**
     * 数组是否包含指定值
     *
     * @param 包含的指定值
     * @param 可选，搜索的起始索引，默认为0
     */
    includes(searchElement: T, fromIndex?: number): boolean;
}
```



> @since ES10

```ts
// lib.es2019.array.d.ts
interface Array<T> {
	/**
	 * 对数组中的每个元素应用给定的回调函数，然后将结果展开一级，返回一个新数组
	 *
	 * @param callback 遍历函数，该遍历函数可接受3个参数：当前元素、当前元素索引、原数组
	 * @param thisArg 可选，遍历函数中this的指向
	 */
    flatMap<U, This = undefined>(
        callback: (this: This, value: T, index: number, array: T[]) => U | ReadonlyArray<U>,
        thisArg?: This,
    ): U[];
		
	/**
	 * 创建一个新的数组，并根据指定深度递归地将所有子数组元素拼接到新的数组中。
	 *
	 * @param depth 
	 */
    flat<A, D extends number = 1>(
        this: A,
        depth?: D,
    ): FlatArray<A, D>[];
}
```



> @since ES6

```ts
// lib.es2015.core.d.ts
interface ArrayConstructor {
    /**
     * 将类数组对象或可迭代对象转化为数组
     *
     * @param arrayLike
     */
    from<T>(arrayLike: ArrayLike<T>): T[];
    
    // 将类数组对象或可迭代对象的属性或元素转换后转化为数组，thisArg为mapfn函数中的this对象
    from<T, U>(arrayLike: ArrayLike<T>, mapfn: (v: T, k: number) => U, thisArg?: any): U[];
    
    // 将参数中所有值作为元素形成数组
    of<T>(...items: T[]): T[];
}
```



## 实践应用

### 静态方法

#### from

`Array.from()`方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。

下面是一个类似数组的对象，`Array.from()`将它转为真正的数组。

```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5 的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6 的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

实际应用中，常见的类似数组的对象是 DOM 操作返回的 NodeList 集合，以及函数内部的`arguments`对象。`Array.from()`都可以将它们转为真正的数组。

```javascript
// NodeList 对象
let ps = document.querySelectorAll('p');
Array.from(ps).filter(p => {
  return p.textContent.length > 100;
});

// arguments 对象
function foo() {
  var args = Array.from(arguments);
  // ...
}
```

上面代码中，`querySelectorAll()`方法返回的是一个类似数组的对象，可以将这个对象转为真正的数组，再使用`filter()`方法。

只要是部署了 Iterator 接口的数据结构，`Array.from()`都能将其转为数组。

```javascript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']
```

上面代码中，字符串和 Set 结构都具有 Iterator 接口，因此可以被`Array.from()`转为真正的数组。

如果参数是一个真正的数组，`Array.from()`会返回一个一模一样的新数组。

```javascript
Array.from([1, 2, 3])
// [1, 2, 3]
```

值得提醒的是，扩展运算符（`...`）也可以将某些数据结构转为数组。

```javascript
// arguments对象
function foo() {
  const args = [...arguments];
}

// NodeList对象
[...document.querySelectorAll('div')]
```

扩展运算符背后调用的是遍历器接口（`Symbol.iterator`），如果一个对象没有部署这个接口，就无法转换。`Array.from()`方法还支持类似数组的对象。所谓类似数组的对象，本质特征只有一点，即必须有`length`属性。因此，任何有`length`属性的对象，都可以通过`Array.from()`方法转为数组，而此时扩展运算符就无法转换。

```javascript
Array.from({ length: 3 });
// [ undefined, undefined, undefined ]
```

上面代码中，`Array.from()`返回了一个具有三个成员的数组，每个位置的值都是`undefined`。扩展运算符转换不了这个对象。

对于还没有部署该方法的浏览器，可以用`Array.prototype.slice()`方法替代。

```javascript
const toArray = (() =>
  Array.from ? Array.from : obj => [].slice.call(obj)
)();
```

`Array.from()`还可以接受一个函数作为第二个参数，作用类似于数组的`map()`方法，用来对每个元素进行处理，将处理后的值放入返回的数组。

```javascript
Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
```

下面的例子是取出一组 DOM 节点的文本内容。

```javascript
let spans = document.querySelectorAll('span.name');

// map()
let names1 = Array.prototype.map.call(spans, s => s.textContent);

// Array.from()
let names2 = Array.from(spans, s => s.textContent)
```

下面的例子将数组中布尔值为`false`的成员转为`0`。

```javascript
Array.from([1, , 2, , 3], (n) => n || 0)
// [1, 0, 2, 0, 3]
```

另一个例子是返回各种数据的类型。

```javascript
function typesOf () {
  return Array.from(arguments, value => typeof value)
}
typesOf(null, [], NaN)
// ['object', 'object', 'number']
```

如果`map()`函数里面用到了`this`关键字，还可以传入`Array.from()`的第三个参数，用来绑定`this`。

`Array.from()`可以将各种值转为真正的数组，并且还提供`map`功能。这实际上意味着，只要有一个原始的数据结构，你就可以先对它的值进行处理，然后转成规范的数组结构，进而就可以使用数量众多的数组方法。

```javascript
Array.from({ length: 2 }, () => 'jack')
// ['jack', 'jack']
```

上面代码中，`Array.from()`的第一个参数指定了第二个参数运行的次数。这种特性可以让该方法的用法变得非常灵活。

`Array.from()`的另一个应用是，将字符串转为数组，然后返回字符串的长度。因为它能正确处理各种 Unicode 字符，可以避免 JavaScript 将大于`\uFFFF`的 Unicode 字符，算作两个字符的 bug。

```javascript
function countSymbols(string) {
  return Array.from(string).length;
}
```



将类数组对象或可迭代对象转化为数组，其中包括ES6新增的Set和Map结构。

```ts
// 参数为数组,返回与原数组一样的数组
console.log(Array.from([1, 2])); // [1, 2]
// 参数含空位
console.log(Array.from([1, , 3])); // [1, undefined, 3]
// 可选map函数参数，用于对每个元素进行处理，放入数组的是处理后的元素
console.log(Array.from([1, 2, 3], (n) => n * 2)); // [2, 4, 6]
// 可选thisArg参数，用于指定map函数执行时的this对象
let mapObject = {
    do: function(n) {
        return n * 2;
    }
}
let arrayLike = [1, 2, 3];
console.log(Array.from(arrayLike, function (n){
    return this.do(n);
}, mapObject)); // [2, 4, 6]
```



将类数组对象转换为数组。一个类数组对象必须含有 length 属性，且元素属性名必须是数值或者可转换为数值的字符。

```ts
let arr = Array.from({
  0: '1',
  1: '2',
  2: 3,
  length: 3
});
console.log(arr); // ['1', '2', 3]
 
// 没有 length 属性,则返回空数组
let array = Array.from({
  0: '1',
  1: '2',
  2: 3,
});
console.log(array); // []
 
// 元素属性名不为数值且无法转换为数值，返回长度为length元素值为undefined的数组  
let array1 = Array.from({
  a: 1,
  b: 2,
  length: 2
});
console.log(array1); // [undefined, undefined]
```



将可迭代对象转换为数组。

```ts
// 转换map
let map = new Map();
map.set('key0', 'value0');
map.set('key1', 'value1');
console.log(Array.from(map)); // [['key0', 'value0'],['key1', 'value1']]

// 转换set
let arr = [1, 2, 3];
let set = new Set(arr);
console.log(Array.from(set)); // [1, 2, 3]

// 转换string，通过转换string可以避免JavaScript将大于\uFFFF的Unicode字符，算作两个字符的bug。
let str = 'abc';
console.log(Array.from(str)); // ["a", "b", "c"]
```



#### of

`Array.of()`方法用于将一组值，转换为数组。

```ts
console.log(Array.of(1, 2, 3, 4)); // [1, 2, 3, 4]
// 参数值可为不同类型
console.log(Array.of(1, '2', true)); // [1, '2', true]
// 参数为空时返回空数组
console.log(Array.of()); // []
```



这个方法的主要目的，是弥补数组构造函数`Array()`的不足。因为参数个数的不同，会导致`Array()`的行为有差异。

```ts
Array() // []
Array(3) // [undefined, undefined, undefined]
Array(3,11,8) // [3, 11, 8]
```

上面代码中，`Array()`方法没有参数、一个参数、三个参数时，返回的结果都不一样。只有当参数个数不少于 2 个时，`Array()`才会返回由参数组成的新数组。参数只有一个正整数时，实际上是指定数组的长度。



`Array.of()`基本上可以用来替代`Array()`或`new Array()`，并且不存在由于参数不同而导致的重载。它的行为非常统一。

```javascript
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```



`Array.of()`总是返回参数值组成的数组。如果没有参数，就返回一个空数组。`Array.of()`方法可以用下面的代码模拟实现。

```javascript
function ArrayOf(){
  return [].slice.call(arguments);
}
```



### 实例方法

#### at

长久以来，JavaScript 不支持数组的负索引，如果要引用数组的最后一个成员，不能写成`arr[-1]`，只能使用`arr[arr.length - 1]`。

这是因为方括号运算符`[]`在 JavaScript 语言里面，不仅用于数组，还用于对象。对于对象来说，方括号里面就是键名，比如`obj[1]`引用的是键名为字符串`1`的键，同理`obj[-1]`引用的是键名为字符串`-1`的键。由于 JavaScript 的数组是特殊的对象，所以方括号里面的负数无法再有其他语义了，也就是说，不可能添加新语法来支持负索引。

为了解决这个问题，[ES2022](https://github.com/tc39/proposal-relative-indexing-method/) 为数组实例增加了`at()`方法，接受一个整数作为参数，返回对应位置的成员，并支持负索引。这个方法不仅可用于数组，也可用于字符串和类型数组（TypedArray）。

```javascript
const arr = [5, 12, 8, 130, 44];
arr.at(2) // 8
arr.at(-2) // 130
```

如果参数位置超出了数组范围，`at()`返回`undefined`。

```javascript
const sentence = 'This is a sample sentence';

sentence.at(0); // 'T'
sentence.at(-1); // 'e'

sentence.at(-100) // undefined
sentence.at(100) // undefined
```



#### copyWithin

将一定范围索引的数组元素修改为此数组另一指定范围索引的元素。

```ts
console.log([1, 2, 3, 4].copyWithin(0,2,4)); // [3, 4, 3, 4]
// 参数1为负数表示倒数
console.log([1, 2, 3, 4].copyWithin(-2, 0)); // [1, 2, 1, 2]
console.log([1, 2, ,4].copyWithin(0, 2, 4)); // [, 4, , 4]
```



数组实例的`copyWithin()`方法，在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。也就是说，使用这个方法，会修改当前数组。

```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
```

它接受三个参数。

- target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
- start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
- end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。

这三个参数都应该是数值，如果不是，会自动转为数值。

```javascript
[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]
```

上面代码表示将从 3 号位直到数组结束的成员（4 和 5），复制到从 0 号位开始的位置，结果覆盖了原来的 1 和 2。

下面是更多例子。

```javascript
// 将3号位复制到0号位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 将3号位复制到0号位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}

// 将2号位到数组结束，复制到0号位
let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// 对于没有部署 TypedArray 的 copyWithin 方法的平台
// 需要采用下面的写法
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```



#### entries

遍历键值对。

```ts
for(let [key, value] of ['a', 'b'].entries()){
    console.log(key, value); // 依次输出：0 "a"、1 "b"
}
```



不使用 for-of 循环遍历时：

```ts
let entries = ['a', 'b'].entries();
console.log(entries.next().value); // [0, "a"]
console.log(entries.next().value); // [1, "b"]
 
// 数组含空位
console.log([...[,'a'].entries()]); // [[0, undefined], [1, "a"]]
```



ES6 提供三个新的方法——`entries()`，`keys()`和`values()`——用于遍历数组。它们都返回一个遍历器对象（详见《Iterator》一章），可以用`for...of`循环进行遍历，唯一的区别是`keys()`是对键名的遍历、`values()`是对键值的遍历，`entries()`是对键值对的遍历。

```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

如果不使用`for...of`循环，可以手动调用遍历器对象的`next`方法，进行遍历。

```javascript
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```



#### find

查找数组中符合条件的元素，若有多个符合条件的元素，则返回第一个元素。若无符合条件的元素，则返回undefined。

```ts
let arr = Array.of(1, 2, 3, 4);
console.log(arr.find(item => item > 2)); // 3
 
// 数组空位处理为 undefined
console.log([, 1].find(n => true)); // undefined
```



数组实例的`find()`方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回`undefined`。

```javascript
[1, 4, -5, 10].find((n) => n < 0)
// -5
```

上面代码找出数组中第一个小于 0 的成员。

```javascript
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```

上面代码中，`find()`方法的回调函数可以接受三个参数，依次为当前的值、当前的位置和原数组。

数组实例的`findIndex()`方法的用法与`find()`方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回`-1`。

```javascript
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

这两个方法都可以接受第二个参数，用来绑定回调函数的`this`对象。

```javascript
function f(v){
  return v > this.age;
}
let person = {name: 'John', age: 20};
[10, 12, 26, 15].find(f, person);    // 26
```

上面的代码中，`find()`函数接收了第二个参数`person`对象，回调函数中的`this`对象指向`person`对象。

另外，这两个方法都可以发现`NaN`，弥补了数组的`indexOf()`方法的不足。

```javascript
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

上面代码中，`indexOf()`方法无法识别数组的`NaN`成员，但是`findIndex()`方法可以借助`Object.is()`方法做到。

`find()`和`findIndex()`都是从数组的0号位，依次向后检查。[ES2022](https://github.com/tc39/proposal-array-find-from-last) 新增了两个方法`findLast()`和`findLastIndex()`，从数组的最后一个成员开始，依次向前检查，其他都保持不变。

```javascript
const array = [
  { value: 1 },
  { value: 2 },
  { value: 3 },
  { value: 4 }
];

array.findLast(n => n.value % 2 === 1); // { value: 3 }
array.findLastIndex(n => n.value % 2 === 1); // 2
```

上面示例中，`findLast()`和`findLastIndex()`从数组结尾开始，寻找第一个`value`属性为奇数的成员。结果，该成员是`{ value: 3 }`，位置是2号位。



#### findIndex

查找数组中符合条件的元素索引，若有多个符合条件的元素，则返回第一个元素索引。若无符合条件的元素，则返回-1。

```ts
let arr = Array.of(1, 2, 1, 3);
console.log(arr.findIndex(item => item == 2)); // 1
 
// 数组空位处理为 undefined
console.log([, 1].findIndex(n => true)); //0
```



find和findIndex方法都可以发现NaN，弥补了IndexOf()的不足。

```ts
[NaN].indexOf(NaN) // -1
[NaN].findIndex(y => Object.is(NaN, y)) // 0
```



#### findLast

#### findLastIndex



#### fill

将一定范围索引的数组元素内容填充为单个指定的值。

```ts
['a', 'b', 'c'].fill(7) // [7, 7, 7]
new Array(3).fill(7) // [7, 7, 7]

// 第二个和第三个参数，用于指定填充的起始位置和结束位置。
let arr = Array.of(1, 2, 3, 4);
console.log(arr.fill(0, 1, 2)); // [1, 0, 3, 4]
```



`fill`方法使用给定值，填充一个数组。

```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```

上面代码表明，`fill`方法用于空数组的初始化非常方便。数组中已有的元素，会被全部抹去。

`fill`方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。

```javascript
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']
```

上面代码表示，`fill`方法从 1 号位开始，向原数组填充 7，到 2 号位之前结束。

注意，如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。

```javascript
let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```



#### flat

```ts
console.log([1 ,[2, 3]].flat()); // [1, 2, 3]
// 指定转换的嵌套层数
console.log([1, [2, [3, [4, 5]]]].flat(2)); // [1, 2, 3, [4, 5]]
// 不管嵌套多少层
console.log([1, [2, [3, [4, 5]]]].flat(Infinity)); // [1, 2, 3, 4, 5]
// 自动跳过空位
console.log([1, [2, , 3]].flat());<p> // [1, 2, 3]
```



数组的成员有时还是数组，`Array.prototype.flat()`用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。

```javascript
[1, 2, [3, 4]].flat()
// [1, 2, 3, 4]
```

上面代码中，原数组的成员里面有一个数组，`flat()`方法将子数组的成员取出来，添加在原来的位置。

`flat()`默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以将`flat()`方法的参数写成一个整数，表示想要拉平的层数，默认为1。

```javascript
[1, 2, [3, [4, 5]]].flat()
// [1, 2, 3, [4, 5]]

[1, 2, [3, [4, 5]]].flat(2)
// [1, 2, 3, 4, 5]
```

上面代码中，`flat()`的参数为2，表示要“拉平”两层的嵌套数组。

如果不管有多少层嵌套，都要转成一维数组，可以用`Infinity`关键字作为参数。

```javascript
[1, [2, [3]]].flat(Infinity)
// [1, 2, 3]
```

如果原数组有空位，`flat()`方法会跳过空位。

```javascript
[1, 2, , 4, 5].flat()
// [1, 2, 4, 5]
```

`flatMap()`方法对原数组的每个成员执行一个函数（相当于执行`Array.prototype.map()`），然后对返回值组成的数组执行`flat()`方法。该方法返回一个新数组，不改变原数组。

```javascript
// 相当于 [[2, 4], [3, 6], [4, 8]].flat()
[2, 3, 4].flatMap((x) => [x, x * 2])
// [2, 4, 3, 6, 4, 8]
```

`flatMap()`只能展开一层数组。

```javascript
// 相当于 [[[2]], [[4]], [[6]], [[8]]].flat()
[1, 2, 3, 4].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

上面代码中，遍历函数返回的是一个双层的数组，但是默认只能展开一层，因此`flatMap()`返回的还是一个嵌套数组。

`flatMap()`方法的参数是一个遍历函数，该函数可以接受三个参数，分别是当前数组成员、当前数组成员的位置（从零开始）、原数组。

```javascript
arr.flatMap(function callback(currentValue[, index[, array]]) {
  // ...
}[, thisArg])
```

`flatMap()`方法还可以有第二个参数，用来绑定遍历函数里面的`this`。



#### flatMap

先对数组中每个元素进行了的处理，再对数组执行 flat() 方法。

```ts
console.log([1, 2, 3].flatMap(n => [n * 2])); // [2, 4, 6]
```



#### group

数组成员分组是一个常见需求，比如 SQL 有`GROUP BY`子句和函数式编程有 MapReduce 方法。现在有一个[提案](https://github.com/tc39/proposal-array-grouping)，为 JavaScript 新增了数组实例方法`group()`和`groupToMap()`，它们可以根据分组函数的运行结果，将数组成员分组。

`group()`的参数是一个分组函数，原数组的每个成员都会依次执行这个函数，确定自己是哪一个组。

```javascript
const array = [1, 2, 3, 4, 5];

array.group((num, index, array) => {
  return num % 2 === 0 ? 'even': 'odd';
});
// { odd: [1, 3, 5], even: [2, 4] }
```

`group()`的分组函数可以接受三个参数，依次是数组的当前成员、该成员的位置序号、原数组（上例是`num`、`index`和`array`）。分组函数的返回值应该是字符串（或者可以自动转为字符串），以作为分组后的组名。

`group()`的返回值是一个对象，该对象的键名就是每一组的组名，即分组函数返回的每一个字符串（上例是`even`和`odd`）；该对象的键值是一个数组，包括所有产生当前键名的原数组成员。

下面是另一个例子。

```javascript
[6.1, 4.2, 6.3].group(Math.floor)
// { '4': [4.2], '6': [6.1, 6.3] }
```

上面示例中，`Math.floor`作为分组函数，对原数组进行分组。它的返回值原本是数值，这时会自动转为字符串，作为分组的组名。原数组的成员根据分组函数的运行结果，进入对应的组。

`group()`还可以接受一个对象，作为第二个参数。该对象会绑定分组函数（第一个参数）里面的`this`，不过如果分组函数是一个箭头函数，该对象无效，因为箭头函数内部的`this`是固化的。

`groupToMap()`的作用和用法与`group()`完全一致，唯一的区别是返回值是一个 Map 结构，而不是对象。Map 结构的键名可以是各种值，所以不管分组函数返回什么值，都会直接作为组名（Map 结构的键名），不会强制转为字符串。这对于分组函数返回值是对象的情况，尤其有用。

```javascript
const array = [1, 2, 3, 4, 5];

const odd  = { odd: true };
const even = { even: true };
array.groupToMap((num, index, array) => {
  return num % 2 === 0 ? even: odd;
});
//  Map { {odd: true}: [1, 3, 5], {even: true}: [2, 4] }
```

上面示例返回的是一个 Map 结构，它的键名就是分组函数返回的两个对象`odd`和`even`。

总之，按照字符串分组就使用`group()`，按照对象分组就使用`groupToMap()`。



#### groupToMap



#### includes

数组是否包含指定值。注意：与Set和Map的has方法区分；Set的has方法用于查找值；Map的has方法用于查找键名。

```ts
[1, 2, 3].includes(1);    // true
[1, 2, 3].includes(1, 2); // false
[1, NaN, 3].includes(NaN); // true
```



`Array.prototype.includes`方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的`includes`方法类似。ES2016 引入了该方法。

```javascript
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
```

该方法的第二个参数表示搜索的起始位置，默认为`0`。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为`-4`，但数组长度为`3`），则会重置为从`0`开始。

```javascript
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

没有该方法之前，我们通常使用数组的`indexOf`方法，检查是否包含某个值。

```javascript
if (arr.indexOf(el) !== -1) {
  // ...
}
```

`indexOf`方法有两个缺点，一是不够语义化，它的含义是找到参数值的第一个出现位置，所以要去比较是否不等于`-1`，表达起来不够直观。二是，它内部使用严格相等运算符（`===`）进行判断，这会导致对`NaN`的误判。

```javascript
[NaN].indexOf(NaN)
// -1
```

`includes`使用的是不一样的判断算法，就没有这个问题。

```javascript
[NaN].includes(NaN)
// true
```

下面代码用来检查当前环境是否支持该方法，如果不支持，部署一个简易的替代版本。

```javascript
const contains = (() =>
  Array.prototype.includes
    ? (arr, value) => arr.includes(value)
    : (arr, value) => arr.some(el => el === value)
)();
contains(['foo', 'bar'], 'baz'); // => false
```

另外，Map 和 Set 数据结构有一个`has`方法，需要注意与`includes`区分。

- Map 结构的`has`方法，是用来查找键名的，比如`Map.prototype.has(key)`、`WeakMap.prototype.has(key)`、`Reflect.has(target, propertyKey)`。
- Set 结构的`has`方法，是用来查找值的，比如`Set.prototype.has(value)`、`WeakSet.prototype.has(value)`。



#### keys

遍历键名。

```ts
for(let key of ['a', 'b'].keys()){
    console.log(key); // 依次输出：0、1
}
```



数组含空位时，该空位的值为undefined。

```ts
console.log([...[,'a'].keys()]); // [0, 1]
```



#### sort

排序稳定性（stable sorting）是排序算法的重要属性，指的是排序关键字相同的项目，排序前后的顺序不变。

```javascript
const arr = [
  'peach',
  'straw',
  'apple',
  'spork'
];

const stableSorting = (s1, s2) => {
  if (s1[0] < s2[0]) return -1;
  return 1;
};

arr.sort(stableSorting)
// ["apple", "peach", "straw", "spork"]
```

上面代码对数组`arr`按照首字母进行排序。排序结果中，`straw`在`spork`的前面，跟原始顺序一致，所以排序算法`stableSorting`是稳定排序。

```javascript
const unstableSorting = (s1, s2) => {
  if (s1[0] <= s2[0]) return -1;
  return 1;
};

arr.sort(unstableSorting)
// ["apple", "peach", "spork", "straw"]
```

上面代码中，排序结果是`spork`在`straw`前面，跟原始顺序相反，所以排序算法`unstableSorting`是不稳定的。

常见的排序算法之中，插入排序、合并排序、冒泡排序等都是稳定的，堆排序、快速排序等是不稳定的。不稳定排序的主要缺点是，多重排序时可能会产生问题。假设有一个姓和名的列表，要求按照“姓氏为主要关键字，名字为次要关键字”进行排序。开发者可能会先按名字排序，再按姓氏进行排序。如果排序算法是稳定的，这样就可以达到“先姓氏，后名字”的排序效果。如果是不稳定的，就不行。

早先的 ECMAScript 没有规定，`Array.prototype.sort()`的默认排序算法是否稳定，留给浏览器自己决定，这导致某些实现是不稳定的。ES2019 明确规定，`Array.prototype.sort()`的默认排序算法必须稳定。这个规定已经做到了，现在 JavaScript 各个主要实现的默认排序算法都是稳定的。



#### toReversed

很多数组的传统方法会改变原数组，比如`push()`、`pop()`、`shift()`、`unshift()`等等。数组只要调用了这些方法，它的值就变了。[ES2023](https://github.com/tc39/proposal-change-array-by-copy)引入了四个新方法，对数组进行操作时，不改变原数组，而返回一个原数组的拷贝。

- `Array.prototype.toReversed() -> Array`
- `Array.prototype.toSorted(compareFn) -> Array`
- `Array.prototype.toSpliced(start, deleteCount, ...items) -> Array`
- `Array.prototype.with(index, value) -> Array`

它们分别对应数组的原有方法。

- `toReversed()`对应`reverse()`，用来颠倒数组成员的位置。
- `toSorted()`对应`sort()`，用来对数组成员排序。
- `toSpliced()`对应`splice()`，用来在指定位置，删除指定数量的成员，并插入新成员。
- `with(index, value)`对应`splice(index, 1, value)`，用来将指定位置的成员替换为新的值。

上面是这四个新方法对应的原有方法，含义和用法完全一样，唯一不同的是不会改变原数组，而是返回原数组操作后的拷贝。

下面是示例。

```javascript
const sequence = [1, 2, 3];
sequence.toReversed() // [3, 2, 1]
sequence // [1, 2, 3]

const outOfOrder = [3, 1, 2];
outOfOrder.toSorted() // [1, 2, 3]
outOfOrder // [3, 1, 2]

const array = [1, 2, 3, 4];
array.toSpliced(1, 2, 5, 6, 7) // [1, 5, 6, 7, 4]
array // [1, 2, 3, 4]

const correctionNeeded = [1, 1, 3];
correctionNeeded.with(1, 2) // [1, 2, 3]
correctionNeeded // [1, 1, 3]
```



#### toSorted

#### toSpliced



#### values

遍历键值。

```ts
for(let value of ['a', 'b'].values()){
    console.log(value); // 依次输出："a"、"b"
}
```



数组含空位时，该空位的值为undefined。

```ts
console.log([...[,'a'].values()]); // [undefined, "a"]
```



#### with



## 参考资料

- [数组的扩展 - ECMAScript 6入门](https://es6.ruanyifeng.com/#docs/array#%E6%89%A9%E5%B1%95%E8%BF%90%E7%AE%97%E7%AC%A6)



# ECMAScript ArrayBuffer

## 功能特性

数组缓冲区是内存中的一段地址，是定型数组的基础。实际字节数在创建时确定，之后只可修改其中的数据，不可修改大小。通过构造函数创建数组缓冲区：

```ts
let buffer = new ArrayBuffer(10);
console.log(buffer.byteLength); // 10
let buffer1 = buffer.slice(1, 3); // 分割已有数组缓冲区
console.log(buffer1.byteLength); // 2
```



视图是用来操作内存的接口，视图可以操作数组缓冲区或缓冲区字节的子集，并按照其中一种数值数据类型来读取和写入数据。DataView类型是一种通用的数组缓冲区视图，其支持所有8种数值型数据类型。

```ts
// 默认 DataView 可操作数组缓冲区全部内容
let buffer = new ArrayBuffer(10);
dataView = new DataView(buffer); 
dataView.setInt8(0,1);
console.log(dataView.getInt8(0)); // 1

// 通过设定偏移量(参数2)与长度(参数3)指定DataView可操作的字节范围
let buffer1 = new ArrayBuffer(10);
dataView1 = new DataView(buffer1, 0, 3);
dataView1.setInt8(5,1); // RangeError
```



# ECMAScript Int16Array

## 功能特性

定型数组是指数组缓冲区的特定类型的视图。可以强制使用特定的数据类型，而不是使用通用的 DataView 对象来操作数组缓冲区。通过数组缓冲区生成定型数组：

```ts
let buffer = new ArrayBuffer(10),
view = new Int8Array(buffer);
console.log(view.byteLength); // 10
```



通过构造函数创建定型数组：

```ts
let view = new Int32Array(10);
console.log(view.byteLength); // 40
console.log(view.length);     // 10

// 不传参则默认长度为0，在这种情况下数组缓冲区分配不到空间，创建的定型数组不能用来保存数据
let view1 = new Int32Array();
console.log(view1.byteLength); // 0
console.log(view1.length);     // 0

// 可接受参数包括定型数组、可迭代对象、数组、类数组对象
let arr = Array.from({
    0: '1',
    1: '2',
    2: 3,
    length: 3
});
let view2 = new Int16Array([1, 2]),
    view3 = new Int32Array(view2),
    view4 = new Int16Array(new Set([1, 2, 3])),
    view5 = new Int16Array([1, 2, 3]),
    view6 = new Int16Array(arr);
console.log(view2 .buffer === view3.buffer); // false
console.log(view4.byteLength); // 6
console.log(view5.byteLength); // 6
console.log(view6.byteLength); // 6
```



length属性不可写，如果尝试修改这个值，在非严格模式下会直接忽略该操作，在严格模式下会抛出错误。

```ts
let view = new Int16Array([1, 2]);
view.length = 3;
console.log(view.length); // 2
```



定型数组可使用 entries()、keys()、values()进行迭代。

```ts
let view = new Int16Array([1, 2]);
for(let [k, v] of view.entries()){
    console.log(k, v); // 依次输出：0 1、1 2
}
```



find()等方法也可用于定型数组，但是定型数组中的方法会额外检查数值类型是否安全，也会通过Symbol.species确认方法的返回值是定型数组而非普通数组。concat()方法由于两个定型数组合并结果不确定，故不能用于定型数组；另外，由于定型数组的尺寸不可更改，所以可以改变数组的尺寸的方法，例如 splice() ，不适用于定型数组。

```ts
let view = new Int16Array([1, 2]);
view.find((n) > 1); // 2
```



所有定型数组都含有静态of()方法和from()方法，运行效果分别与Array.of()方法和Array.from()方法相似，区别是定型数组的方法返回定型数组，而普通数组的方法返回普通数组。

```ts
let view = Int16Array.of(1, 2);
console.log(view instanceof Int16Array); // true
```



定型数组不是普通数组，不继承自Array。

```ts
let view = new Int16Array([1, 2]);
console.log(Array.isArray(view)); // false
```



定型数组中增加了set()与subarray()方法，set()方法用于将其他数组复制到已有定型数组，subarray()用于提取已有定型数组的一部分形成新的定型数组。

```ts
let view= new Int16Array(4);
view.set([1, 2]);
view.set([3, 4], 2);
console.log(view); // [1, 2, 3, 4]
 
let view= new Int16Array([1, 2, 3, 4]), 
    subview1 = view.subarray(), 
    subview2 = view.subarray(1), 
    subview3 = view.subarray(1, 3);
console.log(subview1); // [1, 2, 3, 4]
console.log(subview2); // [2, 3, 4]
console.log(subview3); // [2, 3]
```



## 源码解析

### 函数定义

> @since ES5

```ts
// lib.es5.d.ts
interface Int16Array<TArrayBuffer extends ArrayBufferLike = ArrayBufferLike> {
    /**
     * The size in bytes of each element in the array.
     */
    readonly BYTES_PER_ELEMENT: number;

    /**
     * The ArrayBuffer instance referenced by the array.
     */
    readonly buffer: TArrayBuffer;

    /**
     * The length in bytes of the array.
     */
    readonly byteLength: number;

    /**
     * The offset in bytes of the array.
     */
    readonly byteOffset: number;

    copyWithin(target: number, start: number, end?: number): this;

    every(predicate: (value: number, index: number, array: this) => unknown, thisArg?: any): boolean;

    fill(value: number, start?: number, end?: number): this;

    filter(predicate: (value: number, index: number, array: this) => any, thisArg?: any): Int16Array<ArrayBuffer>;

    find(predicate: (value: number, index: number, obj: this) => boolean, thisArg?: any): number | undefined;

    findIndex(predicate: (value: number, index: number, obj: this) => boolean, thisArg?: any): number;

    forEach(callbackfn: (value: number, index: number, array: this) => void, thisArg?: any): void;

    indexOf(searchElement: number, fromIndex?: number): number;

    join(separator?: string): string;

    lastIndexOf(searchElement: number, fromIndex?: number): number;

    readonly length: number;

    map(callbackfn: (value: number, index: number, array: this) => number, thisArg?: any): Int16Array<ArrayBuffer>;

    reduce(callbackfn: (previousValue: number, currentValue: number, currentIndex: number, array: this) => number): number;
    reduce(callbackfn: (previousValue: number, currentValue: number, currentIndex: number, array: this) => number, initialValue: number): number;
    reduce<U>(callbackfn: (previousValue: U, currentValue: number, currentIndex: number, array: this) => U, initialValue: U): U;

    reduceRight(callbackfn: (previousValue: number, currentValue: number, currentIndex: number, array: this) => number): number;
    reduceRight(callbackfn: (previousValue: number, currentValue: number, currentIndex: number, array: this) => number, initialValue: number): number;

    reduceRight<U>(callbackfn: (previousValue: U, currentValue: number, currentIndex: number, array: this) => U, initialValue: U): U;

    reverse(): this;

    set(array: ArrayLike<number>, offset?: number): void;

    slice(start?: number, end?: number): Int16Array<ArrayBuffer>;

    some(predicate: (value: number, index: number, array: this) => unknown, thisArg?: any): boolean;

    sort(compareFn?: (a: number, b: number) => number): this;

    subarray(begin?: number, end?: number): Int16Array<TArrayBuffer>;

    toLocaleString(): string;

    toString(): string;

    valueOf(): this;

    [index: number]: number;
}
```
