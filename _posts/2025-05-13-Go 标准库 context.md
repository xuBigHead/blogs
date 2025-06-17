---
layout: post
title: Go 标准库 context.md
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
# Go 标准库 `context`

## 概述

`context`包在`go1.7`版本中引入到标准库，用来在`goroutine`之间传递上下文信息，相同的`context`可以传递给运行在不同`goroutine`中的函数，上下文对于多个`goroutine`同时使用是并发安全的，`context`包定义了上下文类型，可以使用 `Background`、`TODO`创建一个上下文，在函数调用链之间传播`context`，也可以使用`WithDeadline`、`WithTimeout`、`WithCancel` 或 `WithValue` 创建的修改副本替换它。

总结起就是一句话：`context`的作用就是在不同的`goroutine`之间同步请求特定的数据、取消信号、超时实践以及处理请求的截止日期。



### 包元素简介

context包定义接口如下：

| 类型     | 作用                             |
| -------- | -------------------------------- |
| Context  | 定义了 Context 接口的四个方法    |
| canceler | context 取消接口，定义了两个方法 |



context包定义结构体如下：

| 类型                  | 作用                                            |
| --------------------- | ----------------------------------------------- |
| emptyCtx              | 实现了 Context 接口，它其实是个空的 context     |
| backgroundCtx         | 通常用在 main 函数中，作为所有 context 的根节点 |
| todoCtx               | 通常用在并不知道传递什么 context的情形          |
| cancelCtx             | 可以被取消                                      |
| timerCtx              | 超时会被取消                                    |
| valueCtx              | 可以存储 k-v 对                                 |
| deadlineExceededError | 超时异常                                        |



context包定义函数如下：

| 类型            | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| CancelFunc      | 取消函数                                                     |
| Background      | 返回一个空的 context，常作为根 context                       |
| TODO            | 返回一个空的 context，常用于重构时期，没有合适的 context 可用 |
| WithCancel      | 基于父 context，生成一个可以取消的 context                   |
| newCancelCtx    | 创建一个可取消的 context                                     |
| propagateCancel | 向下传递 context 节点间的取消关系                            |
| parentCancelCtx | 找到第一个可取消的父节点                                     |
| removeChild     | 去掉父节点的孩子节点                                         |
| init            | 包初始化                                                     |
| WithDeadline    | 创建一个有 deadline 的 context                               |
| WithTimeout     | 创建一个有 timeout 的 context                                |
| WithValue       | 创建一个存储 k-v 对的 context                                |



## 应用

### 创建 `context`

`context`包主要提供了两种方式创建 `context`，这两个函数其实只是互为别名，没有差别：

- `context.Backgroud()`
- `context.TODO()`

```go
func TestCtxCreate(t *testing.T) {
	// 上下文的默认值，所有其他的上下文都应该从它衍生（Derived）出来。在大多数情况下，都使用`context.Background`作为起始的上下文向下传递。
	ctxByBackground := context.Background()
	ctxByBackground = context.WithValue(ctxByBackground, "test", "test")
	test.Assert(ctxByBackground.Value("test"), "test", t)

	// 在不确定应该使用哪种上下文时使用
	ctxByTodo := context.TODO()
	ctxByTodo = context.WithValue(ctxByTodo, "test", "test")
	test.Assert(ctxByTodo.Value("test"), "test", t)
}
```



`context.Backgroud()` 和 `context.TODO()` 创建根 `context` 后，不具备任何功能，具体实践还是要依靠 `context` 包提供的 `With` 系列函数来进行派生：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context
```



这四个函数都要基于父`Context`衍生，通过这些函数，就创建了一颗Context树，树的每个节点都可以有任意多个子节点，节点层级可以有任意多个，画个图表示一下：

![1980416381-9bbc1138d1d9cea5_fix732](D:\download\image\1826534021669183490.png)

基于一个父`Context`可以随意衍生，其实这就是一个`Context`树，树的每个节点都可以有任意多个子节点，节点层级可以有任意多个，每个子节点都依赖于其父节点。例如上图，可以基于`Context.Background`衍生出四个子`context`：`ctx1.0-cancel`、`ctx2.0-deadline`、`ctx3.0-timeout`、`ctx4.0-withvalue`，这四个子`context`还可以作为父`context`继续向下衍生，即使其中`ctx1.0-cancel` 节点取消了，也不影响其他三个父节点分支。



### `WithValue`携带数据

类似于在`java`中用`ThreadLocal`来传递数据，在`Go`语言中就可以使用`Context`来传递。

```go
// TestCtxWithValue 测试ctx使用WithValue携带数据并取出数据
func TestCtxWithValue(t *testing.T) {
	key := "customCtxValue"
	want := "这是自定义ctx值"
	ctx := context.WithValue(context.Background(), key, want)
	got := getCustomCtxValue(ctx, key)
	test.Assert(got, want, t) // excepted:这是自定义ctx值, got:这是自定义ctx值
}

