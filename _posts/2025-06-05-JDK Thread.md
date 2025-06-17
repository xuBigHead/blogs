---
layout: post
title: 第018章-JDK Thread
categories: [Java]
description: 
keywords: JDK Thread.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Thread

## Object

### wait



### notify



### Additional

#### wait()之前执行了notify()

如果当前的线程不是此对象锁的所有者，却调用该对象的notify()或wait()方法时抛出IllegalMonitorStateException异常；

如果当前线程是此对象锁的所有者，wait()将一直阻塞，因为后续将没有其它notify()唤醒它。



## 工作原理

### 常用方法

#### sleep

Thread.sleep(long millis)，一定是当前线程调用此方法，当前线程进入TIMED_WAITING状态，但不释放对象锁，millis后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。



#### yield

Thread.yield()，一定是当前线程调用此方法，当前线程放弃获取的CPU时间片，但不释放锁资源，由运行状态变为就绪状态，让OS再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield()不会导致阻塞。该方法与sleep()类似，只是不能由用户指定暂停多长时间。

yield方法可以暂停当前正在执行的线程对象，让其它有相同优先级的线程执行。它是一个静态方法而且只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行。



Thread类的sleep()和yield()方法将在当前正在执行的线程上运行。所以在其他处于等待状态的线程上调用这些方法是没有意义的。这就是为什么这些方法是静态的。它们可以在当前正在执行的线程中工作，并避免程序员错误的认为可以在其他非运行线程调用这些方法。



#### join

thread.join()/thread.join(long millis)，当前线程里调用其它线程t的join方法，当前线程进入WAITING/TIMED_WAITING状态，当前线程不会释放已经持有的对象锁。线程t执行完毕或者millis时间到，当前线程一般情况下进入RUNNABLE状态，也有可能进入BLOCKED状态（因为join是基于wait实现的）。



#### wait

obj.wait()，当前线程调用对象的wait()方法，当前线程释放对象锁，进入等待队列。依靠notify()/notifyAll()唤醒或者wait(long timeout) timeout时间到自动唤醒。



#### 其它

LockSupport.park()/LockSupport.parkNanos(long nanos),LockSupport.parkUntil(long deadlines), 当前线程进入WAITING/TIMED_WAITING状态。对比wait方法,不需要获得锁就可以让线程进入WAITING/TIMED_WAITING状态，需要通过LockSupport.unpark(Thread thread)唤醒。



### start和run的区别

start()方法被用来启动新创建的线程，使该被创建的线程状态变为可运行状态。

直接调用run()方法时，只会是在原来的线程中调用，没有新的线程启动。只有调用start()方法才会启动新线程。



- `start`方法可以启动一个新线程，`run`方法只是类的一个普通方法而已，如果直接调用`run`方法，程序中依然只有主线程这一个线程。
- `start`方法实现了多线程，而`run`方法没有实现多线程。
- `start`不能被重复调用，而`run`方法可以。
- `start`方法中的`run`代码可以不执行完，就继续执行下面的代码，也就是说进行了**线程切换**。然而，如果直接调用`run`方法，就必须等待其代码全部执行完才能继续执行下面的代码。



### sleep和wait的区别

- 两者最主要的区别在于：**`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- 两者都可以暂停线程的执行。
- `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。`sleep()`方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。



Thread.sleep()不会释放占有的锁，Object.wait()会释放占有的锁；

Thread.sleep()必须传入时间，Object.wait()可传可不传，不传表示一直阻塞下去；

Thread.sleep()到时间了会自动唤醒，然后继续执行；

Object.wait()不带时间的，需要另一个线程使用Object.notify()唤醒；

Object.wait()带时间的，假如没有被notify，到时间了会自动唤醒，这时又分好两种情况，一是立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁；



Java程序中wait 和 sleep都会造成某种形式的暂停，它们可以满足不同的需要。wait()方法用于线程间通信，如果等待条件为真且其它线程被唤醒时它会释放锁，而sleep()方法仅仅释放CPU资源或者让当前线程停止执行一段时间，但不会释放锁。需要注意的是，sleep（）并不会让线程终止，一旦从休眠中唤醒线程，线程的状态将会被改变为Runnable，并且根据线程调度，它将得到执行。



### interrupted 和 isInterruptedd的区别

