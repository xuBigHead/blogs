---
layout: post
title: ECMAScript Map.md
categories: [ECMAScript]
description: 
keywords: ECMAScript Map.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# ECMAScript Map

## 概述

JavaScript的对象是键值对的集合，只能用字符串当作键，在使用上有很大的限制。

```javascript
const data = {};
const element = document.getElementById('myDiv');

data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```

上面代码原意是将一个 DOM 节点作为对象`data`的键，但是由于对象只接受字符串作为键名，所以`element`被自动转为字符串`[object HTMLDivElement]`。



ES6提供了map数据结构，类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。

```ts
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```



Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键，比如`0`和`-0`就是一个键，布尔值`true`和字符串`true`则是两个不同的键。另外，`undefined`和`null`也是两个不同的键。虽然`NaN`不严格相等于自身，但 Map 将其视为同一个键。



**Maps和Object的区别**

- Object的键只能是字符串或者Symbols，但Map的键可以是任意值；
- Map中的键值是有序的（FIFO 原则），而添加到对象中的键则不是；
- Map的键值对个数可以从size属性获取，而Object的键值对个数只能手动计算；
- Object都有自己的原型，原型链上的键名有可能和对象上的设置的键名产生冲突。



## 源码解析

### 成员定义

> @since ES6

```ts
// lib.es2025.collection.d.ts
interface Map<K, V> {
    /**
     * @returns size属性返回 Map 结构的成员总数
     */
    readonly size: number;
}
```



### 方法定义

> @since ES6

```ts
// lib.es2025.collection.d.ts
interface Map<K, V> {
    /**
     * 清除所有成员
     */
    clear(): void;
    
    /**
     * 删除某个键，返回true。如果删除失败，返回false
     */
    delete(key: K): boolean;
    
    /**
     * Executes a provided function once per each key/value pair in the Map, in insertion order
     */
    forEach(callbackfn: (value: V, key: K, map: Map<K, V>) => void, thisArg?: any): void;
    
    /**
     * 读取key对应的键值，如果找不到key，返回undefined
     * 
     * @param 键 
     */
    get(key: K): V | undefined;
    
    /**
     * 返回一个布尔值，表示某个键是否在Map数据结构中。
     */
    has(key: K): boolean;
    
    /**
     * 设置key所对应的键值，然后返回整个Map结构。如果key已经有值，则键值会被更新，否则就新生成该键。
     */
    set(key: K, value: V): this;
}

interface MapConstructor {
    new (): Map<any, any>;
    new <K, V>(entries?: readonly (readonly [K, V])[] | null): Map<K, V>;
    readonly prototype: Map<any, any>;
}
declare var Map: MapConstructor;
```



## 实践应用

### 构造方法

可以通过无参构造方法创建一个元素数量为0的空map。

```ts
var myMap = new Map();
```



Map构造方法也可以接受一个数组作为参数，该数组的成员是一个个表示键值对的数组。

```ts
var kvArray = [["key1", "value1"], ["key2", "value2"]];
var myMap = new Map(kvArray); // Map构造函数可以将一个二维键值对数组转换成一个Map对象
var outArray = Array.from(myMap); // 使用Array.from函数可以将一个Map对象转换成一个二维键值对数组
```



任何具有Iterator接口且每个成员都是一个双元素的数组的数据结构都可以当作`Map`构造函数的参数。这就是说，`Set`和`Map`都可以用来生成新的 Map。

```ts
const set = new Set([['foo', 1], ['bar', 2]]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
```



#### Map的克隆

```ts
var myMap1 = new Map([["key1", "value1"], ["key2", "value2"]]);
var myMap2 = new Map(myMap1);
console.log(original === clone); // false。Map对象构造函数生成实例，迭代出新的对象。
```



#### Map的合并

```ts
var first = new Map([[1, 'one'], [2, 'two'], [3, 'three'],]);
var second = new Map([[1, 'uno'], [2, 'dos']]);
var merged = new Map([...first, ...second]); // 合并两个Map对象时，如果有重复的键值，则后面的会覆盖前面的，对应值即uno、dos、three
```