func getCustomCtxValue(ctx context.Context, key string) any {
	return ctx.Value(key)
}
```



在使用`withVaule`时要注意四个事项：

- 不建议使用`context`值传递关键参数，关键参数应该显示的声明出来，不应该隐式处理，`context`中最好是携带签名、`trace_id`这类值。
- 因为携带`value`也是`key`、`value`的形式，为了避免`context`因多个包同时使用`context`而带来冲突，`key`建议采用内置类型。
- 上面的例子我们获取`trace_id`是直接从当前`ctx`获取的，实际我们也可以获取父`context`中的`value`，在获取键值对是，我们先从当前`context`中查找，没有找到会在从父`context`中查找该键对应的值直到在某个父`context`中返回 `nil` 或者查找到对应的值。
- `context`传递的数据中`key`、`value`都是`interface`类型，这种类型编译期无法确定类型，所以不是很安全，所以在类型断言时别忘了保证程序的健壮性。



### `WithTimeout` 和 `WithDeadline` 超时控制

`withTimeout`和`withDeadline`作用是一样的，就是传递的时间参数不同而已，他们都会通过传入的时间来自动取消`Context`，这里要注意的是他们都会返回一个`cancelFunc`方法，通过调用这个方法可以达到提前进行取消，不过在使用的过程还是建议在自动取消后也调用`cancelFunc`去停止定时减少不必要的资源浪费。

```go
// TestCtxWithTimeout 测试ctx使用WithTime进行超时控制，WithTime内部本质调用的是WithDeadline
func TestCtxWithTimeout(t *testing.T) {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    for i := 0; i < 10; i++ {
       time.Sleep(1 * time.Second)
       select {
       case <-ctx.Done():
          fmt.Println("time out, ", ctx.Err())
          return
       default:
          fmt.Printf("deal time is %d\n", i)
          // 手动调用cancel()方法可以主动结束执行
          // cancel()
       }
    }
}

// TestCtxWithTimeout 测试ctx使用WithDeadline进行超时控制
func TestCtxWithDeadline(t *testing.T) {
    ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(3*time.Second))
    defer cancel()
    for i := 0; i < 10; i++ {
       time.Sleep(1 * time.Second)
       select {
       case <-ctx.Done():
          fmt.Println(ctx.Err())
          return
       default:
          fmt.Printf("deal time is %d\n", i)
       }
    }
}
```



`withTimeout`和`withDeadline`都是可以超时自动取消，又可以手动控制取消。**从请求入口透传的调用链路中的`context`是携带超时时间的，如果想在其中单独开一个goroutine去处理其他的事情并且不会随着请求结束后而被取消的话，那么传递的`context`要基于`context.Background`或者`context.TODO`重新衍生一个传递，否决就会和预期不符合了。**



### `withCancel`取消控制

业务开发中往往为了完成一个复杂的需求会开多个`gouroutine`去做一些事情，这就导致会在一次请求中开了多个`goroutine`却无法控制他们，这时就可以使用`withCancel`来衍生一个`context`传递到不同的`goroutine`中，当想让这些`goroutine`停止运行，就可以调用`cancel`来进行取消。

```go
// TestCtxWithCancel 测试ctx使用WithCancel进行取消控制
func TestCtxWithCancel(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    go speakForCtxWithCancel(ctx)
    time.Sleep(5 * time.Second)
    cancel()
    time.Sleep(1 * time.Second)
}

