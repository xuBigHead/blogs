---
layout: post
title: Go 标准库 reflect.md
categories: [Go]
description: Go
keywords: Go
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Go 标准库 reflect

## 反射概述

Go中提供的反射功能带来了很多动态特性。很多标准库，比如`fmt`和很多`encoding`包均十分依赖于反射机制。可以通过`reflect`库包中`Type`和`Value`两个类型提供的功能来观察不同的Go值。

Go反射机制设计的目标之一是任何非反射操作都可以通过反射机制来完成。由于各种各样的原因，此目标并没有得到100%的实现。但是目前大部分的非反射操作都可以通过反射机制来完成。另一方面，通过反射也可以完成一些使用非反射操作不可能完成的操作。



### 反射限制

- 无法通过反射动态创建一个接口类型；
- 使用反射动态创建结构体类型的时候可能会有各种不完美的情况出现；
- 无法通过反射来声明一个新的类型。



## 反射函数

### DeepEqual

`reflect.DeepEqual(x, y)`和`x == y`的结果可能会不同。

- 如果表达式`x`和`y`的类型不相同，则函数调用`DeepEqual(x, y)`的结果总为`false`，但`x == y`的估值结果有可能为`true`；
- 如果`x`和`y`为同类型的两个引用着不同其它值的指针值，则`x == y`的估值结果总为`false`，但函数调用`DeepEqual(x, y)`的结果可能为`true`，因为函数`reflect.DeepEqual`将比较`x`和`y`所引用的两个值；
- 当`x`和`y`均处于某个循环引用链中时，为了防止死循环，`DeepEqual`调用的结果可能为`true`；
- `DeepEqual(x, y)`调用无论如何不会产生一个恐慌，但是如果`x`和`y`是两个动态类型相同的接口值并且它们的动态类型是不可比较类型的时候，`x == y`将产生一个恐慌。



```go
func TestDeepEqual(t *testing.T) {
	type Book struct{ page int }
	x := struct{ page int }{123}
	y := Book{123}
	fmt.Println(reflect.DeepEqual(x, y)) // false
	fmt.Println(x == y)                  // true

	z := Book{123}
	fmt.Println(reflect.DeepEqual(&z, &y)) // true
	fmt.Println(&z == &y)                  // false

	type Node struct{ peer *Node }
	var q, r, s Node
	q.peer = &q // 形成一个循环引用链
	r.peer = &s // 形成一个循环引用链
	s.peer = &r
	println(reflect.DeepEqual(&q, &r)) // true
	fmt.Println(q == r)                // false

	var f1, f2 func() = nil, func() {}
	fmt.Println(reflect.DeepEqual(f1, f1)) // true
	fmt.Println(reflect.DeepEqual(f2, f2)) // false

	var a, b interface{} = []int{1, 2}, []int{1, 2}
	fmt.Println(reflect.DeepEqual(a, b)) // true
	fmt.Println(a == b)                  // 产生恐慌
}
```



## 反射类型Type

类型`reflect.Type`为一个接口类型，它指定了若干方法，通过这些方法能够获取到`reflect.Type`值所表示的Go类型的各种信息。这些方法中的有些适用于所有种类的类型，有些只适用于一种或几种类型。通过不合适的`reflect.Type`值调用某个方法将在运行时产生一个恐慌。



### 获取Type类型值

#### TypeOf函数

通过调用`reflect.TypeOf`函数可以从任何非接口类型的值创建`reflect.Type`值，此`reflect.Type`值表示着此非接口值的类型。通过此值可以得到很多此非接口类型的信息。也可以将接口值传递给`reflect.TypeOf`函数调用，但是此调用将返回表示此接口值的动态类型的`reflect.Type`值。实际上，`reflect.TypeOf`函数的唯一参数的类型为`interface{}`，`reflect.TypeOf`函数将总是返回表示着此唯一接口参数值的动态类型的`reflect.Type`值。 