interrupted() 和 isInterrupted()的主要区别是前者会将中断状态清除而后者不会。Java多线程的中断机制是用内部标识来实现的，调用Thread.interrupt()来中断一个线程就会设置中断标识为true。当中断线程调用静态方法Thread.interrupted()来检查中断状态时，中断状态会被清零。而非静态方法isInterrupted()用来查询其它线程的中断状态且不会改变中断状态标识。简单的说就是任何抛出InterruptedException异常的方法都会将中断状态清零。无论如何，一个线程的中断状态有有可能被其它线程调用中断来改变。



### wait(),notify()和suspend(),resume()之间的区别

- `wait()`方法使得线程进入阻塞等待状态，并且释放锁
- `notify()`唤醒一个处于等待状态的线程，它一般跟`wait（）`方法配套使用。
- `suspend()`使得线程进入阻塞状态，并且不会自动恢复，必须对应的`resume()`被调用，才能使得线程重新进入可执行状态。`suspend()`方法很容易引起死锁问题。
- `resume()`方法跟`suspend()`方法配套使用。



### 停止线程

Java提供了很丰富的API但没有为停止线程提供API。JDK 1.0本来有一些像stop(), suspend() 和 resume()的控制方法，但是由于潜在的死锁威胁。因此在后续的JDK版本中他们被弃用了，之后Java API的设计者就没有提供一个兼容且线程安全的方法来停止一个线程。当run() 或者 call() 方法执行完的时候线程会自动结束，如果要手动结束一个线程，可以用volatile 布尔变量来退出run()方法的循环或者是取消任务来中断线程。



## 调度方法

| 描述     | 方法                                         |
| -------- | -------------------------------------------- |
| 休眠     | Thread.sleep(long)                           |
| 中断     | thread.interrupt()<br />thread.isInterrupt() |
| 等待     | object.wait()<br />object.wait(long)         |
| 线程让步 | Thread.yield()                               |
| 通知     | object.notify()<br />object.notifyAll()      |



### 休眠

`Thread.sleep(long)`方法，使线程转到**超时等待阻塞（TIMED_WAITING）** 状态。`long`参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，线程自动转为`就绪（Runnable）`状态。

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。



### 中断

`interrupt()`表示中断线程。需要注意的是，`InterruptedException`是线程自己从内部抛出的，并不是`interrupt()`方法抛出的。对某一线程调用`interrupt()`时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出`InterruptedException`。但是，一旦该线程进入到`wait()/sleep()/join()`后，就会立刻抛出InterruptedException。可以用`isInterrupted()`来获取状态。

调用interrupt方法不会真正的结束线程，而是给当前线程打上一个停止的标记
Thread类提供了interrupt方法测试当前线程是否已经中断，isInterrupted方法测试线程是否已经中断。

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

- **InterruptedException**

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new MyThread1();
        thread1.start();
        thread1.interrupt();
        System.out.println("Main run");
    }
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

- **interrupted()**

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。



- **Executor 的中断操作**

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。



### 等待

`Object`类中的`wait()`方法，会导致当前的线程等待，直到其他线程调用此对象的`notify()`方法或`notifyAll()`唤醒方法。



### 线程让步

`Thread.yield()`方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。



### 通知

Object的`notify()`方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。

`notifyAll()`，则是唤醒在此对象监视器上等待的所有线程。



### 停止

stopThread方法使用退出标志，使线程正常停止，即run方法运行完后线程终止。stop强制停止线程可能使一些清理性的工作得不到完成；还会对锁定的对象进行解锁，使数据得不到同步的处理，导致数据不一致。



## 线程通讯方式

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。



### join()方法

如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才 从thread.join()返回。线程Thread除了提供join()方法之外，还提供了join(long millis)和join(long millis,int nanos)两个具备超时特性的方法。这两个超时方法表示，如果线程thread在给定的超时 时间里没有终止，那么将会从该超时方法中返回。



在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

```java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}

public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
```



### 等待/通知机制

等待/通知机制，是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B 调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而 执行后续操作。

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateExeception。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。



### Condition类

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。



### volatile和synchronized关键字

volatile关键字用来修饰共享变量，保证了共享变量的可见性，任何线程需要读取时都要到内存中读取（确保获得最新值）。

synchronized关键字确保只能同时有一个线程访问方法或者变量，保证了线程访问的可见性和排他性。



### 管道输入/输出流

管道输入/输出流和普通的文件输入/输出流或者网络输入/输出流不同之处在于，它主要 用于线程之间的数据传输，而传输的媒介为内存。

管道输入/输出流主要包括了如下4种具体实现：PipedOutputStream、PipedInputStream、 PipedReader和PipedWriter，前两种面向字节，而后两种面向字符。