func speakForCtxWithCancel(ctx context.Context) {
    for range time.Tick(time.Second) {
       select {
       case <-ctx.Done():
          fmt.Println("shut up")
          return
       default:
          fmt.Println("speak ...")
       }
    }
}
```



上游任务仅仅使用context通知下游任务不再需要，但不会直接干涉和中断下游任务的执行，由下游任务自行决定后续的处理操作，也就是说context的取消操作是无侵入的。



### 自定义 Context

因为`Context`本质是一个接口，所以可以通过实现`Context`达到自定义`Context`的目的，一般在实现`Web`框架或`RPC`框架往往采用这种形式，比如`gin`框架的`Context`就是自己有封装了一层。



## 源码解析

### context包接口

#### Context

> context.go

```go
type Context interface {
    // 返回 context 是否会被取消以及自动取消时间（即 deadline）
    Deadline() (deadline time.Time, ok bool)
	// 当 context 被取消或者到了 deadline，返回一个被关闭的 channel
    Done() <-chan struct{}
	// 在 channel Done 关闭后，返回 context 取消原因
    Err() error
    // 获取 key 对应的 value
    Value(key any) any
}
```



`Context` 是一个接口，定义了 4 个方法，它们都是`幂等`的。也就是说连续多次调用同一个方法，得到的结果都是相同的。

`Done()` 返回一个 channel，可以表示 context 被取消的信号：当这个 channel 被关闭时，说明 context 被取消了。注意，这是一个只读的channel。 我们又知道，读一个关闭的 channel 会读出相应类型的零值。并且源码里没有地方会向这个 channel 里面塞入值。换句话说，这是一个 `receive-only` 的 channel。因此在子协程里读这个 channel，除非被关闭，否则读不出来任何东西。也正是利用了这一点，子协程从 channel 里读出了值（零值）后，就可以做一些收尾工作，尽快退出。

`Err()` 返回一个错误，表示 channel 被关闭的原因。例如是被取消，还是超时。

`Deadline()` 返回 context 的截止时间，通过此时间，函数就可以决定是否进行接下来的操作，如果时间太短，就可以不往下做了，否则浪费系统资源。当然，也可以用这个 deadline 来设置一个 I/O 操作的超时时间。

`Value()` 获取之前设置的 key 对应的 value。





#### canceler

```go
type canceler interface {
    cancel(removeFromParent bool, err, cause error)
    Done() <-chan struct{}
}
```



实现了上面定义的两个方法的 Context，就表明该 Context 是可取消的。源码中有两个类型实现了 canceler 接口：`*cancelCtx` 和 `*timerCtx`。注意是加了 `*` 号的，是这两个结构体的指针实现了 canceler 接口。

Context 接口设计成这个样子的原因：

- “取消”操作应该是建议性，而非强制性

caller 不应该去关心、干涉 callee 的情况，决定如何以及何时 return 是 callee 的责任。caller 只需发送“取消”信息，callee 根据收到的信息来做进一步的决策，因此接口并没有定义 cancel 方法。

- “取消”操作应该可传递

“取消”某个函数时，和它相关联的其他函数也应该“取消”。因此，`Done()` 方法返回一个只读的 channel，所有相关函数监听此 channel。一旦 channel 关闭，通过 channel 的“广播机制”，所有监听者都能收到。



### context包结构体

#### emptyCtx

> context.go

```go
// An emptyCtx is never canceled, has no values, and has no deadline.
// It is the common base of backgroundCtx and todoCtx.
type emptyCtx struct{}