```go
func main() {
	type A = [16]int16
	var c <-chan map[A][]byte
	tc := reflect.TypeOf(c)
	fmt.Println(tc.Kind())    // chan
	fmt.Println(tc.ChanDir()) // <-chan
	tm := tc.Elem()
	ta, tb := tm.Key(), tm.Elem()
	fmt.Println(tm.Kind(), ta.Kind(), tb.Kind()) // map array slice
	tx, ty := ta.Elem(), tb.Elem()

	// byte是uint8类型的别名。
	fmt.Println(tx.Kind(), ty.Kind()) // int16 uint8
	fmt.Println(tx.Bits(), ty.Bits()) // 16 8
	fmt.Println(tx.ConvertibleTo(ty)) // true
	fmt.Println(tb.ConvertibleTo(ta)) // false

	// 切片类型和映射类型都是不可比较类型。
	fmt.Println(tb.Comparable()) // false
	fmt.Println(tm.Comparable()) // false
	fmt.Println(ta.Comparable()) // true
	fmt.Println(tc.Comparable()) // true
}
```



#### TypeFor函数

从Go 1.22开始可以调用`reflect.TypeFor`函数来得到表示着编译时刻已知的类型的`reflect.Type`值。此编译时刻已知的类型可以是一个非接口类型，也可以是一个接口类型。

```go
type Animal interface {
	Eat(food string) bool
}

func TestReflectTypeFor(t *testing.T) {
	ti := reflect.TypeFor[Animal]()
	eat, r := ti.MethodByName("Eat")
	fmt.Println(ti.Kind(), eat, r)
}
```



#### 其他函数

`reflect`代码包也提供了一些其它函数来动态地创建出来一些无名组合类型。

```go
func main() {
	ta := reflect.ArrayOf(5, reflect.TypeOf(123))
	fmt.Println(ta) // [5]int
	tc := reflect.ChanOf(reflect.SendDir, ta)
	fmt.Println(tc) // chan<- [5]int
	tp := reflect.PtrTo(ta)
	fmt.Println(tp) // *[5]int
	ts := reflect.SliceOf(tp)
	fmt.Println(ts) // []*[5]int
	tm := reflect.MapOf(ta, tc)
	fmt.Println(tm) // map[[5]int]chan<- [5]int
	tf := reflect.FuncOf([]reflect.Type{ta},
				[]reflect.Type{tp, tc}, false)
	fmt.Println(tf) // func([5]int) (*[5]int, chan<- [5]int)
	tt := reflect.StructOf([]reflect.StructField{
		{Name: "Age", Type: reflect.TypeOf("abc")},
	})
	fmt.Println(tt)            // struct { Age string }
	fmt.Println(tt.NumField()) // 1
}
```



### Type类型方法

| 方法声明                            | 方法描述                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| Method(int) Method                  | 根据指定下标返回类型方法，int参数超出[0, NumMethod())范围时产生恐慌。 |
| MethodByName(string) (Method, bool) | 根据指定方法名返回方法，bool返回值表示指定方法是否存在。     |
| NumMethod() int                     | 对于非接口类型，返回可导出方法数量；对于接口类型，返回所有方法数量。 |
| Kind() Kind                         | 返回26种内定指定类型中的对应类型。                           |
| Implements(u Type) bool             | bool返回值表示类型是否实现了指定类型u。                      |
| Elem() Type                         | 返回元素的类型，类型Kind不是Array, Chan, Map, Pointer或Slice时产生恐慌。 |



### Type类型应用

#### 获取指针类型

在上面例子中使用方法`Elem`来得到某些类型的元素类型，此方法也可以用来得到一个指针类型的基类型。

```go
type T []interface{m()}
func (T) m() {}

func main() {
	tp := reflect.TypeOf(new(interface{}))
	tt := reflect.TypeOf(T{})
	fmt.Println(tp.Kind(), tt.Kind()) // ptr slice

	// 使用间接的方法得到表示两个接口类型的reflect.Type值。
	ti, tim := tp.Elem(), tt.Elem()
	fmt.Println(ti.Kind(), tim.Kind()) // interface interface

	fmt.Println(tt.Implements(tim))  // true
	fmt.Println(tp.Implements(tim))  // false
	fmt.Println(tim.Implements(tim)) // true

	// 所有的类型都实现了任何空接口类型。
	fmt.Println(tp.Implements(ti))  // true
	fmt.Println(tt.Implements(ti))  // true
	fmt.Println(tim.Implements(ti)) // true
	fmt.Println(ti.Implements(ti))  // true
}
```



#### 获取方法和属性

可以通过反射列出一个类型的所有方法和一个结构体类型的所有（导出和非导出）字段的类型，也可以通过反射列出一个函数类型的各个输入参数和返回结果类型。