### ThreadLocal

ThreadLocal，即线程本地变量（每个线程都有自己唯一的一个哦），是一个以ThreadLocal对象为键、任意对象为值的存储结构。底层是一个ThreadLocalMap来存储信息，key是弱引用，value是强引用，所以使用完毕后要及时清理(尤其使用线程池时)。



## 源码解析

### 属性

> java.lang.Thread

```java
    /* 
     * 与此线程有关的 ThreadLocal 值。此映射由 ThreadLocal 类维护。 
     */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * 与此线程有关的 InheritableThreadLocal 值。
     * 该映射由 InheritableThreadLocal 类维护。
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```



- threadLocals

该变量默认为null，只有调用 ThreadLoacl 的 get 和 set 方法时才会创建。



### run

> java.lang.Thread

```java
    /**
     * 如果该线程是使用单独的 Runnable 运行对象构造的，则调用该 Runnable对象的 run 方法；
     * 否则，此方法不执行任何操作并返回。
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```



### start

```java
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();

    /**
     * Notifies the group that the thread {@code t} has failed
     * an attempt to start.
     *
     * <p> The state of this thread group is rolled back as if the
     * attempt to start the thread has never occurred. The thread is again
     * considered an unstarted member of the thread group, and a subsequent
     * attempt to start the thread is permitted.
     *
     * @param  t
     *         the Thread whose start method was invoked
     */
    void threadStartFailed(Thread t) {
        synchronized(this) {
            remove(t);
            nUnstartedThreads++;
        }
    }

    /**
     * 从此组中删除指定的线程。在已销毁的线程组上调用此方法无效。
     */
    private void remove(Thread t) {
        synchronized (this) {
            if (destroyed) {
                return;
            }
            for (int i = 0 ; i < nthreads ; i++) {
                if (threads[i] == t) {
                    System.arraycopy(threads, i + 1, threads, i, --nthreads - i);
                    // 对死线程的空引用，以便垃圾收集器收集它。
                    threads[nthreads] = null;
                    break;
                }
            }
        }
    }
```



new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**



### sleep

> java.lang.Thread

```java
    /**
     * 使当前执行的线程休眠（暂时停止执行）指定的毫秒数，
     * 取决于系统计时器和调度程序的精度和准确性。该线程不会失去任何监视器的所有权。
     */
    public static native void sleep(long millis) throws InterruptedException;

    /**
     * 使当前执行的线程休眠（暂时停止执行）指定的毫秒数加上指定的纳秒数，
     * 具体取决于系统计时器和调度程序的精度和准确性。该线程不会失去任何监视器的所有权。
     */
    public static void sleep(long millis, int nanos)
    throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        sleep(millis);
    }
```



### join

> java.lang.Thread

```java
    public final void join() throws InterruptedException {
        join(0);
    }

    public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }

    public final synchronized void join(long millis, int nanos)
    throws InterruptedException {

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        join(millis);
    }
```



### yield

> java.lang.Thread

```java
    public static native void yield();
```



### interrupt

`interrupt` 它是真正触发中断的方法。



> java.lang.Thread

```java
    public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // Just to set the interrupt flag
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }
```



### interrupted

`interrupted`是Thread中的一个类方法,它也调用了isInterrupted(true)方法，不过它传递的参数是true，表示将会清除中断标志位。



> java.lang.Thread

```java
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }

    private native boolean isInterrupted(boolean ClearInterrupted);
```



### isInterruptedd

`isInterrupted`是`Thread`类中的一个实例方法，可以判断实例线程是否被中断。



> java.lang.Thread

```java
    public boolean isInterrupted() {
        return isInterrupted(false);
    }
```



### holdsLock

> java.lang.Thread

```java
    public static native boolean holdsLock(Object obj);
```



返回true如果当且仅当当前线程拥有某个具体对象的锁。可以通过这个方法来判断线程是否拥有锁。



### setPriority

> java.lang.Thread

```java
    public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
```



### setDaemon

> java.lang.Thread

```java
    public final void setDaemon(boolean on) {
        checkAccess();
        if (isAlive()) {
            throw new IllegalThreadStateException();
        }
        daemon = on;
    }
```



使用Thread类的setDaemon(true)方法可以将线程设置为守护线程，需要注意的是，需要在调用start()方法前调用这个方法，否则会抛出IllegalThreadStateException异常。



### 异常相关

#### 获取未捕获异常

> java.lang.Thread