func (emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (emptyCtx) Done() <-chan struct{} {
	return nil
}

func (emptyCtx) Err() error {
	return nil
}

func (emptyCtx) Value(key any) any {
	return nil
}
```



#### backgroundCtx

backgroundCtx

> context.go

```go
type backgroundCtx struct{ emptyCtx }

func (backgroundCtx) String() string {
	return "context.Background"
}
```



background 通常用在 main 函数中，作为所有 context 的根节点。



#### todoCtx

> context.go

```go
type todoCtx struct{ emptyCtx }

func (todoCtx) String() string {
	return "context.TODO"
}
```



todo 通常用在并不知道传递什么 context的情形。例如，调用一个需要传递 context 参数的函数但没有其他 context 可以传递，这时就可以传递 todo。这常常发生在重构进行中，给一些函数添加了一个 Context 参数，但不知道要传什么，就用 todo “占个位子”，最终要换成其他 context。



#### cancelCtx

> context.go

```go
type cancelCtx struct {
	Context

	mu       sync.Mutex            // protects following fields
	done     atomic.Value          // of chan struct{}, created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
	cause    error                 // set to non-nil by the first cancel call
}
```



这是一个可以取消的 Context，实现了 canceler 接口。它直接将接口 Context 作为它的一个匿名字段，这样，它就可以被看成一个 Context。



##### Value方法

```go
func (c *cancelCtx) Value(key any) any {
	if key == &cancelCtxKey {
		return c
	}
	return value(c.Context, key)
}
```



##### Done方法

```go
func (c *cancelCtx) Done() <-chan struct{} {
	d := c.done.Load()
	if d != nil {
		return d.(chan struct{})
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	d = c.done.Load()
	if d == nil {
		d = make(chan struct{})
		c.done.Store(d)
	}
	return d.(chan struct{})
}
```



c.done 是“懒汉式”创建，只有调用了 Done() 方法的时候才会被创建。再次说明，函数返回的是一个只读的 channel，而且没有地方向这个 channel 里面写数据。所以，直接调用读这个 channel，协程会被 block 住。一般通过搭配 select 来使用。一旦关闭，就会立即读出零值。



##### Err方法

```go
func (c *cancelCtx) Err() error {
	c.mu.Lock()
	err := c.err
	c.mu.Unlock()
	return err
}
```



##### propagateCancel方法

```go
func (c *cancelCtx) propagateCancel(parent Context, child canceler) {
	c.Context = parent

	done := parent.Done()
	if done == nil { // 父节点是个空节点
		return // parent is never canceled
	}

	select {
	case <-done:
		// parent is already canceled
		child.cancel(false, parent.Err(), Cause(parent))
		return
	default:
	}

	if p, ok := parentCancelCtx(parent); ok { // 找到可以取消的父 context
		// parent is a *cancelCtx, or derives from one.
		p.mu.Lock()
		if p.err != nil {
			// 父节点已经被取消了，本节点（子节点）也要取消
			child.cancel(false, p.err, p.cause)
		} else {
            // 父节点未取消
			if p.children == nil {
				p.children = make(map[canceler]struct{})
			}
            // "挂到"父节点上
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
		return
	}
   
	if a, ok := parent.(afterFuncer); ok {
		// parent implements an AfterFunc method.
		c.mu.Lock()
		stop := a.AfterFunc(func() {
			child.cancel(false, parent.Err(), Cause(parent))
		})
		c.Context = stopCtx{
			Context: parent,
			stop:    stop,
		}
		c.mu.Unlock()
		return
	}
    
 	// 如果没有找到可取消的父 context。新启动一个协程监控父节点或子节点取消信号
	goroutines.Add(1)
	go func() {
		select {
		case <-parent.Done():
			child.cancel(false, parent.Err(), Cause(parent))
		case <-child.Done():
		}
	}()
}
```



这个方法的作用就是向上寻找可以“挂靠”的“可取消”的 context，并且“挂靠”上去。这样，调用上层 cancel 方法的时候，就可以层层传递，将那些挂靠的子 context 同时“取消”。



新启动一个协程监控父节点或子节点取消信号时两个case都是必要的：

- 第一个 case 说明当父节点取消，则取消子节点。如果去掉这个 case，那么父节点取消的信号就不能传递到子节点。
- 第二个 case 是说如果子节点自己取消了，那就退出这个 select，父节点的取消信号就不用管了。如果去掉这个 case，那么很可能父节点一直不取消，这个 goroutine 就泄漏了。当然，如果父节点取消了，就会重复让子节点取消，该影响可以忽略。



##### cancel方法

```go
func (c *cancelCtx) cancel(removeFromParent bool, err, cause error) {
	if err == nil {
		panic("context: internal error: missing cancel error")
	}
	if cause == nil {
		cause = err
	}
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return // 已经被其他协程取消
	}
	c.err = err // 给 err 字段赋值
	c.cause = cause
	d, _ := c.done.Load().(chan struct{})
    // 关闭 channel，通知其他协程
	if d == nil {
		c.done.Store(closedchan) 
	} else {
		close(d)
	}
    // 递归地取消所有子节点
	for child := range c.children {
		// NOTE: acquiring the child's lock while holding parent's lock.
		child.cancel(false, err, cause)
	}
	c.children = nil // 将子节点置空
	c.mu.Unlock()

	if removeFromParent {
		removeChild(c.Context, c) // 从父节点中移除自己 
	}
}
```



总体来看，`cancel()` 方法的功能就是关闭 channel：c.done；递归地取消它的所有子节点；从父节点从删除自己。达到的效果是通过关闭 channel，将取消信号传递给了它的所有子节点。goroutine 接收到取消信号的方式就是 select 语句中的只读` c.done` 被选中。



removeFromParent 参数只有在调用WithCancel函数获取到的返回值中会传递true，cancel方法内部取消子节点时，传递的是false，主要原因如下：

- 当前调用cancel方法的ctx的所有子节点都会因为c.children = nil而被设置为空，因此不再需要在调用子节点的cancel方法时，将子节点从父节点中移除；
- 遍历子节点时如果调用 child.cancel 方法传了 true，还会造成同时遍历和删除一个 map 的境地，会有问题的。



#### timerCtx

> context.go

```go
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}
```



timerCtx 基于 cancelCtx，只是多了一个 time.Timer 和一个 deadline。Timer 会在 deadline 到来时，自动取消 context。timerCtx 的 Done 被 Close 掉，主要是由下面的某个事件触发的：

- 截止时间到了
- cancel 函数被调用
- parent 的 Done 被 close



##### cancel方法

```go
func (c *timerCtx) cancel(removeFromParent bool, err, cause error) {
    // 直接调用 cancelCtx 的取消方法
	c.cancelCtx.cancel(false, err, cause)
	if removeFromParent {
		// 从父节点中删除子节点
		removeChild(c.cancelCtx.Context, c)
	}
	c.mu.Lock()
	if c.timer != nil {
		c.timer.Stop() // 关掉定时器，这样，在deadline 到来时，不会再次取消
		c.timer = nil
	}
	c.mu.Unlock()
}
```



#### valueCtx

> context.go

```go
type valueCtx struct {
	Context
	key, val any
}
```



##### Value方法

```go
func (c *valueCtx) Value(key any) any {
    if c.key == key {
       return c.val
    }
    return value(c.Context, key)
}
```



#### deadlineExceededError

> context.go

```go
var DeadlineExceeded error = deadlineExceededError{}

type deadlineExceededError struct{}

func (deadlineExceededError) Error() string   { return "context deadline exceeded" }
func (deadlineExceededError) Timeout() bool   { return true }
func (deadlineExceededError) Temporary() bool { return true }
```



### context包导出函数

#### WithCancel

> context.go

```go
var Canceled = errors.New("context canceled")

func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := withCancel(parent)
    return c, func() { c.cancel(true, Canceled, nil) }
}