### 实例方法

#### clear

`clear()`方法清除所有成员，没有返回值。

```ts
let map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
map.clear()
map.size // 0
```



#### delete

`delete()`方法删除某个键，返回`true`。如果删除失败，返回`false`。

```ts
const m = new Map();
m.set(undefined, 'nah');
m.has(undefined)     // true

m.delete(undefined)
m.has(undefined)       // false
```



#### entries

遍历获取Map所有键值对，需要特别注意的是，Map 的遍历顺序就是插入顺序。

```ts
var myMap = new Map();
var mapIterator = myMap.entries(); // 返回MapIterator类型对象，它按插入顺序包含了Map对象中每个元素的[key, value]数组

for (let item of myMap.entries()) {
  console.log(item[0], item[1]);
}

for (let [key, value] of myMap.entries()) {
  console.log(key, value);
}
```



Map结构的默认遍历器接口（Symbol.iterator属性），就是entries方法。

```ts
map[Symbol.iterator] === map.entries // true

// 等同于使用map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}
```



#### forEach

与数组的`forEach`方法类似，也可以实现遍历。

```ts
var myMap = new Map();
myMap.set(0, "zero");
myMap.set(1, "one");

myMap.forEach(function(value, key, map) {
    console.log(key + " = " + value); // 将会显示两个log，一个是"0 = zero"另一个是"1 = one"
}, myMap)
```



`forEach`方法还可以接受第二个参数，用来绑定`this`。

```javascript
const reporter = {
    report: function(key, value) {
        console.log("Key: %s, Value: %s", key, value);
    }
};

map.forEach(function(value, key, map) {
    this.report(key, value);
}, reporter);
```



#### get

`get`方法读取`key`对应的键值，如果找不到`key`，返回`undefined`。

```ts
var myMap = new Map();
var keyString = "a string"; 
 
myMap.set(keyString, "和键'a string'关联的值");
myMap.get(keyString);    // 和键'a string'关联的值
myMap.get("a string");   // 和键'a string'关联的值，因为keyString === 'a string'
```



对象类型的key

```ts
var myMap = new Map();
var keyObj = {};
 
myMap.set(keyObj, "和键 keyObj 关联的值");
myMap.get(keyObj); // 和键keyObj关联的值
myMap.get({}); // undefined, 因为 keyObj !== {}
```



函数类型的key

```ts
var myMap = new Map();
var keyFunc = function () {};
 
myMap.set(keyFunc, "和键 keyFunc 关联的值");
myMap.get(keyFunc); // 和键keyFunc关联的值
myMap.get(function() {}) // undefined, 因为keyFunc !== function () {}
```



NaN类型的key

```ts
var myMap = new Map();
myMap.set(NaN, "not a number");
myMap.get(NaN); // "not a number"

var otherNaN = Number("foo");
myMap.get(otherNaN); // "not a number"，虽然NaN和任何值甚至和自己都不相等(NaN !== NaN返回true)，但是NaN作为Map的键来说是没有区别的
```



#### has

`has`方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。

```ts
const m = new Map();

m.set('edition', 6);
m.set(262, 'standard');
m.set(undefined, 'nah');

m.has('edition')     // true
m.has('years')       // false
m.has(262)           // true
m.has(undefined)     // true
```



#### keys

获取Map所有键。

```ts
var myMap = new Map();
var mapIterator = myMap.keys(); // 返回MapIterator类型对象，它按插入顺序包含了Map对象中每个元素的键

for (let key of map.keys()) {
  console.log(key);
}
```



#### set

`set`方法设置键名`key`对应的键值为`value`，然后返回整个 Map 结构。如果`key`已经有值，则键值会被更新，否则就新生成该键。

```ts
const m = new Map();

m.set('edition', 6)        // 键是字符串
m.set(262, 'standard')     // 键是数值
m.set(undefined, 'nah')    // 键是 undefined
```



`set`方法返回的是当前的`Map`对象，因此可以采用链式写法。

```ts
let map = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
```