```java
private volatile UncaughtExceptionHandler uncaughtExceptionHandler;

public UncaughtExceptionHandler getUncaughtExceptionHandler() {
    return uncaughtExceptionHandler != null ?
        uncaughtExceptionHandler : group;
}
```



> java.lang.Thread.UncaughtExceptionHandler

```java
@FunctionalInterface
public interface UncaughtExceptionHandler {
    /**
     * 处理未捕获异常
     */
    void uncaughtException(Thread t, Throwable e);
}
```



如果异常没有被捕获该线程将会停止执行。Thread.UncaughtExceptionHandler是用于处理未捕获异常造成线程突然中断情况的一个内嵌接口。

当一个未捕获异常将造成线程中断的时候，JVM会使用Thread.getUncaughtExceptionHandler()来查询线程的UncaughtExceptionHandler并将线程和异常作为参数传递给handler的uncaughtException()方法进行处理。



## Runnable

Runnable 接口不会返回结果或抛出检查异常，但是Callable 接口可以。所以，如果任务不需要返回结果或抛出异常推荐使用 Runnable 接口。



### 互相转换

### 源码解析

> java.lang.Runnable

```java
@FunctionalInterface
public interface Runnable {
    /**
     * 当使用实现接口Runnable的对象来创建线程时，启动线程会导致在该单独执行的线程中调用对象的run方法。
     */
    public abstract void run();
}
```



#### 和Thread的关系

> java.lang.Thread

```java
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}

private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    // ...
    this.target = target;
    // ...
}

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```



Runnable接口实现对象可以作为Thread的构造器参数，Thread对象调用start方法时实际执行的就是Runnable接口实现对象的run方法。

## Best Practice

### 实现线程

- 定义`Thread`类的子类，并重写该类的`run`方法
- 定义`Runnable`接口的实现类，并重写该接口的`run()`方法
- 定义`Callable`接口的实现类，并重写该接口的`call()`方法，一般配合`Future`使用
- 线程池的方式



#### 继承Thread类

继承Thread类，然后重写run方法。由于Java单继承的特性，这种方式用的比较少。

```java
public class MyThread extends Thread {
	public MyThread() {
		
	}
	public void run() {
		for(int i=0;i<10;i++) {
			System.out.println(Thread.currentThread()+":"+i);
		}
	}
	public static void main(String[] args) {
		MyThread mThread1=new MyThread();
		MyThread mThread2=new MyThread();
		MyThread myThread3=new MyThread();
		mThread1.start();
		mThread2.start();
		myThread3.start();
	}
}

```



#### 实现Runnable接口

实现Runable接口并实现run方法。

- 覆写Runnable接口实现多线程可以避免单继承局限；

- 实现Runnable()可以更好的体现共享的概念。



```java
public class MyTarget implements Runnable{
	public static int count=20;
	public void run() {
		while(count>0) {
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName()+"-当前剩余票数:"+count--);
		}
	}
	public static void main(String[] args) {
		MyThread target=new MyTarget();
		Thread mThread1=new Thread(target,"线程1");
		Thread mThread2=new Thread(target,"线程2");
		Thread mThread3=new Thread(target,"线程3");
		mThread1.start();
		mThread2.start();
		myThread3.start();
	}
}

```



#### 实现Callable接口

实现Callable接口并实现call方法。

```java
public class MyTarget implements Callable<String> {
	private int count = 20;

	@Override
	public String call() throws Exception {
		for (int i = count; i > 0; i--) {
			System.out.println(Thread.currentThread().getName()+"当前票数：" + i);
		}
		return "sale out";
	} 

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		Callable<String> callable  =new MyTarget();
		FutureTask <String>futureTask=new FutureTask<>(callable);
		Thread mThread=new Thread(futureTask);
		Thread mThread2=new Thread(futureTask);
		Thread mThread3=new Thread(futureTask);
		mThread.start();
		mThread2.start();
		mThread3.start();
		System.out.println(futureTask.get());
		
	}
}
```



##### 与Runnable对比

Runnable和Callable都代表那些要在不同的线程中执行的任务目标target。

Runnable从JDK1.0开始就有了，Callable是在JDK1.5增加的。它们的主要区别是Callable的 call() 方法可以返回值和抛出异常，而Runnable的run()方法没有这些功能。

- `Runnable`接口中的`run()`方法没有返回值，是`void`类型，它做的事情只是纯粹地去执行`run()`方法中的代码而已；
- `Callable`接口中的`call()`方法是有返回值的，是一个泛型。它一般配合`Future、FutureTask`一起使用，用来获取异步执行的结果。
- `Callable`接口`call()`方法允许抛出异常；而`Runnable`接口`run()`方法不能继续上抛异常；