func withCancel(parent Context) *cancelCtx {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := &cancelCtx{}
	c.propagateCancel(parent, c)
	return c
}
```



`WithCancel`导出方法传入一个父 Context（这通常是一个 `background`，作为根节点），返回新建的 context，新 context 的 done channel 是新建的。

当 WithCancel 函数返回的 CancelFunc 被调用或者是父节点的 done channel 被关闭（父节点的 CancelFunc 被调用），此 context（子节点） 的 done channel 也会被关闭。

注意传给 cancelCtx.cancel 方法的参数，前者是 true，也就是说取消的时候，需要将自己从父节点里删除。



调用 `WithCancel()` 方法的时候，也就是新创建一个可取消的 context 节点时，返回的 cancelFunc 函数会传入 true。这样做的结果是：当调用返回的 cancelFunc 时，会将这个 context 从它的父节点里“除名”，因为父节点可能有很多子节点，你自己取消了，所以我要和你断绝关系，对其他人没影响。



#### WithTimeout

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```



`WithTimeout` 函数直接调用了 `WithDeadline`，传入的 deadline 是当前时间加上 timeout 的时间，也就是从现在开始再经过 timeout 时间就算超时。也就是说，`WithDeadline` 需要用的是绝对时间。



#### WithTimeoutCause

```go
func WithTimeoutCause(parent Context, timeout time.Duration, cause error) (Context, CancelFunc) {
	return WithDeadlineCause(parent, time.Now().Add(timeout), cause)
}
```



