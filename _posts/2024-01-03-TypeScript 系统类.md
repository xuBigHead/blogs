---
layout: post
title: 第012章-TypeScript 系统类
categories: [TypeScript]
description: 
keywords: TypeScript 系统类.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# TypeScript Number

TypeScript 与 JavaScript 类似，支持 Number 对象。Number 对象是原始数值的包装对象。语法

```ts
var num = new Number(value);
```



**注意：** 如果一个参数值不能转换为一个数字将返回 NaN (非数字值)。



## 属性

下表列出了 Number 对象支持的属性：

|         属性          |                             描述                             |
| :-------------------: | :----------------------------------------------------------: |
|     **MAX_VALUE**     | 可表示的最大的数，MAX_VALUE 属性值接近于 1.79E+308。大于 MAX_VALUE 的值代表 "Infinity"。 |
|     **MIN_VALUE**     | 可表示的最小的数，即最接近 0 的正数 (实际上不会变成 0)。最大的负数是 -MIN_VALUE，MIN_VALUE 的值约为 5e-324。小于 MIN_VALUE ("underflow values") 的值将会转换为 0。 |
|        **NaN**        |                  非数字值（Not-A-Number）。                  |
| **NEGATIVE_INFINITY** |        负无穷大，溢出时返回该值。该值小于 MIN_VALUE。        |
| **POSITIVE_INFINITY** |        正无穷大，溢出时返回该值。该值大于 MAX_VALUE。        |
|     **prototype**     |   Number 对象的静态属性。使您有能力向对象添加属性和方法。    |
|    **constructor**    |            返回对创建此对象的 Number 函数的引用。            |



## 方法

### toExponential()

把对象的值转换为指数计数法。

```ts
//toExponential() 
var num1 = 1225.30 
var val = num1.toExponential(); 
console.log(val) // 输出： 1.2253e+3
```



### toFixed()

把数字转换为字符串，并对小数点指定位数。

```ts
var num3 = 177.234 
console.log("num3.toFixed() 为 "+num3.toFixed())    // 输出：177
console.log("num3.toFixed(2) 为 "+num3.toFixed(2))  // 输出：177.23
console.log("num3.toFixed(6) 为 "+num3.toFixed(6))  // 输出：177.234000
```



**toFixed()** 和 **toPrecision()** 转换的结果都是 **string** 类型，不是 **number** 类型。

```ts
var a = 177.234
var b = a.toFixed()
console.log(b, typeof b) // 177 string
var c = 1234.13412
var d = c.toPrecision(6)
console.log(d, typeof d) //1234.13 string
```



### toLocaleString()

把数字转换为字符串，使用本地数字格式顺序。

```ts
var num = new Number(177.1234); 
console.log( num.toLocaleString());  // 输出：177.1234
```



#### 与 toString() 的区别

**1.toLocaleString()，当数字是四位数及以上时，从右往左数，每三位用分号隔开，并且小数点后只保留三位；而toString()单纯将数字转换为字符串。**

**2.toLocaleString()，当目标是标准时间格式时，输出简洁年月日，时分秒；而toString()输出国际表述字符串。**

```ts
var num = new Number(1777.123488); 
console.log(num.toLocaleString());  // 输出：1,777.123
console.log(num.toString());  // 输出：1777.123488

var dateStr = new Date();
console.log(dateStr.toLocaleString());  // 输出：2022/2/15 16:48:35
console.log(dateStr.toString());  // 输出：Tue Feb 15 2022 16:48:58 GMT+0800 (中国标准时间)
```



### toPrecision()

把数字格式化为指定的长度。

```ts
var num = new Number(7.123456); 
console.log(num.toPrecision());  // 输出：7.123456 
console.log(num.toPrecision(1)); // 输出：7
console.log(num.toPrecision(2)); // 输出：7.1
```



### toString()

把数字转换为字符串，使用指定的基数。数字的基数是 2 ~ 36 之间的整数。若省略该参数，则使用基数 10。