```go
type F func(string, int) bool
func (f F) m(s string) bool {
	return f(s, 32)
}
func (f F) M() {}

type I interface{m(s string) bool; M()}

func main() {
	var x struct {
		F F
		i I
	}
	tx := reflect.TypeOf(x)
	fmt.Println(tx.Kind())        // struct
	fmt.Println(tx.NumField())    // 2
	fmt.Println(tx.Field(1).Name) // i
	// 包路径（PkgPath）是非导出字段（或者方法）的内在属性。
	fmt.Println(tx.Field(0).PkgPath) // 
	fmt.Println(tx.Field(1).PkgPath) // main

	tf, ti := tx.Field(0).Type, tx.Field(1).Type
	fmt.Println(tf.Kind())               // func
	fmt.Println(tf.IsVariadic())         // false
	fmt.Println(tf.NumIn(), tf.NumOut()) // 2 1
	t0, t1, t2 := tf.In(0), tf.In(1), tf.Out(0)
	// 下一行打印出：string int bool
	fmt.Println(t0.Kind(), t1.Kind(), t2.Kind())

	fmt.Println(tf.NumMethod(), ti.NumMethod()) // 1 2
	fmt.Println(tf.Method(0).Name)              // M
	fmt.Println(ti.Method(1).Name)              // m
	_, ok1 := tf.MethodByName("m")
	_, ok2 := ti.MethodByName("m")
	fmt.Println(ok1, ok2) // false true
}
```



对于非接口类型，`reflect.Type.NumMethod`方法只返回一个类型的所有导出的方法（包括通过内嵌得来的隐式方法）的个数，并且方法`reflect.Type.MethodByName`不能用来获取一个类型的非导出方法； 而对于接口类型，则并无这些限制。

虽然`reflect.Type.NumField`方法返回一个结构体类型的所有字段（包括非导出字段）的数目，但是不推荐使用方法`reflect.Type.FieldByName`来获取非导出字段。

> FieldByName and related functions consider struct field names to be equal if the names are equal, even if they are unexported names originating in different packages. The practical effect of this is that the result of t.FieldByName("x") is not well defined if the struct type t contains multiple fields named x (embedded from different packages). FieldByName may return one of the fields named x or may report that there are none. See https://golang.org/issue/4876 for more details.



#### 获取标签

可以通过反射来检视结构体字段的标签信息，结构体字段标签的类型为`reflect.StructTag`，它的方法`Get`和`Lookup`用来检视字段标签中的键值对。

```go
type T struct {
	X    int  `max:"99" min:"0" default:"0"`
	Y, Z bool `optional:"yes"`
}

func main() {
	t := reflect.TypeOf(T{})
	x := t.Field(0).Tag
	y := t.Field(1).Tag
	z := t.Field(2).Tag
	fmt.Println(reflect.TypeOf(x)) // reflect.StructTag
	// v的类型为string
	v, present := x.Lookup("max")     
	fmt.Println(len(v), present)      // 2 true
	fmt.Println(x.Get("max"))         // 99
	fmt.Println(x.Lookup("optional")) //  false
	fmt.Println(y.Lookup("optional")) // yes true
	fmt.Println(z.Lookup("optional")) // yes true
}
```



## 反射类型Value

`reflect.Value`类型有很多方法，可以调用这些方法来获取和修改一个`reflect.Value`值表示的Go值。这些方法中的有些适用于所有种类类型的值，有些只适用于一种或几种类型的值。通过不合适的`reflect.Value`值调用某个方法将在运行时产生一个恐慌。



### 获取Value类型值

#### ValueOf函数

调用`reflect.ValueOf`函数从非接口类型的值创建`reflect.Value`值，此`reflect.Value`值代表着此非接口值。和`reflect.TypeOf`函数类似，`reflect.ValueOf`函数也只有一个`interface{}`类型的参数。将一个接口值传递给一个`reflect.ValueOf`函数调用时，此调用返回的是代表着此接口值的动态值的一个`reflect.Value`值，必须通过间接的途径获得一个代表一个接口值的`reflect.Value`值。被一个`reflect.Value`值代表着的值常称为此`reflect.Value`值的底层值（underlying value）。



### Value类型方法