#### WithDeadline 

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
    return WithDeadlineCause(parent, d, nil)
}
```



#### WithDeadlineCause

```go
func WithDeadlineCause(parent Context, d time.Time, cause error) (Context, CancelFunc) {
    if parent == nil {
       panic("cannot create context from nil parent")
    }
    if cur, ok := parent.Deadline(); ok && cur.Before(d) {
       // 如果父节点 context 的 deadline 早于指定时间。直接构建一个可取消的 context。
	   // 原因是一旦父节点超时，自动调用 cancel 函数，子节点也会随之取消。
	   // 所以不用单独处理子节点的计时器时间到了之后，自动调用 cancel 函数
       return WithCancel(parent)
    }
    c := &timerCtx{
       deadline: d,
    }
    // 挂靠到父节点上
    c.cancelCtx.propagateCancel(parent, c)
    dur := time.Until(d) // 计算当前距离 deadline 的时间
    if dur <= 0 {
       c.cancel(true, DeadlineExceeded, cause) // 直接取消
       return c, func() { c.cancel(false, Canceled, nil) }
    }
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.err == nil {
       c.timer = time.AfterFunc(dur, func() { // d 时间后，timer 会自动调用 cancel 函数。自动取消
          c.cancel(true, DeadlineExceeded, cause)
       })
    }
    return c, func() { c.cancel(true, Canceled, nil) }
}
```



仍然要把子节点挂靠到父节点，一旦父节点取消了，会把取消信号向下传递到子节点，子节点随之取消。

有一个特殊情况是，如果要创建的这个子节点的 deadline 比父节点要晚，也就是说如果父节点是时间到自动取消，那么一定会取消这个子节点，导致子节点的 deadline 根本不起作用，因为子节点在 deadline 到来之前就已经被父节点取消了。



#### WithValue 

> context.go

```go
func WithValue(parent Context, key, val any) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
```



对 key 的要求是可比较，因为之后需要通过 key 取出 context 中的值，可比较是必须的。通过层层传递 context，最终形成这样一棵树的结构。和链表有点像，只是它的方向相反：Context 指向它的父节点，链表则指向下一个节点。通过 WithValue 函数，可以创建层层的 valueCtx，存储 goroutine 间可以共享的变量。

`WithValue` 创建 context 节点的过程实际上就是创建链表节点的过程。两个节点的 key 值是可以相等的，但它们是两个不同的 context 节点。查找的时候，会向上查找到最后一个挂载的 context 节点，也就是离得比较近的一个父节点 context。所以，整体上而言，用 `WithValue` 构造的其实是一个低效率的链表。



### context包非导出函数

#### parentCancelCtx

```go
func parentCancelCtx(parent Context) (*cancelCtx, bool) {
	done := parent.Done()
	if done == closedchan || done == nil {
		return nil, false
	}
	p, ok := parent.Value(&cancelCtxKey).(*cancelCtx)
	if !ok {
		return nil, false
	}
	pdone, _ := p.done.Load().(chan struct{})
	if pdone != done {
		return nil, false
	}
	return p, true
}
```



#### removeChild

```go
func removeChild(parent Context, child canceler) {
	if s, ok := parent.(stopCtx); ok {
		s.stop()
		return
	}
	p, ok := parentCancelCtx(parent)
	if !ok {
		return
	}
	p.mu.Lock()
	if p.children != nil {
		delete(p.children, child)
	}
	p.mu.Unlock()
}
```



#### value

```go
func value(c Context, key any) any {
    for {
       switch ctx := c.(type) {
       case *valueCtx:
          if key == ctx.key {
             return ctx.val
          }
          c = ctx.Context
       case *cancelCtx:
          if key == &cancelCtxKey {
             return c
          }
          c = ctx.Context
       case withoutCancelCtx:
          if key == &cancelCtxKey {
             // This implements Cause(ctx) == nil
             // when ctx is created using WithoutCancel.
             return nil
          }
          c = ctx.c
       case *timerCtx:
          if key == &cancelCtxKey {
             return &ctx.cancelCtx
          }
          c = ctx.Context
       case backgroundCtx, todoCtx:
          return nil
       default:
          return c.Value(key)
       }
    }
}
```



取值的过程，实际上是一个递归查找的过程。它会顺着链路一直往上找，比较当前节点的 key 是否是要找的 key，如果是，则直接返回 value。否则，一直顺着 context 往前，最终找到根节点（一般是 emptyCtx），直接返回一个 nil。所以用 Value 方法的时候要判断结果是否为 nil。

因为查找方向是往上走的，所以，父节点没法获取子节点存储的值，子节点却可以获取父节点的值。



## 实践

#### 防止 goroutine 泄漏

```go
func gen() <-chan int {
	ch := make(chan int)
	go func() {
		var n int
		for {
			fmt.Printf("n=%d\n", n)
			ch <- n // n为6时，因为已没有接收通道数据的地方，因此阻塞在此处
			n++
			time.Sleep(time.Second)
		}
	}()
	return ch
}