#### values

获取Map所有值。

```ts
var myMap = new Map();
var mapIterator = myMap.values(); // 返回MapIterator类型对象，它按插入顺序包含了Map对象中每个元素的值
```



### 其他应用

#### Map和数组互相转换

Map 结构转为数组结构，比较快速的方法是使用扩展运算符（`...`）。

```javascript
const map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three'],
]);

[...map.keys()] // [1, 2, 3]
[...map.values()] // ['one', 'two', 'three']
[...map.entries()] // [[1,'one'], [2, 'two'], [3, 'three']]
[...map] // [[1,'one'], [2, 'two'], [3, 'three']]
```



结合数组的`map`方法、`filter`方法，可以实现 Map 的遍历和过滤（Map 本身没有`map`和`filter`方法）。

```javascript
const map0 = new Map()
.set(1, 'a')
.set(2, 'b')
.set(3, 'c');

const map1 = new Map(
    [...map0].filter(([k, v]) => k < 3)
);
// 产生 Map 结构 {1 => 'a', 2 => 'b'}

const map2 = new Map(
    [...map0].map(([k, v]) => [k * 2, '_' + v])
);
// 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```



将数组传入 Map 构造函数，就可以转为 Map。

```ts
new Map([
    [true, 7],
    [{foo: 3}, ['abc']]
])
```



#### Map和对象互相转换

如果所有 Map 的键都是字符串，它可以无损地转为对象。

```javascript
function strMapToObj(strMap) {
    let obj = Object.create(null);
    for (let [k,v] of strMap) {
        obj[k] = v;
    }
    return obj;
}

const myMap = new Map().set('yes', true).set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```

如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名。



对象转为 Map 可以通过`Object.entries()`。

```javascript
let obj = {"a":1, "b":2};
let map = new Map(Object.entries(obj));
```



此外，也可以自己实现一个转换函数。

```javascript
function objToStrMap(obj) {
    let strMap = new Map();
    for (let k of Object.keys(obj)) {
        strMap.set(k, obj[k]);
    }
    return strMap;
}

objToStrMap({yes: true, no: false}) // Map {"yes" => true, "no" => false}
```



#### Map和Json互相转换

Map 转为 JSON 要区分两种情况。一种情况是，Map 的键名都是字符串，这时可以选择转为对象 JSON。

```javascript
function strMapToJson(strMap) {
    return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap) // '{"yes":true,"no":false}'
```



另一种情况是，Map 的键名有非字符串，这时可以选择转为数组 JSON。

```javascript
function mapToArrayJson(map) {
    return JSON.stringify([...map]);
}

let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap) // '[[true,7],[{"foo":3},["abc"]]]'
```



JSON 转为 Map，正常情况下，所有键名都是字符串。

```javascript
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"yes": true, "no": false}') // Map {'yes' => true, 'no' => false}
```



但是，有一种特殊情况，整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为 Map。这往往是 Map 转为数组 JSON 的逆操作。

```javascript
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]') // Map {true => 7, Object {foo: 3} => ['abc']}
```



## 参考资料