```ts
var num = new Number(10); 
console.log(num.toString());  // 输出10进制：10
console.log(num.toString(2)); // 输出2进制：1010
console.log(num.toString(8)); // 输出8进制：12
```



### valueOf()

返回一个 Number 对象的原始数字值。

```ts
var num = new Number(10); 
console.log(num.valueOf()); // 输出：10
```



# TypeScript String

String 对象用于处理文本（字符串）。语法：

```ts
var txt = new String("string");
// 或者更简单方式：
var txt = "string";
```



## 属性

### constructor

对创建该对象的函数的引用。

```ts
var str = new String( "This is string" ); 
console.log("str.constructor is:" + str.constructor);
```



### length

返回字符串的长度。

```ts
var uname = new String("Hello World") 
console.log("Length "+uname.length)  // 输出 11
```



### prototype

允许向对象添加属性和方法。

```ts
function employee(id:number,name:string) { 
    this.id = id 
    this.name = name 
 } 
 var emp = new employee(123,"admin") 
 employee.prototype.email="admin@qq.com" // 添加属性 email
 console.log("员工号: "+emp.id) 
 console.log("员工姓名: "+emp.name) 
 console.log("员工邮箱: "+emp.email)
```



## 方法

### charAt()

返回在指定位置的字符。

```ts
var str = new String("RUNOOB"); 
console.log("str.charAt(0) 为:" + str.charAt(0)); // R
console.log("str.charAt(1) 为:" + str.charAt(1)); // U 
console.log("str.charAt(2) 为:" + str.charAt(2)); // N 
```



###  charCodeAt()

返回在指定的位置的字符的 Unicode 编码。

```ts
var str = new String("RUNOOB"); 
console.log("str.charCodeAt(0) 为:" + str.charCodeAt(0)); // 82
console.log("str.charCodeAt(1) 为:" + str.charCodeAt(1)); // 85 
console.log("str.charCodeAt(2) 为:" + str.charCodeAt(2)); // 78 
```



### concat()

连接两个或更多字符串，并返回新的字符串。

```ts
var str1 = new String( "RUNOOB" ); 
var str2 = new String( "GOOGLE" ); 
var str3 = str1.concat( str2 ); 
console.log("str1 + str2 : "+str3) // RUNOOBGOOGLE
```



### indexOf()

返回某个指定的字符串值在字符串中首次出现的位置。

```ts
var str1 = new String( "RUNOOB" ); 

var index = str1.indexOf( "OO" ); 
console.log("查找的字符串位置 :" + index );  // 3
```



### lastIndexOf()

从后向前搜索字符串，并从起始位置（0）开始计算返回字符串最后出现的位置。

```ts
var str1 = new String( "This is string one and again string" ); 
var index = str1.lastIndexOf( "string" );
console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 29
    
index = str1.lastIndexOf( "one" ); 
console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 15
```



### localeCompare()

用本地特定的顺序来比较两个字符串。

```ts
var str1 = new String( "This is beautiful string" );
var index = str1.localeCompare( "This is beautiful string");  
console.log("localeCompare first :" + index );  // 0
```



### match()

查找找到一个或多个正则表达式的匹配。

```ts
var str="The rain in SPAIN stays mainly in the plain"; 
var n=str.match(/ain/g);  // ain,ain,ain
```



### replace()

替换与正则表达式匹配的子串

```ts
var re = /(\w+)\s(\w+)/; 
var str = "zara ali"; 
var newstr = str.replace(re, "$2, $1"); 
console.log(newstr); // ali, zara
```



### search()

检索与正则表达式相匹配的值

```ts
var re = /apples/gi; 
var str = "Apples are round, and apples are juicy.";
if (str.search(re) == -1 ) { 
   console.log("Does not contain Apples" ); 
} else { 
   console.log("Contains Apples" ); 
} 
```



### slice()

提取字符串的片断，并在新的字符串中返回被提取的部分。



### split()

把字符串分割为子字符串数组。

```ts
var str = "Apples are round, and apples are juicy."; 
var splitted = str.split(" ", 3); 
console.log(splitted)  // [ 'Apples', 'are', 'round,' ]
```