| 方法声明                | 方法描述 |
| ----------------------- | -------- |
| CanSet() bool           |          |
| Call(in []Value)        |          |
| Elem() Value            |          |
| IsZero() bool           |          |
| Kind() Kind             |          |
| Set(x Value)            |          |
| Convert(t Type) Value   |          |
| CanConvert(t Type) bool |          |
| Comparable() bool       |          |
| Equal(u Value) bool     |          |



#### Bytes()

> 注意：`reflect.Value.Bytes()`方法以后可能会被移除。

`reflect.Value.Bytes()`方法返回一个`[]byte`值，它的元素类型`byte`可能并非属主参数代表的Go切片值的元素类型。

假设自定义类型`MyByte`的底层类型为内置类型`byte`，Go类型系统禁止切片类型`[]MyByte`的值转换为类型`[]byte`，但是`reflect.Value`类型的`Bytes`方法的实现可以绕过这个限制。此实现违反了Go类型系统的规则。

```go
type MyByte byte

func TestValueBytes(t *testing.T) {
	var myBytes = []MyByte{'a', 'b', 'c'}
	var bs []byte

	// bs = []byte(myBytes) // this line fails to compile

	v := reflect.ValueOf(myBytes)
	bs = v.Bytes()                                     // okay. Violating Go type system.
	fmt.Println(bytes.HasPrefix(bs, []byte{'a', 'b'})) // true

	bs[1], bs[2] = 'r', 't'
	fmt.Printf("%s \n", myBytes) // art
}
```



### Value类型应用

#### 设置属性

`reflect.Value`值的`CanSet`方法将返回此`reflect.Value`值代表的Go值是否可以被修改（可以被赋值）。如果一个Go值可以被修改，则可以调用对应的`reflect.Value`值的`Set`方法来修改此Go值。 注意：`reflect.ValueOf`函数直接返回的`reflect.Value`值都是不可修改的。

```go
func main() {
	n := 123
	p := &n
	vp := reflect.ValueOf(p)
	fmt.Println(vp.CanSet(), vp.CanAddr()) // false false
	vn := vp.Elem() // 取得vp的底层指针值引用的值的代表值
	fmt.Println(vn.CanSet(), vn.CanAddr()) // true true
	vn.Set(reflect.ValueOf(789)) // <=> vn.SetInt(789)
	fmt.Println(n)               // 789
}
```



一个结构体值的非导出字段不能通过反射来修改。

```go
func main() {
	var s struct {
		X interface{} // 一个导出字段
		y interface{} // 一个非导出字段
	}
	vp := reflect.ValueOf(&s)
	// 如果vp代表着一个指针，下一行等价于"vs := vp.Elem()"。
	vs := reflect.Indirect(vp)
	// vx和vy都各自代表着一个接口值。
	vx, vy := vs.Field(0), vs.Field(1)
	fmt.Println(vx.CanSet(), vx.CanAddr()) // true true
	// vy is addressable but not modifiable.
	fmt.Println(vy.CanSet(), vy.CanAddr()) // false true
	vb := reflect.ValueOf(123)
	vx.Set(vb)     // okay, 因为vx代表的值是可修改的。
	// vy.Set(vb)  // 会造成恐慌，因为vy代表的值是不可修改的。
	fmt.Println(s) // {123 <nil>
	fmt.Println(vx.IsNil(), vy.IsNil()) // false true
}
```



上例中同时也展示了如何间接地获取底层值为接口值的`reflect.Value`值。

有两种方法获取一个代表着一个指针所引用着的值的`reflect.Value`值：

1. 通过调用代表着此指针值的`reflect.Value`值的`Elem`方法。
2. 将代表着此指针值的`reflect.Value`值的传递给一个`reflect.Indirect`函数调用。（如果传递给一个`reflect.Indirect`函数调用的实参不代表着一个指针值，则此调用返回此实参的一个复制。）



#### 设置接口动态绑定值

`reflect.Value.Elem`方法也可以用来获取一个代表着一个接口值的动态值的`reflect.Value`值，比如下例中所示。

```go
func main() {
	var z = 123
	var y = &z
	var x interface{} = y
	v := reflect.ValueOf(&x)
	vx := v.Elem()
	vy := vx.Elem()
	vz := vy.Elem()
	vz.Set(reflect.ValueOf(789))
	fmt.Println(z) // 789
}
```



#### 绑定泛型函数

`reflect`标准库包中也提供了一些对应着内置函数或者各种非反射功能的函数。 下面这个例子展示了如何利用这些函数将一个（效率不高的）自定义泛型函数绑定到不同的类型的函数值上。