- [Set 和 Map 数据结构 - ECMAScript 6入门](https://es6.ruanyifeng.com/#docs/set-map)



# ECMAScript WeakMap

## 概述

WeakMap结构与Map结构基本类似，唯一的区别是它只接受对象作为键名（null除外），不接受原始类型的值作为键名，而且键名所指向的对象，不计入垃圾回收机制。

WeakMap的设计目的在于，键名是对象的弱引用（垃圾回收机制不将该引用考虑在内），所以其所对应的对象可能会被自动回收。当对象被回收后，WeakMap自动移除对应的键值对。典型应用是，一个对应DOM元素的WeakMap结构，当某个DOM元素被清除，其所对应的WeakMap记录就会自动被移除。基本上，WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。

WeakMap与Map在API上的区别主要是两个，一是没有遍历操作（即没有key()、values()和entries()方法），也没有size属性；二是无法清空，即不支持clear方法。这与WeakMap的键不被计入引用、被垃圾回收机制忽略有关。因此，WeakMap只有四个方法可用：get()、set()、has()、delete()。



`WeakMap`结构与`Map`结构类似，也是用于生成键值对的集合。

```javascript
// WeakMap 可以使用 set 方法添加成员
const wm1 = new WeakMap();
const key = {foo: 1};
wm1.set(key, 2);
wm1.get(key) // 2

// WeakMap 也可以接受一个数组，
// 作为构造函数的参数
const k1 = [1, 2, 3];
const k2 = [4, 5, 6];
const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]);
wm2.get(k2) // "bar"
```

`WeakMap`与`Map`的区别有两点。

首先，`WeakMap`只接受对象（`null`除外）和 [Symbol 值](https://github.com/tc39/proposal-symbols-as-weakmap-keys)作为键名，不接受其他类型的值作为键名。

```javascript
const map = new WeakMap();
map.set(1, 2) // 报错
map.set(null, 2) // 报错
map.set(Symbol(), 2) // 不报错
```

上面代码中，如果将数值`1`和`null`作为 WeakMap 的键名，都会报错，将 Symbol 值作为键名不会报错。

其次，`WeakMap`的键名所指向的对象，不计入垃圾回收机制。

`WeakMap`的设计目的在于，有时我们想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用。请看下面的例子。

```javascript
const e1 = document.getElementById('foo');
const e2 = document.getElementById('bar');
const arr = [
  [e1, 'foo 元素'],
  [e2, 'bar 元素'],
];
```

上面代码中，`e1`和`e2`是两个对象，我们通过`arr`数组对这两个对象添加一些文字说明。这就形成了`arr`对`e1`和`e2`的引用。

一旦不再需要这两个对象，我们就必须手动删除这个引用，否则垃圾回收机制就不会释放`e1`和`e2`占用的内存。

```javascript
// 不需要 e1 和 e2 的时候
// 必须手动删除引用
arr [0] = null;
arr [1] = null;
```

上面这样的写法显然很不方便。一旦忘了写，就会造成内存泄露。

WeakMap 就是为了解决这个问题而诞生的，它的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

基本上，如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。一个典型应用场景是，在网页的 DOM 元素上添加数据，就可以使用`WeakMap`结构。当该 DOM 元素被清除，其所对应的`WeakMap`记录就会自动被移除。

```javascript
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```

上面代码中，先新建一个 WeakMap 实例。然后，将一个 DOM 节点作为键名存入该实例，并将一些附加信息作为键值，一起存放在 WeakMap 里面。这时，WeakMap 里面对`element`的引用就是弱引用，不会被计入垃圾回收机制。

也就是说，上面的 DOM 节点对象除了 WeakMap 的弱引用外，其他位置对该对象的引用一旦消除，该对象占用的内存就会被垃圾回收机制释放。WeakMap 保存的这个键值对，也会自动消失。

总之，`WeakMap`的专用场合就是，它的键所对应的对象，可能会在将来消失。`WeakMap`结构有助于防止内存泄漏。

注意，WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。

```javascript
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```

上面代码中，键值`obj`是正常引用。所以，即使在 WeakMap 外部消除了`obj`的引用，WeakMap 内部的引用依然存在。

### WeakMap 的语法

WeakMap 与 Map 在 API 上的区别主要是两个，一是没有遍历操作（即没有`keys()`、`values()`和`entries()`方法），也没有`size`属性。因为没有办法列出所有键名，某个键名是否存在完全不可预测，跟垃圾回收机制是否运行相关。这一刻可以取到键名，下一刻垃圾回收机制突然运行了，这个键名就没了，为了防止出现不确定性，就统一规定不能取到键名。二是无法清空，即不支持`clear`方法。因此，`WeakMap`只有四个方法可用：`get()`、`set()`、`has()`、`delete()`。

```javascript
const wm = new WeakMap();

// size、forEach、clear 方法都不存在
wm.size // undefined
wm.forEach // undefined
wm.clear // undefined
```

### WeakMap 的示例

WeakMap 的例子很难演示，因为无法观察它里面的引用会自动消失。此时，其他引用都解除了，已经没有引用指向 WeakMap 的键名了，导致无法证实那个键名是不是存在。

贺师俊老师[提示](https://github.com/ruanyf/es6tutorial/issues/362#issuecomment-292109104)，如果引用所指向的值占用特别多的内存，就可以通过 Node 的`process.memoryUsage`方法看出来。根据这个思路，网友[vtxf](https://github.com/ruanyf/es6tutorial/issues/362#issuecomment-292451925)补充了下面的例子。

首先，打开 Node 命令行。

```bash
$ node --expose-gc
```

上面代码中，`--expose-gc`参数表示允许手动执行垃圾回收机制。

然后，执行下面的代码。

```javascript
// 手动执行一次垃圾回收，保证获取的内存使用状态准确
> global.gc();
undefined

// 查看内存占用的初始状态，heapUsed 为 4M 左右
> process.memoryUsage();
{ rss: 21106688,
  heapTotal: 7376896,
  heapUsed: 4153936,
  external: 9059 }

> let wm = new WeakMap();
undefined

// 新建一个变量 key，指向一个 5*1024*1024 的数组
> let key = new Array(5 * 1024 * 1024);
undefined

// 设置 WeakMap 实例的键名，也指向 key 数组
// 这时，key 数组实际被引用了两次，
// 变量 key 引用一次，WeakMap 的键名引用了第二次
// 但是，WeakMap 是弱引用，对于引擎来说，引用计数还是1
> wm.set(key, 1);
WeakMap {}

> global.gc();
undefined

// 这时内存占用 heapUsed 增加到 45M 了
> process.memoryUsage();
{ rss: 67538944,
  heapTotal: 7376896,
  heapUsed: 45782816,
  external: 8945 }

// 清除变量 key 对数组的引用，
// 但没有手动清除 WeakMap 实例的键名对数组的引用
> key = null;
null

// 再次执行垃圾回收
> global.gc();
undefined

// 内存占用 heapUsed 变回 4M 左右，
// 可以看到 WeakMap 的键名引用没有阻止 gc 对内存的回收
> process.memoryUsage();
{ rss: 20639744,
  heapTotal: 8425472,
  heapUsed: 3979792,
  external: 8956 }
```

上面代码中，只要外部的引用消失，WeakMap 内部的引用，就会自动被垃圾回收清除。由此可见，有了 WeakMap 的帮助，解决内存泄漏就会简单很多。

Chrome 浏览器的 Dev Tools 的 Memory 面板，有一个垃圾桶的按钮，可以强制垃圾回收（garbage collect）。这个按钮也能用来观察 WeakMap 里面的引用是否消失。

### WeakMap 的用途

前文说过，WeakMap 应用的典型场合就是 DOM 节点作为键名。下面是一个例子。

```javascript
let myWeakmap = new WeakMap();

myWeakmap.set(
  document.getElementById('logo'),
  {timesClicked: 0})
;

document.getElementById('logo').addEventListener('click', function() {
  let logoData = myWeakmap.get(document.getElementById('logo'));
  logoData.timesClicked++;
}, false);
```

上面代码中，`document.getElementById('logo')`是一个 DOM 节点，每当发生`click`事件，就更新一下状态。我们将这个状态作为键值放在 WeakMap 里，对应的键名就是这个节点对象。一旦这个 DOM 节点删除，该状态就会自动消失，不存在内存泄漏风险。

WeakMap 的另一个用处是部署私有属性。

```javascript
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

const c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
// DONE
```

上面代码中，`Countdown`类的两个内部属性`_counter`和`_action`，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏。



## 源码解析



## 实践应用

### 其他应用

#### 部署私有属性

WeakMap的一个用处是部署私有属性。下面代码中，Countdown类的两个内部属性_counter和_action，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏。

```ts
et _counter = new WeakMap();
let _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

let c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
```



## 参考资料

- [Set 和 Map 数据结构 - ECMAScript 6入门](https://es6.ruanyifeng.com/#docs/set-map)