### substr()

从起始索引号提取字符串中指定数目的字符。



### substring()

提取字符串中两个指定的索引号之间的字符。

```ts
var str = "RUNOOB GOOGLE TAOBAO FACEBOOK"; 
console.log("(1,2): "    + str.substring(1,2));   // U
console.log("(0,10): "   + str.substring(0, 10)); // RUNOOB GOO
console.log("(5): "      + str.substring(5));     // B GOOGLE TAOBAO FACEBOOK
```



### toLocaleLowerCase()

根据主机的语言环境把字符串转换为小写，只有几种语言（如土耳其语）具有地方特有的大小写映射。

```ts
var str = "Runoob Google"; 
console.log(str.toLocaleLowerCase( ));  // runoob google
```



### toLocaleUpperCase()

据主机的语言环境把字符串转换为大写，只有几种语言（如土耳其语）具有地方特有的大小写映射。

```ts
var str = "Runoob Google"; 
console.log(str.toLocaleUpperCase( ));  // RUNOOB GOOGLE
```



### toLowerCase()

把字符串转换为小写。



### toString()

返回字符串。



### toUpperCase()

把字符串转换为大写。



### valueOf()

返回指定字符串对象的原始值。

```ts
var str = new String("Runoob"); 
console.log(str.valueOf( ));  // Runoob
```



# TypeScript Array

可以使用 Array 对象创建数组，数组对象是使用单独的变量名来存储一系列的值。Array 对象的构造函数接受以下两种值：

- 表示数组大小的数值。
- 初始化的数组列表，元素使用逗号分隔值。



指定数组初始化大小：

```ts
var arr_names:number[] = new Array(4)  
for(var i = 0; i<arr_names.length; i++) { 
        arr_names[i] = i * 2 
        console.log(arr_names[i]) 
}
```



直接初始化数组元素：

```ts
var sites:string[] = new Array("Google","Runoob","Taobao","Facebook") 
for(var i = 0;i<sites.length;i++) { 
        console.log(sites[i]) 
}
```



## 数组解构

可以把数组元素赋值给变量，如下所示：

```ts
var arr:number[] = [12,13] 
var[x,y] = arr // 将数组的两个元素赋值给变量 x 和 y
```



## 数组迭代

使用 for 语句来循环输出数组的各个元素：

```ts
var j:any; 
var nums:number[] = [1001,1002,1003,1004] 
 
for(j in nums) { 
    console.log(nums[j]) 
}
```



## 多维数组

一个数组的元素可以是另外一个数组，这样就构成了多维数组（Multi-dimensional Array）。最简单的多维数组是二维数组，定义方式如下：

```ts
var arr_name:datatype[][]=[ [val1,val2,val3],[v1,v2,v3] ]
```



## 作为函数参数和返回值

### 作为函数参数

```ts
var sites:string[] = new Array("Google","Runoob","Taobao","Facebook") 
 
function disp(arr_sites:string[]) {
        for(var i = 0;i<arr_sites.length;i++) { 
                console.log(arr_sites[i]) 
        }  
}  
disp(sites);
```



### 作为函数返回值

```ts
function disp():string[] { 
        return new Array("Google", "Runoob", "Taobao", "Facebook");
} 
 
var sites:string[] = disp() 
for(var i in sites) { 
        console.log(sites[i]) 
}
```



## 方法

### concat()

连接两个或更多的数组，并返回结果。

```ts
var alpha = ["a", "b", "c"]; 
var numeric = [1, 2, 3];

var alphaNumeric = alpha.concat(numeric); 
console.log("alphaNumeric : " + alphaNumeric );    // a,b,c,1,2,3   
```



### every()

检测数值元素的每个元素是否都符合条件。

```ts
function isBigEnough(element, index, array) { 
        return (element >= 10); 
} 
        
var passed = [12, 5, 8, 130, 44].every(isBigEnough); 
console.log("Test Value : " + passed ); // false
```



### filter()

检测数值元素，并返回符合条件所有元素的数组。

