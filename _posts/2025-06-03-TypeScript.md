---
layout: post
title: TypeScript
categories: [TypeScript]
description: 
keywords: TypeScript.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# TypeScript 简介

TypeScript 是 JavaScript 的一个超集，支持 ECMAScript 6 标准（[ES6 教程](https://www.runoob.com/w3cnote/es6-tutorial.html)）。TypeScript 由微软开发的自由和开源的编程语言。TypeScript 设计目标是开发大型应用，它可以编译成纯 JavaScript，编译出来的 JavaScript 可以运行在任何浏览器上。



## 语言特性

TypeScript 是一种给 JavaScript 添加特性的语言扩展。增加的功能包括：

- 类型批注和编译时类型检查
- 类型推断
- 类型擦除
- 接口
- 枚举
- Mixin
- 泛型编程
- 名字空间
- 元组
- Await

以下功能是从 ECMA 2015 反向移植而来：

- 类
- 模块
- lambda 函数的箭头语法
- 可选参数以及默认参数



## JavaScript 与 TypeScript 的区别

TypeScript 是 JavaScript 的超集，扩展了 JavaScript 的语法，因此现有的 JavaScript 代码可与 TypeScript 一起工作无需任何修改，TypeScript 通过类型注解提供编译时的静态类型检查。

TypeScript 可处理已有的 JavaScript 代码，并只对其中的 TypeScript 代码进行编译。

![img](https://oss.xubighead.top/oss/image/202506/1929820019182637058.png)



## TypeScript 安装

安装 typescript：

```sh
npm install -g typescript
```



安装完成后可以使用 **tsc** 命令来执行 TypeScript 的相关代码，以下是查看版本号：

```sh
$ tsc -v
Version 3.2.2
```



## 保留关键字

TypeScript 保留关键字如下表所示：

| break    | as         | catch      | switch   |
| -------- | ---------- | ---------- | -------- |
| case     | if         | throw      | else     |
| var      | number     | string     | get      |
| module   | type       | instanceof | typeof   |
| public   | private    | enum       | export   |
| finally  | for        | while      | void     |
| null     | super      | this       | new      |
| in       | return     | true       | false    |
| any      | extends    | static     | let      |
| package  | implements | interface  | function |
| do       | try        | yield      | const    |
| continue |            |            |          |



## 语法特点

- TypeScript 会忽略程序中出现的空格、制表符和换行符。空格、制表符通常用来缩进代码，使代码易于阅读和理解。
- TypeScript 区分大写和小写字符。
- TypeScript 中分号是可选的，建议使用。



## 注释类型

- **单行注释 ( // )** − 在 // 后面的文字都是注释内容。
- **多行注释 (/\* \*/)** − 这种注释可以跨越多行。



## TypeScript与面向对象

面向对象是一种对现实世界理解和抽象的方法，TypeScript 是一种面向对象的编程语言。面向对象主要有两个概念：对象和类。

- **对象**：对象是类的一个实例，有状态和行为。例如，一条狗是一个对象，它的状态有：颜色、名字、品种；行为有：摇尾巴、叫、吃等。
- **类**：类是一个模板，它描述一类对象的行为和状态。
- **方法**：方法是类的操作的实现步骤。



```typescript
class Site { 
   name():void { 
      console.log("TypeScript");
   } 
} 
var obj = new Site(); 
obj.name();
```



# TypeScript 基础语法

## 基础类型

TypeScript 和 JavaScript 没有整数类型。TypeScript 包含的数据类型如下表:



### 任意类型

- 关键字：any
- 描述：声明为 any 的变量可以赋予任意类型的值。



任意值是 TypeScript 针对编程时类型不明确的变量使用的一种数据类型，它常用于以下三种情况：

1. 变量的值会动态改变时，比如来自用户的输入，任意值类型可以让这些变量跳过编译阶段的类型检查，示例代码如下：

```ts
let x: any = 1;    // 数字类型
x = 'I am who I am';    // 字符串类型
x = false;    // 布尔类型
```



2. 改写现有代码时，任意值允许在编译时可选择地包含或移除类型检查，示例代码如下：

```ts
let x: any = 4;
x.ifItExists();    // 正确，ifItExists方法在运行时可能存在，但这里并不会检查
x.toFixed();    // 正确
```



3. 定义存储各种类型数据的数组时，示例代码如下：

```ts
let arrayList: any[] = [1, false, 'fine'];
arrayList[1] = 100;
```



### 数字类型

- 关键字：number
- 描述：双精度 64 位浮点值，可以用来表示整数和分数。

```ts
let binaryLiteral: number = 0b1010; // 二进制
let octalLiteral: number = 0o744;    // 八进制
let decLiteral: number = 6;    // 十进制
let hexLiteral: number = 0xf00d;    // 十六进制
```



### 字符串类型

- 关键字：string
- 描述：

```ts
let name: string = "Hello World";
let years: number = 5;
let words: string = `您好，今年是 ${ name } 发布 ${ years + 1} 周年`;
```



### 布尔类型

最基本的数据类型就是简单的true/false值，在JavaScript和TypeScript里叫做`boolean`（其它语言中也一样）。

- 关键字：boolean
- 描述：

```ts
let flag: boolean = true;
```



### 数组类型

声明变量为数组。

```ts
// 在元素类型后面加上[]
let arr: number[] = [1, 2];

// 或者使用数组泛型
let arr: Array<number> = [1, 2];
```



TypeScript 声明数组的语法格式如下所示：

```ts
// 声明后初始化
var array_name[:datatype];        //声明 
array_name = [val1,val2,valn..]   //初始化
// 或者直接在声明时初始化
var array_name[:datatype] = [val1,val2…valn]
```



如果数组声明时未设置类型，则会被认为是 any 类型，在初始化时根据第一个元素的类型来推断数组的类型。



### 元组

- 描述：元组类型用来表示已知元素数量和类型的数组，各元素的类型不必相同，对应位置的类型需要相同。

```ts
let x: [string, number];
x = ['Hello World', 1];    // 运行正常
x = [1, 'Hello World'];    // 报错
console.log(x[0]);    // 输出 Hello World
```



### 枚举

- 关键字：enum
- 描述：枚举类型用于定义数值集合。

```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Blue;
console.log(c);    // 输出 2
```



#### 枚举对象赋值

默认情况下，从0开始为元素编号，也可以手动的指定成员的数值。 

```ts
const getValue = () => {
  return 0
}

enum List {
  A = getValue(),
  B = 2,  	// 此处必须要初始化值，不然编译不通过
  C			// C的值是在前一个对象的基础上 + 1
}
console.log(List.A) // 0
console.log(List.B) // 2
console.log(List.C) // 3
```



或者全部都采用手动赋值；

```ts
enum Color {Red = 1, Green = 2, Blue = 4}
```



枚举类型提供的一个便利是可以由枚举的值得到它的名字。

 ```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

console.log(colorName);  // 显示'Green'因为上面代码里它的值是2
 ```



### void

- 关键字：void
- 描述：用于标识方法返回值的类型，表示该方法没有返回值。

```ts
function hello(): void {
    alert("Hello World");
}
```



### null

- 关键字：null
- 描述：表示对象值缺失。



在 JavaScript 中 null 表示 "什么都没有"。null是一个只有一个值的特殊类型。表示一个空对象引用。用 typeof 检测 null 返回是 object。



### undefined

- 关键字：undefined
- 描述： 用于初始化变量为一个未定义的值



在 JavaScript 中, undefined 是一个没有设置值的变量。typeof 一个没有值的变量会返回 undefined。

Null 和 Undefined 是其他任何类型（包括 void）的子类型，可以赋值给其它类型，如数字类型，此时赋值后的类型会变成 null 或 undefined。而在TypeScript中启用严格的空校验（--strictNullChecks）特性，就可以使得null 和 undefined 只能被赋值给 void 或本身对应的类型，示例代码如下：



```ts
// 启用 --strictNullChecks
let x: number;
x = 1; 			// 编译正确
x = undefined;  // 编译错误
x = null;    	// 编译错误
```



上面的例子中变量 x 只能是数字类型。如果一个类型可能出现 null 或 undefined， 可以用 | 来支持多种类型，示例代码如下：

```ts
// 启用 --strictNullChecks
let x: number | null | undefined;
x = 1; 			// 编译正确
x = undefined;  // 编译正确
x = null;    	// 编译正确
```



### never

- 关键字：never
- 描述：never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。



never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。这意味着声明为 never 类型的变量只能被 never 类型所赋值，在函数中它通常表现为抛出异常或无法执行到终止点（例如无限循环），示例代码如下：

```ts
let x: never;
let y: number;

// 编译错误，数字类型不能转为 never 类型
x = 123;

// 运行正确，never 类型可以赋值给 never类型
x = (()=>{ throw new Error('exception')})();

// 运行正确，never 类型可以赋值给 数字类型
y = (()=>{ throw new Error('exception')})();

// 返回值为 never 的函数可以是抛出异常的情况
function error(message: string): never {
    throw new Error(message);
}

// 返回值为 never 的函数可以是无法被执行到的终止点的情况
function loop(): never {
    while (true) {}
}
```



## 变量声明

变量是一种占位符，用于引用计算机内存地址，可以把变量看做存储数据的容器。TypeScript 变量的命名规则：

- 变量名称可以包含数字和字母。
- 除了下划线 **_** 和美元 **$** 符号外，不能包含其他特殊字符，包括空格。
- 变量名不能以数字开头。



变量使用前必须先声明，使用 var 来声明变量。可以使用以下四种方式来声明变量：



- 声明变量的类型及初始值：var [变量名] : [类型] = 值;

```ts
var uname:string = "Hello World";
```



- 声明变量的类型，但没有初始值，变量值会设置为 undefined：var [变量名] : [类型];

```ts
var uname:string;
```



- 声明变量并初始值，但不设置类型，该变量可以是任意类型：var [变量名] = 值;

```ts
var uname = "Hello World";
```



- 声明变量没有设置类型和初始值，类型可以是任意类型，默认初始值为 undefined：var [变量名];

```ts
var uname;
```



TypeScript 遵循强类型，如果将不同的类型赋值给变量会编译错误，如下实例：

```ts
var num:number = "hello"     // 这个代码会编译错误
```



### 类型断言

类型断言（Type Assertion）可以用来手动指定一个值的类型，即允许变量从一种类型更改为另一种类型。语法格式：`<类型>值` 或 `值 as 类型`。

```ts
var str = '1' 
var str2:number = <number> <any> str   //str、str2 是 string 类型
console.log(str2)
```



当 S 类型是 T 类型的子集，或者 T 类型是 S 类型的子集时，S 能被成功断言成 T。这是为了在进行类型断言时提供额外的安全性，完全毫无根据的断言是危险的，如果想这么做，可以使用 any。

它之所以不被称为**类型转换**，是因为转换通常意味着某种运行时的支持。但是类型断言纯粹是一个编译时语法，同时，它也是一种为编译器提供关于如何分析代码的方法。



### 类型推断

当类型没有给出时，TypeScript 编译器利用类型推断来推断类型。如果由于缺乏声明而不能推断出类型，那么它的类型被视作默认的动态 any 类型。



```ts
var num = 2;    // 类型推断为 number
console.log("num 变量的值为 "+num); 
num = "12";    // 编译错误
console.log(num);
```

- 第一行代码声明了变量 num 并=设置初始值为 2。 注意变量声明没有指定类型。因此，程序使用类型推断来确定变量的数据类型，第一次赋值为 2，**num** 设置为 number 类型。
- 第三行代码，当再次为变量设置字符串类型的值时，这时编译会错误。因为变量已经设置为了 number 类型。



### 变量作用域

变量作用域指定了变量定义的位置，程序中变量的可用性由变量作用域决定。TypeScript 有以下几种作用域：

- **全局作用域** − 全局变量定义在程序结构的外部，它可以在代码的任何位置使用。
- **类作用域** − 这个变量也可以称为 **字段**。类变量声明在一个类里头，但在类的方法外面。 该变量可以通过类的对象来访问。类变量也可以是静态的，静态的变量可以通过类名直接访问。
- **局部作用域** − 局部变量，局部变量只能在声明它的一个代码块（如：方法）中使用。



```ts
var global_num = 12          // 全局变量
class Numbers { 
   num_val = 13;             // 实例变量
   static sval = 10;         // 静态变量
   
   storeNum():void { 
      var local_num = 14;    // 局部变量
   } 
} 
console.log("全局变量为: "+global_num)  
console.log(Numbers.sval)   // 静态变量
var obj = new Numbers(); 
console.log("实例变量: "+obj.num_val)
```



执行以上 JavaScript 代码，输出结果为：

```
全局变量为: 12
10
实例变量: 13
```



如果我们在方法外部调用局部变量 local_num，会报错：`error TS2322: Could not find symbol 'local_num'.`。



## 运算符

运算符用于执行程序代码运算，会针对一个以上操作数项目来进行运算。TypeScript 主要包含以下几种运算：

- 算术运算符
- 逻辑运算符
- 关系运算符
- 按位运算符
- 赋值运算符
- 三元/条件运算符
- 字符串运算符
- 类型运算符



### 算术运算符

| 运算符 | 描述         | 例子  | x 运算结果 | y 运算结果 |
| :----- | :----------- | :---- | :--------- | :--------- |
| +      | 加法         | x=y+2 | 7          | 5          |
| -      | 减法         | x=y-2 | 3          | 5          |
| *      | 乘法         | x=y*2 | 10         | 5          |
| /      | 除法         | x=y/2 | 2.5        | 5          |
| %      | 取模（余数） | x=y%2 | 1          | 5          |
| ++     | 自增         | x=++y | 6          | 6          |
| x=y++  | 5            | 6     |            |            |
| --     | 自减         | x=--y | 4          | 4          |
| x=y--  | 5            | 4     |            |            |



### 关系运算符

关系运算符用于计算结果是否为 true 或者 false。

| 运算符 | 描述       | 比较 | 返回值  |
| :----- | :--------- | :--- | :------ |
| ==     | 等于       | x==8 | *false* |
| x==5   | *true*     |      |         |
| !=     | 不等于     | x!=8 | *true*  |
| >      | 大于       | x>8  | *false* |
| <      | 小于       | x<8  | *true*  |
| >=     | 大于或等于 | x>=8 | *false* |
| <=     | 小于或等于 | x<=8 | *true*  |



### 逻辑运算符

逻辑运算符用于测定变量或值之间的逻辑。

| 运算符 | 描述 | 例子                       |
| :----- | :--- | :------------------------- |
| &&     | and  | (x < 10 && y > 1) 为 true  |
| \|\|   | or   | (x=\=5 \|\| y==5) 为 false |
| !      | not  | !(x==y) 为 true            |



#### 短路运算符

短路运算符 && 与 || 运算符可用于组合表达式。 && 运算符只有在左右两个表达式都为 true 时才返回 true。|| 运算符只要其中一个表达式为 true ，则该组合表达式就会返回 true。



### 按位运算符

位操作是程序设计中对位模式按位或二进制数的一元和二元操作。

| <span style="display:inline-block;width:50px">运算符 </span> | 描述                                                         | 例子        | 类似于       | 结果 | <span style="display:inline-block;width:50px">十进制</span> |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :---------- | :----------- | :--- | :---------------------------------------------------------- |
| &                                                            | AND，按位与处理两个长度相同的二进制数，两个相应的二进位都为 1，该位的结果值才为 1，否则为 0。 | x = 5 & 1   | 0101 & 0001  | 0001 | 1                                                           |
| \|                                                           | OR，按位或处理两个长度相同的二进制数，两个相应的二进位中只要有一个为 1，该位的结果值为 1。 | x = 5 \| 1  | 0101 \| 0001 | 0101 | 5                                                           |
| ~                                                            | 取反，取反是一元运算符，对一个二进制数的每一位执行逻辑反操作。使数字 1 成为 0，0 成为 1。 | x = ~ 5     | ~0101        | 1010 | -6                                                          |
| ^                                                            | 异或，按位异或运算，对等长二进制模式按位或二进制数的每一位执行逻辑异按位或操作。操作的结果是如果某位不同则该位为 1，否则该位为 0。 | x = 5 ^ 1   | 0101 ^ 0001  | 0100 | 4                                                           |
| <<                                                           | 左移，把 << 左边的运算数的各二进位全部左移若干位，由 << 右边的数指定移动的位数，高位丢弃，低位补 0。 | x = 5 << 1  | 0101 << 1    | 1010 | 10                                                          |
| >>                                                           | 右移，把 >> 左边的运算数的各二进位全部右移若干位，>> 右边的数指定移动的位数。 | x = 5 >> 1  | 0101 >> 1    | 0010 | 2                                                           |
| >>>                                                          | 无符号右移，与有符号右移位类似，除了左边一律使用0 补位。     | x = 2 >>> 1 | 0010 >>> 1   | 0001 | 1                                                           |



### 赋值运算符

赋值运算符用于给变量赋值。类似的逻辑运算符也可以与赋值运算符联合使用：<<=、>>=、>>>=、&=、|=与^=。

| 运算符                  | 例子   | 实例      | x 值   |
| :---------------------- | :----- | :-------- | :----- |
| = (赋值)                | x = y  | x = y     | x = 5  |
| += (先进行加运算后赋值) | x += y | x = x + y | x = 15 |
| -= (先进行减运算后赋值) | x -= y | x = x - y | x = 5  |
| *= (先进行乘运算后赋值) | x *= y | x = x * y | x = 50 |
| /= (先进行除运算后赋值) | x /= y | x = x / y | x = 2  |



### 三元/条件运算符

三元运算有 3 个操作数，并且需要判断布尔表达式的值。该运算符的主要是决定哪个值应该赋值给变量。

```
Test ? expr1 : expr2
```

- Test − 指定的条件语句
- expr1 − 如果条件语句 Test 返回 true 则返回该值
- expr2 − 如果条件语句 Test 返回 false 则返回该值



### 类型运算符

#### typeof 运算符

typeof 是一元运算符，返回操作数的数据类型。



#### instanceof

instanceof 运算符用于判断对象是否为指定的类型。



### 其他运算符

#### 负号运算符(-)

```ts
var x:number = 4 
var y = -x; 
console.log("x 值为: ",x);   // 输出结果 4 
console.log("y 值为: ",y);   // 输出结果 -4
```



#### 字符串运算符: 连接运算符 (+)

```ts
var msg:string = "Hello "+"World" 
console.log(msg);
```



## 条件语句

条件语句用于基于不同的条件来执行不同的动作。TypeScript 条件语句是通过一条或多条语句的执行结果（True 或 False）来决定执行的代码块。

通常在写代码时，总是需要为不同的决定来执行不同的动作，在代码中使用条件语句来完成该任务。在 TypeScript 中可使用以下条件语句：

- **if 语句** - 只有当指定条件为 true 时，使用该语句来执行代码
- **if...else 语句** - 当条件为 true 时执行代码，当条件为 false 时执行其他代码
- **if...else if....else 语句**- 使用该语句来选择多个代码块之一来执行
- **switch...case 语句** - 使用该语句来选择多个代码块之一来执行



### if语句

if 语句由一个布尔表达式后跟一个或多个语句组成。

```ts
if(boolean_expression){
    # 在布尔表达式 boolean_expression 为 true 执行
}
```



如果布尔表达式 boolean_expression为 true，则 if 语句内的代码块将被执行。如果布尔表达式为 false，则 if 语句结束后的第一组代码（闭括号后）将被执行。



### if...else语句

一个 if 语句后可跟一个可选的 else 语句，else 语句在布尔表达式为 false 时执行。

```ts
if(boolean_expression){
   # 在布尔表达式 boolean_expression 为 true 执行
}else{
   # 在布尔表达式 boolean_expression 为 false 执行
}
```



如果布尔表达式 boolean_expression 为 true，则执行 if 块内的代码。如果布尔表达式为 false，则执行 else 块内的代码。



### if...else if....else语句

if...else if....else 语句在执行多个判断条件的时候很有用。

```ts
if(boolean_expression 1) {
    // 在布尔表达式 boolean_expression 1 为 true 执行
} else if( boolean_expression 2) {
    // 在布尔表达式 boolean_expression 2 为 true 执行
} else if( boolean_expression 3) {
    # 在布尔表达式 boolean_expression 3 为 true 执行
} else {
    # 布尔表达式的条件都为 false 时执行
}
```



需要注意以下几点：

- 一个 **if** 判断语句可以有 0 或 1 个 **else** 语句，她必需在 **else..if** 语句后面。
- 一个 **if** 判断语句可以有 0 或多个 **else..if**，这些语句必需在 **else** 之前。
- 一旦执行了 **else..if** 内的代码，后面的 **else..if** 或 **else** 将不再执行。



### switch...case语句

一个 **switch** 语句允许测试一个变量等于多个值时的情况。每个值称为一个 case，且被测试的变量会对每个 **switch case** 进行检查。**switch** 语句的语法：

```ts
switch(expression){
    case constant-expression  :
       statement(s);
       break; /* 可选的 */
    case constant-expression  :
       statement(s);
       break; /* 可选的 */
  
    /* 您可以有任意数量的 case 语句 */
    default : /* 可选的 */
       statement(s);
}
```



**switch** 语句必须遵循下面的规则：

- **switch** 语句中的 **expression** 是一个要被比较的表达式，可以是任何类型，包括基本数据类型（如 number、string、boolean）、对象类型（如 object、Array、Map）以及自定义类型（如 class、interface、enum）等。
- 在一个 switch 中可以有任意数量的 case 语句。每个 case 后跟一个要比较的值和一个冒号。
- case 的 **constant-expression** 必须与 switch 中的变量 expression 具有相同或兼容的数据类型。
- 当被测试的变量等于 case 中的常量时，case 后跟的语句将被执行，直到遇到 **break** 语句为止。
- 当遇到 **break** 语句时，switch 终止，控制流将跳转到 switch 语句后的下一行。
- 不是每一个 case 都需要包含 **break**。如果 case 语句不包含 **break**，控制流将会 *继续* 后续的 case，直到遇到 break 为止。
- 一个 **switch** 语句可以有一个可选的 **default** case，出现在 switch 的结尾。default 关键字则表示当表达式的值与所有 case 值都不匹配时执行的代码块。default case 中的 **break** 语句不是必需的。



## 循环

循环语句允许多次执行一个语句或语句组。



### for循环

for 循环用于多次执行一个语句序列，简化管理循环变量的代码。语法格式如下所示：

```ts
for ( init; condition; increment ){
    statement(s);
}
```



下面是 for 循环的控制流程解析：

1. **init** 会首先被执行，且只会执行一次。这一步允许声明并初始化任何循环控制变量。也可以不在这里写任何语句，只要有一个分号出现即可。
2. 接下来，会判断 **condition**。如果为 true，则执行循环主体。如果为 false，则不执行循环主体，且控制流会跳转到紧接着 for 循环的下一条语句。
3. 在执行完 for 循环主体后，控制流会跳回上面的 **increment** 语句。该语句允许更新循环控制变量。该语句可以留空，只要在条件后有一个分号出现即可。
4. 条件再次被判断。如果为 true，则执行循环，这个过程会不断重复（循环主体，然后增加步值，再然后重新判断条件）。在条件变为 false 时，for 循环终止。

在这里，statement(s) 可以是一个单独的语句，也可以是几个语句组成的代码块。condition 可以是任意的表达式，当条件为 true 时执行循环，当条件为 false 时，退出循环。



### for...in循环

for...in 语句用于一组值的集合或列表进行迭代输出。语法格式如下所示：

```ts
for (var val in list) { // val 需要为 string 或 any 类型。
    // 语句 
}
```



### for…of 、forEach、every 和 some 循环

此外，TypeScript 还支持 for…of 、forEach、every 和 some 循环。

for...of 语句创建一个循环来迭代可迭代的对象。在 ES6 中引入的 for...of 循环，以替代 for...in 和 forEach() ，并支持新的迭代协议。for...of 允许你遍历 Arrays（数组）, Strings（字符串）, Maps（映射）, Sets（集合）等可迭代的数据结构等。

```ts
let someArray = [1, "string", false];
 
for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```



forEach、every 和 some 是 JavaScript 的循环语法，TypeScript 作为 JavaScript 的语法超集，当然默认也是支持的。因为 forEach 在 iteration 中是无法返回的，所以可以使用 every 和 some 来取代 forEach。

```ts
let list = [4, 5, 6];
list.forEach((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
});

list.every((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
    return true; // 返回true则继续循环，false则退出循环
});
```



### while 循环

while 语句在给定条件为 true 时，重复执行语句或语句组。循环主体执行之前会先测试条件。语法格式如下所示：

```ts
while(condition) {
   statement(s);
}
```



在这里，statement(s) 可以是一个单独的语句，也可以是几个语句组成的代码块。condition 可以是任意的表达式，当条件为 true 时执行循环。 当条件为 false 时，程序流将退出循环。

while 循环的关键点是循环可能一次都不会执行。当条件为 false 时，会跳过循环主体，直接执行紧接着 while 循环的下一条语句。



### do...while 循环

不像 **for** 和 **while** 循环，它们是在循环头部测试循环条件。**do...while** 循环是在循环的尾部检查它的条件。语法格式如下所示：

```ts
do {
   statement(s);
} while( condition );
```



条件表达式出现在循环的尾部，所以循环中的 statement(s) 会在条件被测试之前至少执行一次。如果条件为 true，控制流会跳转回上面的 do，然后重新执行循环中的 statement(s)。这个过程会不断重复，直到给定条件变为 false 为止。



### break 语句

**break** 语句有以下两种用法：

1. 当 **break** 语句出现在一个循环内时，循环会立即终止，且程序流将继续执行紧接着循环的下一条语句。
2. 它可用于终止 **switch** 语句中的一个 case。



如果使用的是嵌套循环（即一个循环内嵌套另一个循环），break 语句会停止执行最内层的循环，然后开始执行该块之后的下一行代码。

```ts
var i:number = 1 
while(i<=10) { 
    if (i % 5 == 0) {   
        console.log ("在 1~10 之间第一个被 5 整除的数为 : "+i) 
        break     // 找到一个后退出循环
    } 
    i++ 
}  // 输出 5 然后程序执行结束
```



### continue 语句

**continue** 语句有点像 **break** 语句。但它不是强制终止，continue 会跳过当前循环中的代码，强迫开始下一次循环。对于 **for** 循环，**continue** 语句执行后自增语句仍然会执行。对于 **while** 和 **do...while** 循环，**continue** 语句重新执行条件判断语句。

```ts
var num:number = 0
var count:number = 0;
 
for(num=0;num<=20;num++) {
    if (num % 2==0) {
        continue
    }
    count++
}
console.log ("0 ~20 之间的奇数个数为: "+count)    //输出10个偶数
```



### 无限循环

无限循环就是一直在运行不会停止的循环。 for 和 while 循环都可以创建无限循环。

```ts
for(;;) { 
   // 语句
}

while(true) { 
   console.log("这段代码会不停的执行") 
}
```



# TypeScript 函数

函数是一组一起执行一个任务的语句。可以把代码划分到不同的函数中。如何划分代码到不同的函数中是由您来决定的，但在逻辑上，划分通常是根据每个函数执行一个特定的任务来进行的。函数声明告诉编译器函数的名称、返回类型和参数。函数定义提供了函数的实际主体。



## 函数定义和调用

函数就是包裹在花括号中的代码块，前面使用了关键词 function。语法格式如下所示：

```ts
function function_name(){
    // 执行代码
}
```



函数只有通过调用才可以执行函数内的代码。语法格式如下所示：

```ts
function_name()
```



```ts
function test() {   // 函数定义
    console.log("调用函数") 
} 
test()              // 调用函数
```



## 函数返回值

使用 return 语句，函数会停止执行，并返回指定的值。语法格式如下所示：

```ts
function function_name():return_type { 
    // 语句
    return value; 
}
```



- return_type 是返回值的类型。
- return 关键词后跟着要返回的结果。
- 一般情况下，一个函数只有一个 return 语句。
- 返回值的类型需要与函数定义的返回类型(return_type)一致。



## 带参函数

在调用函数时，您可以向其传递值，这些值被称为参数。这些参数可以在函数中使用。可以向函数发送多个参数，每个参数使用逗号 **,** 分隔。语法格式如下所示：

```ts
function func_name( param1 [:datatype], param2 [:datatype]) {   
}
```



### 可选参数

在 TypeScript 函数里，如果定义了参数，则必须传入这些参数，除非将这些参数设置为可选，可选参数使用问号标识 ？。

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}
 
let result1 = buildName("Bob");  // 正确
let result2 = buildName("Bob", "Adams", "Sr.");  // 错误，参数太多了
let result3 = buildName("Bob", "Adams");  // 正确
```



可选参数必须跟在必需参数后面。 如果上例想让 firstName 是可选的，lastName 必选，那么就要调整它们的位置，把 firstName 放在后面。如果都是可选参数就没关系。



### 默认参数

可以设置参数的默认值，这样在调用函数的时候，如果不传入该参数的值，则使用默认参数，语法格式为：

```ts
function function_name(param1[:type],param2[:type] = default_value) { 
    // ...
}
```



**注意：参数不能同时设置为可选和默认。**

```ts
function calculate_discount(price:number,rate:number = 0.50) { 
    var discount = price * rate; 
    console.log("计算结果: ",discount); 
} 
calculate_discount(1000);
calculate_discount(1000, 0.30);
```



### 剩余参数

不知道要向函数传入多少个参数时就可以使用剩余参数来定义。剩余参数语法允许将一个不确定数量的参数作为一个数组传入。

```ts
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}
  
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```



## 匿名函数

匿名函数是一个没有函数名的函数。匿名函数在程序运行时动态声明，除了没有函数名外，其他的与标准函数一样。可以将匿名函数赋值给一个变量，这种表达式就成为函数表达式。语法格式如下：

```ts
var res = function( [arguments] ) { 
    // ... 
}
```



### 无参匿名函数

```ts
var msg = function() { 
    return "hello world";  
} 
console.log(msg());
```



### 带参匿名函数

匿名函数和普通函数一样可以使用可选参数、默认参数和剩余参数语法。

```ts
var res = function(a:number,b:number) { 
    return a*b;  
}; 
console.log(res(12,2));
```



### 匿名函数自调用

匿名函数自调用在函数后使用 () 即可：

```ts
(function () { 
    var x = "Hello!!";   
    console.log(x)     
 })();
```



## 构造函数

TypeScript 也支持使用 JavaScript 内置的构造函数 Function() 来定义函数。语法格式如下：

```
var res = new Function ([arg1[, arg2[, ...argN]],] functionBody)
```



参数说明：

- **arg1, arg2, ... argN**：参数列表。
- **functionBody**：一个含有包括函数定义的 JavaScript 语句的字符串。



```ts
var myFunction = new Function("a", "b", "return a * b"); 
var x = myFunction(4, 3); 
console.log(x);
```



## 递归函数

递归函数即在函数内调用函数本身。

```ts
function factorial(number) {
    if (number <= 0) {         // 停止执行
        return 1; 
    } else {     
        return (number * factorial(number - 1));     // 调用自身
    } 
}; 
console.log(factorial(6));      // 输出 720
```



## Lambda函数

Lambda 函数也称之为箭头函数，箭头函数表达式的语法比函数表达式更短。函数只有一行语句：

```ts
([param1, param2,…param n]) => statement;
```



```ts
var foo = (x:number)=>10 + x 
console.log(foo(100))      //输出结果为 110
```



可以不指定函数的参数类型，通过函数内来推断参数类型：

```ts
var func = (x)=> { 
    if(typeof x=="number") { 
        console.log(x+" 是一个数字") 
    } else if(typeof x=="string") { 
        console.log(x+" 是一个字符串") 
    }  
} 
func(12) 
func("Tom")
```



单个参数时 **()** 是可选的：

```ts
var display = x => { 
    console.log("输出为 "+x) 
} 
display(12)
```



无参数时可以设置空括号：

```ts
var display = ()=> { 
    console.log("Function invoked"); 
} 
display();
```



## 函数重载

重载是方法名字相同，而参数不同，返回类型可以相同也可以不同。每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。



参数类型不同：

```ts
function disp(string):void; 
function disp(number):void;
```



参数数量不同：

```ts
function disp(n1:number):void; 
function disp(x:number,y:number):void;
```



参数类型顺序不同：

```ts
function disp(n1:number,s1:string):void; 
function disp(s:string,n:number):void;
```



如果参数类型不同，则参数类型应设置为 **any**。参数数量不同你可以将不同的参数设置为可选。



# TypeScript 对象

对象是包含一组键值对的实例。 值可以是标量、函数、数组、对象等，如下实例：

```ts
var object_name = { 
    key1: "value1", // 标量
    key2: "value",  
    key3: function() {
        // 函数
    }, 
    key4:["content1", "content2"] //集合
}
```



以上对象包含了标量，函数，集合(数组或元组)。

```ts
var sites = { 
   site1:"Runoob", 
   site2:"Google" 
}; 
// 访问对象的值
console.log(sites.site1) 
console.log(sites.site2)
```



## 类型模板

在 JavaScript 定义了一个对象：

```ts
var sites = { 
   site1:"Runoob", 
   site2:"Google" 
};
```



这时如果想在对象中添加方法，可以做以下修改：

```ts
sites.sayHello = function(){ return "hello";}
```



如果在 TypeScript 中使用以上方式则会出现编译错误，因为Typescript 中的对象必须是特定类型的实例。

```ts
var sites = {
    site1: "Runoob",
    site2: "Google",
    sayHello: function () { } // 类型模板
};
sites.sayHello = function () {
    console.log("hello " + sites.site1);
};
sites.sayHello();
```



## 作为函数参数

对象也可以作为一个参数传递给函数，如下实例：

```ts
var sites = { 
    site1:"Runoob", 
    site2:"Google",
}; 
var invokesites = function(obj: { site1:string, site2 :string }) { 
    console.log("site1 :"+obj.site1) 
    console.log("site2 :"+obj.site2) 
} 
invokesites(sites)
```



## 鸭子类型

鸭子类型（英语：duck typing）是动态类型的一种风格，是多态(polymorphism)的一种形式。在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由"当前方法和属性的集合"决定。



> 可以这样表述：
>
> "当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。"



在鸭子类型中，关注点在于对象的行为能做什么，而不是关注对象所属的类型。例如，在不使用鸭子类型的语言中，可以编写一个函数，它接受一个类型为"鸭子"的对象，并调用它的"走"和"叫"方法。在使用鸭子类型的语言中，这样的一个函数可以接受一个任意类型的对象，并调用它的"走"和"叫"方法。如果这些需要被调用的方法不存在，那么将引发一个运行时错误。任何拥有这样的正确的"走"和"叫"方法的对象都可被函数接受的这种行为引出了以上表述，这种决定类型的方式因此得名。

```ts
interface IPoint { 
    x:number 
    y:number 
} 
function addPoints(p1:IPoint,p2:IPoint):IPoint { 
    var x = p1.x + p2.x 
    var y = p1.y + p2.y 
    return {x:x,y:y} 
} 
 
// 正确
var newPoint = addPoints({x:3,y:4},{x:5,y:1})  
 
// 错误 
var newPoint2 = addPoints({x:1},{x:4,y:3})
```



# TypeScript 联合类型

联合类型（Union Types）可以通过管道(|)将变量设置多种类型，赋值时可以根据设置的类型来赋值。**注意**：只能赋值指定的类型，如果赋值其它类型就会报错。创建联合类型的语法格式如下：

```ts
Type1|Type2|Type3 
```



声明一个联合类型：

```ts
var val:string|number 
val = 12 
console.log("数字为 "+ val) 
val = "Runoob" 
console.log("字符串为 " + val)
```



也可以将联合类型作为函数参数使用：

```ts
function disp(name:string|string[]) { 
    if(typeof name == "string") { 
        console.log(name) 
    } else { 
        var i; 
        for(i = 0;i<name.length;i++) { 
        console.log(name[i])
        } 
    } 
} 
disp("Runoob") 
console.log("输出数组....") 
disp(["Runoob","Google","Taobao","Facebook"])
```



也可以将数组声明为联合类型：

```ts
var arr:number[]|string[]; 
var i:number; 
arr = [1,2,4] 
console.log("**数字数组**")  
 
for(i = 0;i<arr.length;i++) { 
   console.log(arr[i]) 
}  
 
arr = ["Runoob","Google","Taobao"] 
console.log("**字符串数组**")  
 
for(i = 0;i<arr.length;i++) { 
   console.log(arr[i]) 
}
```



# TypeScript 类

TypeScript 是面向对象的 JavaScript。类描述了所创建的对象共同的属性和方法。TypeScript 支持面向对象的所有特性，比如 类、接口等。TypeScript 类定义方式如下：

```ts
class class_name { 
    // 类作用域
}
```



定义类的关键字为 class，后面紧跟类名，类可以包含以下几个模块（类的数据成员）：

- **字段** − 字段是类里面声明的变量。字段表示对象的有关数据。
- **构造函数** − 类实例化时调用，可以为类的对象分配内存。
- **方法** − 方法为对象要执行的操作。



创建一个 Person 类：

```ts
class Person {
}
```



## 创建类的数据成员

以下实例声明了类 Car，包含字段为 engine，构造函数在类实例化后初始化字段 engine。this 关键字表示当前类实例化的对象。注意构造函数的参数名与字段名相同，this.engine 表示类的字段。此外也在类中定义了一个方法 disp()。

```ts
class Car { 
    // 字段 
    engine:string; 
 
    // 构造函数 
    constructor(engine:string) { 
        this.engine = engine 
    }  
 
    // 方法 
    disp():void { 
        console.log("发动机为 :   "+this.engine) 
    } 
}
```



## 创建实例化对象

使用 new 关键字来实例化类的对象，语法格式如下：

```ts
var object_name = new class_name([ arguments ])
```



类实例化时会调用构造函数，例如：

```ts
var obj = new Car("Engine 1")
```



类中的字段属性和方法可以使用 **.** 号来访问：

```ts
// 访问属性
obj.field_name 

// 访问方法
obj.function_name()
```



以下实例创建来一个 Car 类，然后通过关键字 new 来创建一个对象并访问属性和方法：

```ts
class Car { 
   // 字段
   engine:string; 
   
   // 构造函数
   constructor(engine:string) { 
      this.engine = engine 
   }  
   
   // 方法
   disp():void { 
      console.log("函数中显示发动机型号  :   "+this.engine) 
   } 
} 
 
// 创建一个对象
var obj = new Car("XXSY1")
 
// 访问字段
console.log("读取发动机型号 :  "+obj.engine)  
 
// 访问方法
obj.disp()
```



## 类的继承

TypeScript 支持继承类，即可以在创建类的时候继承一个已存在的类，这个已存在的类称为父类，继承它的类称为子类。类继承使用关键字 **extends**，子类除了不能继承父类的私有成员(方法和属性)和构造函数，其他的都可以继承。

TypeScript 一次只能继承一个类，不支持继承多个类，但 TypeScript 支持多重继承（A 继承 B，B 继承 C）。语法格式如下：

```ts
class child_class_name extends parent_class_name
```



实例中创建了 Shape 类，Circle 类继承了 Shape 类，Circle 类可以直接使用 Area 属性：

```ts
class Shape { 
   Area:number 
   
   constructor(a:number) { 
      this.Area = a 
   } 
} 
 
class Circle extends Shape { 
   disp():void { 
      console.log("圆的面积:  "+this.Area) 
   } 
}
  
var obj = new Circle(223); 
obj.disp()
```



## 方法重写

类继承后，子类可以对父类的方法重新定义，这个过程称之为方法的重写。其中 super 关键字是对父类的直接引用，该关键字可以引用父类的属性和方法。

```ts
class PrinterClass { 
   doPrint():void {
      console.log("父类的 doPrint() 方法。") 
   } 
} 
 
class StringPrinter extends PrinterClass { 
   doPrint():void { 
      super.doPrint() // 调用父类的函数
      console.log("子类的 doPrint()方法。")
   } 
}
```



## static 关键字

static 关键字用于定义类的数据成员（属性和方法）为静态的，静态成员可以直接通过类名调用。

```ts
class StaticMem {  
   static num:number; 
   
   static disp():void { 
      console.log("num 值为 "+ StaticMem.num) 
   } 
} 
 
StaticMem.num = 12     // 初始化静态变量
StaticMem.disp()       // 调用静态方法
```



## instanceof 运算符

instanceof 运算符用于判断对象是否是指定的类型，如果是返回 true，否则返回 false。

```ts
class Person{ } 
var obj = new Person() 
var isPerson = obj instanceof Person; 
console.log("obj 对象是 Person 类实例化来的吗？ " + isPerson);
```



## 访问控制修饰符

TypeScript 中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。TypeScript 支持 3 种不同的访问权限。

- **public（默认）** : 公有，可以在任何地方被访问。
- **protected** : 受保护，可以被其自身以及其子类访问。
- **private** : 私有，只能被其定义所在的类访问。

以下实例定义了两个变量 str1 和 str2，str1 为 public，str2 为 private，实例化后可以访问 str1，如果要访问 str2 则会编译错误。

```ts
class Encapsulate { 
   str1:string = "hello" 
   private str2:string = "world" 
}
 
var obj = new Encapsulate() 
console.log(obj.str1)     // 可访问 
console.log(obj.str2)   // 编译错误， str2 是私有的
```



## 类和接口

类可以实现接口，使用关键字 implements，并将 interest 字段作为类的属性使用。以下实例中 AgriLoan 类实现了 ILoan 接口：

```ts
interface ILoan { 
   interest:number 
} 
 
class AgriLoan implements ILoan { 
   interest:number 
   rebate:number 
   
   constructor(interest:number,rebate:number) { 
      this.interest = interest 
      this.rebate = rebate 
   } 
} 
 
var obj = new AgriLoan(10,1) 
console.log("利润为 : "+obj.interest+"，抽成为 : "+obj.rebate )
```



# TypeScript 接口

接口是一系列抽象方法的声明，是一些方法特征的集合，这些方法都应该是抽象的，需要由具体的类去实现，然后第三方就可以通过这组抽象方法调用，让具体的类执行具体的方法。TypeScript 接口定义如下：

```ts
interface interface_name { 
}
```



以下实例中定义了一个接口 IPerson，接着定义了一个变量 customer，它的类型是 IPerson。customer 实现了接口 IPerson 的属性和方法。

```ts
interface IPerson { 
    firstName:string, 
    lastName:string, 
    sayHi: ()=>string 
} 
 
var customer:IPerson = { 
    firstName:"Tom",
    lastName:"Hanks", 
    sayHi: ():string =>{return "Hi there"} 
} 
 
console.log("Customer 对象 ") 
console.log(customer.firstName) 
console.log(customer.lastName) 
console.log(customer.sayHi())  
```



## 联合类型和接口

以下实例演示了如何在接口中使用联合类型：

```ts
interface RunOptions { 
    program:string; 
    commandline:string[]|string|(()=>string); 
} 
 
// commandline 是字符串
var options:RunOptions = {program:"test1",commandline:"Hello"}; 
console.log(options.commandline)  
 
// commandline 是字符串数组
options = {program:"test1",commandline:["Hello","World"]}; 
console.log(options.commandline[0]); 
console.log(options.commandline[1]);  
 
// commandline 是一个函数表达式
options = {program:"test1",commandline:()=>{return "**Hello World**";}}; 
 
var fn:any = options.commandline; 
console.log(fn());
```



## 接口和数组

接口中可以将数组的索引值和元素设置为不同类型，索引值可以是数字或字符串。设置元素为字符串类型：

```ts
interface namelist { 
   [index:number]:string 
} 
 
// 类型一致，正确
var list2:namelist = ["Google","Runoob","Taobao"]
// 错误元素 1 不是 string 类型
// var list2:namelist = ["Runoob",1,"Taobao"]
```



```ts
interface ages { 
   [index:string]:number 
} 
 
var agelist:ages; 
 // 类型正确 
agelist["runoob"] = 15  
 
// 类型错误，输出  error TS2322: Type '"google"' is not assignable to type 'number'.
// agelist[2] = "google"
```



## 接口继承

接口继承就是说接口可以通过其他接口来扩展自己。Typescript 允许接口继承多个接口。继承使用关键字 **extends**。继承的各个接口使用逗号 **,** 分隔。



单接口继承语法格式：

```
Child_interface_name extends super_interface_name
```



多接口继承语法格式：

```
Child_interface_name extends super_interface1_name, super_interface2_name,…,super_interfaceN_name
```



### 单继承实例

```ts
interface Person { 
   age:number 
} 
 
interface Musician extends Person { 
   instrument:string 
} 
 
var drummer = <Musician>{}; 
drummer.age = 27 
drummer.instrument = "Drums" 
console.log("年龄:  "+drummer.age)
console.log("喜欢的乐器:  "+drummer.instrument)
```



### 多继承实例

```ts
interface IParent1 { 
    v1:number 
} 
 
interface IParent2 { 
    v2:number 
} 
 
interface Child extends IParent1, IParent2 { } 
var Iobj:Child = { v1:12, v2:23} 
console.log("value 1: "+Iobj.v1+" value 2: "+Iobj.v2)
```



# TypeScript 泛型

泛型（Generics）是一种编程语言特性，允许在定义函数、类、接口等时使用占位符来表示类型，而不是具体的类型。泛型是一种在编写可重用、灵活且类型安全的代码时非常有用的功能。

使用泛型的主要目的是为了处理不特定类型的数据，使得代码可以适用于多种数据类型而不失去类型检查。

**泛型的优势包括：**

- **代码重用：** 可以编写与特定类型无关的通用代码，提高代码的复用性。
- **类型安全：** 在编译时进行类型检查，避免在运行时出现类型错误。
- **抽象性：** 允许编写更抽象和通用的代码，适应不同的数据类型和数据结构。



## 泛型标识符

在泛型中，通常使用一些约定俗成的标识符，比如常见的 `T`（表示 Type）、`U`、`V` 等，但实际上你可以使用任何标识符。

**T**: 代表 "Type"，是最常见的泛型类型参数名。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```



**K, V**: 用于表示键（Key）和值（Value）的泛型类型参数。

```ts
interface KeyValuePair<K, V> {
    key: K;
    value: V;
}
```



**E**: 用于表示数组元素的泛型类型参数。

```ts
function printArray<E>(arr: E[]): void {
    arr.forEach(item => console.log(item));
}
```



**R**: 用于表示函数返回值的泛型类型参数。

```ts
function getResult<R>(value: R): R {
    return value;
}
```



**U, V**: 通常用于表示第二、第三个泛型类型参数。

```ts
function combine<U, V>(first: U, second: V): string {
    return `${first} ${second}`;
}
```



这些标识符是约定俗成的，实际上可以选择任何符合标识符规范的名称。关键是使得代码易读和易于理解，所以建议在泛型类型参数上使用描述性的名称，以便于理解其用途。



## 泛型函数

泛型函数（Generic Functions）指使用泛型来创建一个可以处理不同类型的函数：

```ts
function identity<T>(arg: T): T {
    return arg;
}

// 使用泛型函数
let result = identity<string>("Hello");
console.log(result); // 输出: Hello

let numberResult = identity<number>(42);
console.log(numberResult); // 输出: 42
```



 以上例子中，`identity` 是一个泛型函数，使用 `<T>` 表示泛型类型。它接受一个参数 `arg` 和返回值都是泛型类型 `T`。在使用时，可以通过尖括号 `<>` 明确指定泛型类型。第一个调用指定了 `string` 类型，第二个调用指定了 `number` 类型。



## 泛型接口

泛型接口（Generic Interfaces）指使用泛型来定义接口，使接口的成员能够使用任意类型：

```ts
// 基本语法
interface Pair<T, U> {
    first: T;
    second: U;
}

// 使用泛型接口
let pair: Pair<string, number> = { first: "hello", second: 42 };
console.log(pair); // 输出: { first: 'hello', second: 42 }
```



这里定义了一个泛型接口 `Pair`，它有两个类型参数 `T` 和 `U`。然后，使用这个泛型接口创建了一个对象 `pair`，其中 `first` 是字符串类型，`second` 是数字类型。



## 泛型类

泛型类（Generic Classes）指应用于类的实例变量和方法：

```ts
// 基本语法
class Box<T> {
    private value: T;

    constructor(value: T) {
        this.value = value;
    }

    getValue(): T {
        return this.value;
    }
}

// 使用泛型类
let stringBox = new Box<string>("TypeScript");
console.log(stringBox.getValue()); // 输出: TypeScript
```



在这个例子中，`Box` 是一个泛型类，使用 `<T>` 表示泛型类型。构造函数和方法都可以使用泛型类型 `T`。通过实例化 `Box<string>`，我们创建了一个存储字符串的 `Box` 实例，并通过 `getValue` 方法获取了存储的值。



## 泛型约束

泛型约束（Generic Constraints）可以用于限制泛型的类型范围。

```ts
// 基本语法
interface Lengthwise {
    length: number;
}

function logLength<T extends Lengthwise>(arg: T): void {
    console.log(arg.length);
}

// 正确的使用
logLength("hello"); // 输出: 5

// 错误的使用，因为数字没有 length 属性
logLength(42); // 错误
```



在这个例子中，定义了一个泛型函数 `logLength`，它接受一个类型为 `T` 的参数，但有一个约束条件，即 `T` 必须实现 `Lengthwise` 接口，该接口要求有 `length` 属性。因此，可以正确调用 `logLength("hello")`，但不能调用 `logLength(42)`，因为数字没有 `length` 属性。



## 泛型与默认值

可以给泛型设置默认值，使得在不指定类型参数时能够使用默认类型：

```ts
// 基本语法
function defaultValue<T = string>(arg: T): T {
    return arg;
}

// 使用带默认值的泛型函数
let result1 = defaultValue("hello"); // 推断为 string 类型
let result2 = defaultValue(42);      // 推断为 number 类型
```



这个例子展示了带有默认值的泛型函数。函数 `defaultValue` 接受一个泛型参数 `T`，并给它设置了默认类型为 `string`。在使用时，如果没有显式指定类型，会使用默认类型。在例子中，第一个调用中 `result1` 推断为 `string` 类型，第二个调用中 `result2` 推断为 `number` 类型。



# TypeScript 命名空间

命名空间一个最明确的目的就是解决重名问题。命名空间定义了标识符的可见范围，一个标识符可在多个命名空间中定义，它在不同命名空间中的含义是互不相干的。这样，在一个新的命名空间中可定义任何标识符，它们不会与任何已有的标识符发生冲突，因为已有的定义都处于其他命名空间中。

TypeScript 中命名空间使用 **namespace** 来定义，语法格式如下：

```ts
namespace SomeNameSpaceName { 
   export interface ISomeInterfaceName {      }  
   export class SomeClassName {      }  
}
```



以上定义了一个命名空间 SomeNameSpaceName，如果需要在外部可以调用 SomeNameSpaceName 中的类和接口，则需要在类和接口添加 **export** 关键字。要在另外一个命名空间调用语法格式为：

```ts
SomeNameSpaceName.SomeClassName;
```



如果一个命名空间在一个单独的 TypeScript 文件中，则应使用三斜杠 /// 引用它，语法格式如下：

```ts
/// <reference path = "SomeFileName.ts" />
```



以下实例演示了命名空间的使用，定义在不同文件中：

> IShape.ts

```ts
namespace Drawing { 
    export interface IShape { 
        draw(); 
    }
}
```



> Circle.ts

```ts
/// <reference path = "IShape.ts" /> 
namespace Drawing { 
    export class Circle implements IShape { 
        public draw() { 
            console.log("Circle is drawn"); 
        }  
    }
}
```



> Triangle.ts

```ts
/// <reference path = "IShape.ts" /> 
namespace Drawing { 
    export class Triangle implements IShape { 
        public draw() { 
            console.log("Triangle is drawn"); 
        } 
    } 
}
```



> TestShape.ts

```ts
/// <reference path = "IShape.ts" />   
/// <reference path = "Circle.ts" /> 
/// <reference path = "Triangle.ts" />  
function drawAllShapes(shape:Drawing.IShape) { 
    shape.draw(); 
} 
drawAllShapes(new Drawing.Circle());
drawAllShapes(new Drawing.Triangle());
```



## 嵌套命名空间

命名空间支持嵌套，即可以将命名空间定义在另外一个命名空间里头。

```ts
namespace namespace_name1 { 
    export namespace namespace_name2 {
        export class class_name {    } 
    } 
}
```



成员的访问使用点号 **.** 来实现，如下实例：

> Invoice.ts

```ts
namespace Runoob { 
   export namespace invoiceApp { 
      export class Invoice { 
         public calculateDiscount(price: number) { 
            return price * .40; 
         } 
      } 
   } 
}
```



> InvoiceTest.ts

```ts
/// <reference path = "Invoice.ts" />
var invoice = new Runoob.invoiceApp.Invoice(); 
console.log(invoice.calculateDiscount(500));
```



# TypeScript 声明文件

TypeScript 作为 JavaScript 的超集，在开发过程中不可避免要引用其他第三方的 JavaScript 的库。虽然通过直接引用可以调用库的类和方法，但是却无法使用TypeScript 诸如类型检查等特性功能。为了解决这个问题，需要将这些库里的函数和方法体去掉后只保留导出类型声明，而产生了一个描述 JavaScript 库和模块信息的声明文件。通过引用这个声明文件，就可以借用 TypeScript 的各种特性来使用库文件了。

假如想使用第三方库，比如 jQuery，通常这样获取一个 id 是 foo 的元素：

```ts
$('#foo');
// 或
jQuery('#foo');
```



但是在 TypeScript 中并不知道 $ 或 jQuery 是什么东西：

```ts
jQuery('#foo');

// index.ts(1,1): error TS2304: Cannot find name 'jQuery'.
```



这时需要使用 declare 关键字来定义它的类型，帮助 TypeScript 判断传入的参数类型对不对：

```ts
declare var jQuery: (selector: string) => any;

jQuery('#foo');
```



declare 定义的类型只会用于编译时的检查，编译结果中会被删除。上例的编译结果是：

```ts
jQuery('#foo');
```



## 声明文件

声明文件以 **.d.ts** 为后缀，例如：

```
runoob.d.ts
```



声明文件或模块的语法格式如下：

```ts
declare module Module_Name {
}
```



TypeScript 引入声明文件语法格式：

```ts
/// <reference path = " runoob.d.ts" />
```



很多流行的第三方库的声明文件不需要定义了，比如 jQuery 已经定义好了：[jQuery in DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/jquery/index.d.ts)。



# TypeScript 模块

TypeScript 模块的设计理念是可以更换的组织代码。

模块是在其自身的作用域里执行，并不是在全局作用域，这意味着定义在模块里面的变量、函数和类等在模块外部是不可见的，除非明确地使用 export 导出它们。类似地，我们必须通过 import 导入其他模块导出的变量、函数、类等。

两个模块之间的关系是通过在文件级别上使用 import 和 export 建立的。

模块使用模块加载器去导入其它的模块。 在运行时，模块加载器的作用是在执行此模块代码前去查找并执行这个模块的所有依赖。 大家最熟知的JavaScript模块加载器是服务于 Node.js 的 CommonJS 和服务于 Web 应用的 Require.js。此外还有有 SystemJs 和 Webpack。

模块导出使用关键字 **export** 关键字，语法格式如下：

```ts
// 文件名 : SomeInterface.ts 
export interface SomeInterface { 
   // 代码部分
}
```



要在另外一个文件使用该模块就需要使用 **import** 关键字来导入：

```ts
import someInterfaceRef = require("./SomeInterface");
```



示例如下：

> IShape.ts

```ts
/// <reference path = "IShape.ts" /> 
export interface IShape { 
   draw(); 
}
```



> Circle.ts

```ts
import shape = require("./IShape"); 
export class Circle implements shape.IShape { 
   public draw() { 
      console.log("Cirlce is drawn (external module)"); 
   } 
}
```



> Triangle.ts

```ts
import shape = require("./IShape"); 
export class Triangle implements shape.IShape { 
   public draw() { 
      console.log("Triangle is drawn (external module)"); 
   } 
}
```



> TestShape.ts

```ts
import shape = require("./IShape"); 
import circle = require("./Circle"); 
import triangle = require("./Triangle");  
 
function drawAllShapes(shapeToDraw: shape.IShape) {
   shapeToDraw.draw(); 
} 
 
drawAllShapes(new circle.Circle()); 
drawAllShapes(new triangle.Triangle());
```



# 