```go
func InvertSlice(args []reflect.Value) []reflect.Value {
	inSlice, n := args[0], args[0].Len()
	outSlice := reflect.MakeSlice(inSlice.Type(), 0, n)
	for i := n-1; i >= 0; i-- {
		element := inSlice.Index(i)
		outSlice = reflect.Append(outSlice, element)
	}
	return []reflect.Value{outSlice}
}

func Bind(p interface{}, f func ([]reflect.Value) []reflect.Value) {
	// invert代表着一个函数值。
	invert := reflect.ValueOf(p).Elem()
	invert.Set(reflect.MakeFunc(invert.Type(), f))
}

func main() {
	var invertInts func([]int) []int
	Bind(&invertInts, InvertSlice)
	fmt.Println(invertInts([]int{2, 3, 5})) // [5 3 2]

	var invertStrs func([]string) []string
	Bind(&invertStrs, InvertSlice)
	fmt.Println(invertStrs([]string{"Go", "C"})) // [C Go]
}
```



#### 调用函数

如果一个`reflect.Value`值的底层值为一个函数值，则我们可以调用此`reflect.Value`值的`Call`方法来调用此函数。 每个`Call`方法调用接受一个`[]reflect.Value`类型的参数（表示传递给相应函数调用的各个实参）并返回一个同类型结果（表示相应函数调用返回的各个结果）。

```go
type T struct {
	A, b int
}

func (t T) AddSubThenScale(n int) (int, int) {
	return n * (t.A + t.b), n * (t.A - t.b)
}

func main() {
	t := T{5, 2}
	vt := reflect.ValueOf(t)
	vm := vt.MethodByName("AddSubThenScale")
	results := vm.Call([]reflect.Value{reflect.ValueOf(3)})
	fmt.Println(results[0].Int(), results[1].Int()) // 21 9

	neg := func(x int) int {
		return -x
	}
	vf := reflect.ValueOf(neg)
	fmt.Println(vf.Call(results[:1])[0].Int()) // -21
	fmt.Println(vf.Call([]reflect.Value{
		vt.FieldByName("A"), // 如果是字段b，则造成恐慌
	})[0].Int()) // -5
}
```



非导出结构体字段值不能用做反射函数调用中的实参。 如果上例中的`vt.FieldByName("A")`被替换为`vt.FieldByName("b")`，则将产生一个恐慌。



#### 获取映射

下面是一个使用映射反射值的例子。方法`reflect.Value.MapRange`方法是从Go 1.12开始才支持的。

```go
func main() {
	valueOf := reflect.ValueOf
	m := map[string]int{"Unix": 1973, "Windows": 1985}
	v := valueOf(m)
	// 第二个实参为Value零值时，表示删除一个映射条目。
	v.SetMapIndex(valueOf("Windows"), reflect.Value{})
	v.SetMapIndex(valueOf("Linux"), valueOf(1991))
	for i := v.MapRange(); i.Next(); {
		fmt.Println(i.Key(), "\t:", i.Value())
	}
}
```



#### 获取通道

下面是一个使用通道反射值的例子。

```go
func main() {
	c := make(chan string, 2)
	vc := reflect.ValueOf(c)
	vc.Send(reflect.ValueOf("C"))
	succeeded := vc.TrySend(reflect.ValueOf("Go"))
	fmt.Println(succeeded) // true
	succeeded = vc.TrySend(reflect.ValueOf("C++"))
	fmt.Println(succeeded) // false
	fmt.Println(vc.Len(), vc.Cap()) // 2 2
	vs, succeeded := vc.TryRecv()
	fmt.Println(vs.String(), succeeded) // C true
	vs, sentBeforeClosed := vc.Recv()
	fmt.Println(vs.String(), sentBeforeClosed) // Go true
	vs, succeeded = vc.TryRecv()
	fmt.Println(vs.String()) // <$1>
	fmt.Println(succeeded)   // false
}
```



`reflect.Value`类型的`TrySend`和`TryRecv`方法对应着只有一个`case`分支和一个`default`分支的`select`流程控制代码块。可以使用`reflect.Select`函数在运行时刻来模拟具有不定`case`分支数量的`select`流程控制代码块。