```ts
function isBigEnough(element, index, array) { 
   return (element >= 10); 
} 
          
var passed = [12, 5, 8, 130, 44].filter(isBigEnough); 
console.log("Test Value : " + passed ); // 12,130,44
```



### forEach()

数组每个元素都执行一次回调函数。

```ts
let num = [7, 8, 9];
num.forEach(function (value) {
    console.log(value);
}); 
```



### indexOf()

搜索数组中的元素，并返回它所在的位置。如果搜索不到，返回值 -1，代表没有此项。

```ts
var index = [12, 5, 8, 130, 44].indexOf(8); 
console.log("index is : " + index );  // 2
```



### join()

把数组的所有元素放入一个字符串。

```ts
var arr = new Array("Google","Runoob","Taobao"); 
          
var str = arr.join(); 
console.log("str : " + str );  // Google,Runoob,Taobao
          
var str = arr.join(", "); 
console.log("str : " + str );  // Google, Runoob, Taobao
          
var str = arr.join(" + "); 
console.log("str : " + str );  // Google + Runoob + Taobao
```



### lastIndexOf()

返回一个指定的字符串值最后出现的位置，在一个字符串中的指定位置从后向前搜索。

```ts
var index = [12, 5, 8, 130, 44].lastIndexOf(8); 
console.log("index is : " + index );  // 2
```



### map()

通过指定函数处理数组的每个元素，并返回处理后的数组。

```ts
var numbers = [1, 4, 9]; 
var roots = numbers.map(Math.sqrt); 
console.log("roots is : " + roots );  // 1,2,3
```



### pop()

删除数组的最后一个元素并返回删除的元素。

```ts
var numbers = [1, 4, 9]; 
          
var element = numbers.pop(); 
console.log("element is : " + element );  // 9
          
var element = numbers.pop(); 
console.log("element is : " + element );  // 4
```



### push()

向数组的末尾添加一个或更多元素，并返回新的长度。

```ts
var numbers = new Array(1, 4, 9); 
var length = numbers.push(10); 
console.log("new numbers is : " + numbers );  // 1,4,9,10 
length = numbers.push(20); 
console.log("new numbers is : " + numbers );  // 1,4,9,10,20
```



### reduce()

将数组元素计算为一个值（从左到右）。

```ts
var total = [0, 1, 2, 3].reduce(function(a, b){ return a + b; }); 
console.log("total is : " + total );  // 6
```



### reduceRight()

将数组元素计算为一个值（从右到左）。

```ts
var total = [0, 1, 2, 3].reduceRight(function(a, b){ return a + b; }); 
console.log("total is : " + total );  // 6
```



### reverse()

反转数组的元素顺序。

```ts
var arr = [0, 1, 2, 3].reverse(); 
console.log("Reversed array is : " + arr );  // 3,2,1,0
```



### shift()

删除并返回数组的第一个元素。

```ts
var arr = [10, 1, 2, 3].shift(); 
console.log("Shifted value is : " + arr );  // 10
```



### slice()

选取数组的的一部分，并返回一个新数组。

```ts
var arr = ["orange", "mango", "banana", "sugar", "tea"]; 
console.log("arr.slice( 1, 2) : " + arr.slice( 1, 2) );  // mango
console.log("arr.slice( 1, 3) : " + arr.slice( 1, 3) );  // mango,banana
```



### some()

检测数组元素中是否有元素符合指定条件。

```ts
function isBigEnough(element, index, array) { 
   return (element >= 10); 
          
} 
          
var retval = [2, 5, 8, 1, 4].some(isBigEnough);
console.log("Returned value is : " + retval );  // false
          
var retval = [12, 5, 8, 1, 4].some(isBigEnough); 
console.log("Returned value is : " + retval );  // true
```



### sort()

对数组的元素进行排序。

```ts
var arr = new Array("orange", "mango", "banana", "sugar"); 
var sorted = arr.sort(); 
console.log("Returned string is : " + sorted );  // banana,mango,orange,sugar
```



### splice()

从数组中添加或删除元素。