func TestGoroutineLeaks(t *testing.T) {
	for n := range gen() {
		fmt.Println(n)
		if n == 5 {
			break
		}
	}
	time.Sleep(5 * time.Second) // gen函数开启的协程仍在运行
}
```



当 n == 5 的时候，直接 break 掉，那么 gen 函数的协程在程序声明周期内就会执行下一步循环并阻塞在`ch <- n`处，永远不会停下来，发生了 goroutine 泄漏。用 context 改进这个例子：

```go
func genFix(ctx context.Context) <-chan int {
	ch := make(chan int)
	go func() {
		var n int
		for {
			select {
			case <-ctx.Done():
				fmt.Println("cancel ctx")
				return
			case ch <- n:
				n++
				time.Sleep(time.Second)
			}
		}
	}()
	return ch
}

func TestGoroutineLeaksFix(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel() // 避免其他地方忘记 cancel，且重复调用不影响

	for n := range genFix(ctx) {
		fmt.Println(n)
		if n == 5 {
			cancel()
			break
		}
	}
	time.Sleep(3 * time.Second)
}
```



#### 使用NewCtx和FromCtx来存取value

不要直接使用`context.WithValue()`和`context.Value("key")`来存取数据，将`context.Value`的存取做一层封装能有效降低代码冗余，增强代码可读性同时最大限度的防止一些粗心的错误。

```go
type User struct {
    Name string
}

const userKey = "user:key"

func NewCtx(ctx context.Context, u *User) context.Context {
    return context.WithValue(ctx, userKey, u)
}

func FromCtx(ctx context.Context) (*User, bool) {
    u, ok := ctx.Value(userKey).(*User)
    return u, ok
}

func TestNewContextAndFromContext(t *testing.T) {
    u := &User{Name: "Tom"}
    ctx := NewCtx(context.Background(), u)
    u2, ok := FromCtx(ctx)
    if ok {
       fmt.Println("ok:", ok, u2)
    }
}
```



#### 封装以减少TODO()或Background()

为了避免大量无用的`context`在代码中扩散，可以对这些方法做一层封装。

```go
// 假设有如下查询方法，但几乎不使用其提供的context
func QueryContext(ctx context.Context, query string, args []NamedValue) (Rows, error) {
    // ...
}
// 封装一下
func Query(query string, args []NamedValue) (Rows, error) {
    return QueryContext(context.Background(), query, args)
}
```



## 总结

context包最大的作用就是解决了cancelation的问题，所以一般在不能取消的场景下，尽量不要使用context。



### 官方使用建议

在官方博客里，对于使用 context 提出了几点建议：

1. 不要将 Context 塞到结构体里。直接将 Context 类型作为函数的第一参数，而且一般都命名为 ctx。
2. 不要向函数传入一个 nil 的 context，如果你实在不知道传什么，标准库给你准备好了一个 context：todo。
3. 不要把本应该作为函数参数的类型塞到 context 中，context 存储的应该是一些共同的数据。例如：登陆的 session、cookie 等。
4. 同一个 context 可能会被传递到多个 goroutine，别担心，context 是并发安全的。



### 优缺点

`context`包被设计出来就是做并发控制的，这个包有利有弊。



#### 缺点

- 影响代码美观，现在基本所有`web`框架、`RPC`框架都是实现了`context`，这就导致代码中每一个函数的一个参数都是`context`，即使不用也要带着这个参数透传下去。
- `context`可以携带值，但是没有任何限制，类型和大小都没有限制，也就是没有任何约束，这样很容易导致滥用，程序的健壮很难保证；还有一个问题就是通过`context`携带值不如显式传值舒服，可读性变差了。
- 可以自定义`context`，这样风险不可控，更加会导致滥用。
- `context`取消和自动取消的错误返回不够友好，无法自定义错误，出现难以排查的问题时不好排查。
- 创建衍生节点实际是创建一个个链表节点，其时间复杂度为O(n)，节点多了会掉支效率变低。



#### 优点

- 使用`context`可以更好的做并发控制，能更好的管理`goroutine`滥用。
- `context`的携带者功能没有任何限制，这样可以传递任何的数据，可以说这是一把双刃剑