```go
func main() {
	c := make(chan int, 1)
	vc := reflect.ValueOf(c)
	succeeded := vc.TrySend(reflect.ValueOf(123))
	fmt.Println(succeeded, vc.Len(), vc.Cap()) // true 1 1

	vSend, vZero := reflect.ValueOf(789), reflect.Value{}
	branches := []reflect.SelectCase{
		{Dir: reflect.SelectDefault, Chan: vZero, Send: vZero},
		{Dir: reflect.SelectRecv, Chan: vc, Send: vZero},
		{Dir: reflect.SelectSend, Chan: vc, Send: vSend},
	}
	selIndex, vRecv, sentBeforeClosed := reflect.Select(branches)
	fmt.Println(selIndex)         // 1
	fmt.Println(sentBeforeClosed) // true
	fmt.Println(vRecv.Int())      // 123
	vc.Close()
	// 再模拟一次select流程控制代码块。因为vc已经关闭了，
	// 所以需将最后一个case分支去除，否则它可能会造成一个恐慌。
	selIndex, _, sentBeforeClosed = reflect.Select(branches[:2])
	fmt.Println(selIndex, sentBeforeClosed) // 1 false
}
```



#### 获取空值

一些`reflect.Value`值可能表示着不合法的Go值。 这样的值为`reflect.Value`类型的零值（即没有底层值的`reflect.Value`值）。

```go
func main() {
	var z reflect.Value // 一个reflect.Value零值
	fmt.Println(z)      // <$1>
	v := reflect.ValueOf((*int)(nil)).Elem()
	fmt.Println(v)      // <$1>
	fmt.Println(v == z) // true
	var i = reflect.ValueOf([]interface{}{nil}).Index(0)
	fmt.Println(i)             // <nil>
	fmt.Println(i.Elem())      // <$1>
	fmt.Println(i.Elem() == z) // true
}
```



#### 断言接口

使用空接口`interface{}`值做为中介，一个Go值可以转换为一个`reflect.Value`值。 逆过程类似，通过调用一个`reflect.Value`值的`Interface`方法得到一个`interface{}`值，然后将此`interface{}`断言为原来的Go值。 但是，请注意，调用一个代表着非导出字段的`reflect.Value`值的`Interface`方法将导致一个恐慌。

```go
func main() {
	vx := reflect.ValueOf(123)
	vy := reflect.ValueOf("abc")
	vz := reflect.ValueOf([]bool{false, true})
	vt := reflect.ValueOf(time.Time{})

	x := vx.Interface().(int)
	y := vy.Interface().(string)
	z := vz.Interface().([]bool)
	m := vt.MethodByName("IsZero").Interface().(func() bool)
	fmt.Println(x, y, z, m()) // 123 abc [false true] true

	type T struct {x int}
	t := &T{3}
	v := reflect.ValueOf(t).Elem().Field(0)
	fmt.Println(v)             // 3
	fmt.Println(v.Interface()) // panic
}
```



`Value.IsZero`方法是Go 1.13中引进的，此方法用来查看一个值是否为零值。



#### 判断切片是否可转换

从Go 1.17开始，一个切片可以被转化为一个相同元素类型的数组的指针类型。但是如果在这样的一个转换中数组类型的长度过长，将导致恐慌产生。因此Go 1.17同时引入了一个`Value.CanConvert(T Type)`方法，用来检查一个转换是否会成功（即不会产生恐慌）。一个使用了`CanConvert`方法的例子：

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	s := reflect.ValueOf([]int{1, 2, 3, 4, 5})
	ts := s.Type()
	t1 := reflect.TypeOf(&[5]int{})
	t2 := reflect.TypeOf(&[6]int{})
	fmt.Println(ts.ConvertibleTo(t1)) // true
	fmt.Println(ts.ConvertibleTo(t2)) // true
	fmt.Println(s.CanConvert(t1))     // true
	fmt.Println(s.CanConvert(t2))     // false
}
```



## 反射类型Method

### Method类型方法

| 方法声明          | 方法描述 |
| ----------------- | -------- |
| IsExported() bool |          |



## 反射类型StrucField

### StrucField类型方法

| 方法声明          | 方法描述 |
| ----------------- | -------- |
| IsExported() bool |          |



## 反射类型StructTag

### StructTag类型方法

| 方法声明                                   | 方法描述 |
| ------------------------------------------ | -------- |
| Get(key string) string                     |          |
| Lookup(key string) (value string, ok bool) |          |