#### 通过线程池



### 线程顺序执行

在多线程中有多种方法让线程按特定顺序执行，你可以用线程类的join()方法在一个线程中启动另一个线程，另外一个线程完成该线程继续执行。为了确保三个线程的顺序你应该先启动最后一个(T3调用T2，T2调用T1)，这样T1就会先完成而T3最后完成。

```java
public class ThreadTest {
    public static void main(String[] args) {
        Thread spring = new Thread(new SeasonThreadTask("春天"));
        Thread summer = new Thread(new SeasonThreadTask("夏天"));
        Thread autumn = new Thread(new SeasonThreadTask("秋天"));

        try
        {
            //春天线程先启动
            spring.start();
            //主线程等待线程spring执行完，再往下执行
            spring.join();
            //夏天线程再启动
            summer.start();
            //主线程等待线程summer执行完，再往下执行
            summer.join();
            //秋天线程最后启动
            autumn.start();
            //主线程等待线程autumn执行完，再往下执行
            autumn.join();
        } catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }
}

class SeasonThreadTask implements Runnable{
    private String name;
    public SeasonThreadTask(String name){
        this.name = name;
    }
    @Override
    public void run() {
        for (int i = 1; i <4; i++) {
            System.out.println(this.name + "来了: " + i + "次");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



### 异常处理

#### 同步代码块

无论你的同步块是正常还是异常退出的，里面的线程都会释放锁，所以对比锁接口我们更喜欢同步块，因为它不用花费精力去释放锁，该功能可以在finally块里释放锁实现。



#### Thread线程

Thread.UncaughtExceptionHandler是java SE5中的新接口，它允许我们在每一个Thread对象上添加一个异常处理器。



### 使用线程

**给线程起个有意义的名字**，这样可以方便找bug或追踪；

**避免锁定和缩小同步的范围**，锁花费的代价高昂且上下文切换更耗费时间空间，试试最低限度的使用同步和锁，缩小临界区。因此相对于同步方法我更喜欢同步块，它给我拥有对锁的绝对控制权。

**多用同步类少用wait 和 notify**，首先，CountDownLatch, Semaphore, CyclicBarrier 和 Exchanger 这些同步类简化了编码操作，而用wait和notify很难实现对复杂控制流的控制。其次，这些类是由最好的企业编写和维护在后续的JDK中它们还会不断优化和完善。

**多用并发集合少用同步集合**，这是另外一个容易遵循且受益巨大的最佳实践，并发集合比同步集合的可扩展性更好，所以在并发编程时使用并发集合效果更好。

## Others_Related_Class

### ThreadGroup

ThreadGroup是一个类，它的目的是提供关于线程组的信息。

ThreadGroup API比较薄弱，它并没有比Thread提供了更多的功能。它有两个主要的功能：一是获取线程组中处于活跃状态线程的列表；二是设置为线程设置未捕获异常处理器(ncaught exception handler)。但在Java 1.5中Thread类也添加了setUncaughtExceptionHandler(UncaughtExceptionHandler eh) 方法，所以ThreadGroup是已经过时的，不建议继续使用。



### Timer相关

java.util.Timer是一个工具类，可以用于安排一个线程在未来的某个特定时间执行。Timer类可以用安排一次性任务或者周期任务。

java.util.TimerTask是一个实现了Runnable接口的抽象类，我们需要去继承这个类来创建我们自己的定时任务并使用Timer去安排它的执行。



## Application Example

### wait & notify

```java
class MyThread extends Thread {
    
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();            
        synchronized (myThread) {
            try {        
                myThread.start();
                // 主线程睡眠3s
                Thread.sleep(3000);
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

运行结果：

```
before wait
before notify
after notify
after wait
```



执行流程如下：

![img](https://oss.xubighead.top/oss/image/202506/1930512680407371778.jpg)



使用wait/notify实现同步时，必须先调用wait，后调用notify，如果先调用notify，再调用wait，将起不了作用。具体代码如下：

```java
class MyThread extends Thread {
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();        
        myThread.start();
        // 主线程睡眠3s
        Thread.sleep(3000);
        synchronized (myThread) {
            try {        
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

运行结果：

```
before notify
after notify
before wait
```



说明: 由于先调用了notify，再调用的wait，此时主线程还是会一直阻塞。