```ts
var arr = ["orange", "mango", "banana", "sugar", "tea"];  
var removed = arr.splice(2, 0, "water");  
console.log("After adding 1: " + arr );    // orange,mango,water,banana,sugar,tea 
console.log("removed is: " + removed); 
          
removed = arr.splice(3, 1);  
console.log("After removing 1: " + arr );  // orange,mango,water,sugar,tea 
console.log("removed is: " + removed);  // banana
```



### toString()

把数组转换为字符串，并返回结果。

```ts
var arr = new Array("orange", "mango", "banana", "sugar");         
var str = arr.toString(); 
console.log("Returned string is : " + str );  // orange,mango,banana,sugar
```



### unshift()

向数组的开头添加一个或更多元素，并返回新的长度。

```ts
var arr = new Array("orange", "mango", "banana", "sugar"); 
var length = arr.unshift("water"); 
console.log("Returned array is : " + arr );  // water,orange,mango,banana,sugar 
console.log("Length of the array is : " + length ); // 5
```



# TypeScript Map

Map 对象保存键值对，并且能够记住键的原始插入顺序。任何值(对象或者原始值) 都可以作为一个键或一个值。Map 是 ES6 中引入的一种新的数据结构，可以参考 [ES6 Map 与 Set](https://www.runoob.com/w3cnote/es6-map-set.html)。



## 创建Map

TypeScript 使用 Map 类型和 new 关键字来创建 Map：

```ts
let myMap = new Map();
```



初始化 Map，可以以数组的格式来传入键值对：

```ts
let myMap = new Map([
        ["key1", "value1"],
        ["key2", "value2"]
    ]); 
```



Map 相关的函数与属性：

- **map.clear()** – 移除 Map 对象的所有键/值对 。
- **map.set()** – 设置键值对，返回该 Map 对象。
- **map.get()** – 返回键对应的值，如果不存在，则返回 undefined。
- **map.has()** – 返回一个布尔值，用于判断 Map 中是否包含键对应的值。
- **map.delete()** – 删除 Map 中的元素，删除成功返回 true，失败返回 false。
- **map.size** – 返回 Map 对象键/值对的数量。
- **map.keys()** - 返回一个 Iterator 对象， 包含了 Map 对象中每个元素的键 。
- **map.values()** – 返回一个新的Iterator对象，包含了Map对象中每个元素的值 。



```ts
let nameSiteMapping = new Map();
 
// 设置 Map 对象
nameSiteMapping.set("Google", 1);
nameSiteMapping.set("Runoob", 2);
nameSiteMapping.set("Taobao", 3);
 
// 获取键对应的值
console.log(nameSiteMapping.get("Runoob"));     // 2
 
// 判断 Map 中是否包含键对应的值
console.log(nameSiteMapping.has("Taobao"));       // true
console.log(nameSiteMapping.has("Zhihu"));        // false
 
// 返回 Map 对象键/值对的数量
console.log(nameSiteMapping.size);                // 3
 
// 删除 Runoob
console.log(nameSiteMapping.delete("Runoob"));    // true
console.log(nameSiteMapping);
// 移除 Map 对象的所有键/值对
nameSiteMapping.clear();             // 清除 Map
console.log(nameSiteMapping);
```



## 迭代Map

Map 对象中的元素是按顺序插入的，可以迭代 Map 对象，每一次迭代返回 [key, value] 数组。TypeScript使用 **for...of** 来实现迭代：

```ts
let nameSiteMapping = new Map();
 
nameSiteMapping.set("Google", 1);
nameSiteMapping.set("Runoob", 2);
nameSiteMapping.set("Taobao", 3);
 
// 迭代 Map 中的 key
for (let key of nameSiteMapping.keys()) {
    console.log(key);                  
}
 
// 迭代 Map 中的 value
for (let value of nameSiteMapping.values()) {
    console.log(value);                 
}
 
// 迭代 Map 中的 key => value
for (let entry of nameSiteMapping.entries()) {
    console.log(entry[0], entry[1]);   
}
 
// 使用对象解析
for (let [key, value] of nameSiteMapping) {
    console.log(key, value);            
}
```
