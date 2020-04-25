# Java 并发
> 进程是资源分配的最小单位，线程是 CPU 调度的最小单位。

一个简单比喻：进程 = 火车，线程 = 车厢

- 线程在进程下行进（单纯的车厢无法运行）
- 一个进程可以包含多个线程（一辆火车可以有多个车厢）
- 不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘）
- 同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）
- 进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）
- 进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）
- 进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）
- 进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"
- 进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”

### 参考：
- [biaodianfu](https://www.zhihu.com/question/25532384/answer/411179772)

## 线程的生命周期
线程是一个动态执行的过程，它也有一个从产生到死亡的过程。
![](http://images.intflag.com/concurrent02.jpg)

### 1、新建状态
使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。

### 2、就绪状态
当线程对象调用了 start() 方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待 JVM 里线程调度器的调度。

### 3、运行状态
如果就绪状态的线程获取 CPU 资源，就可以执行 run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。

### 4、阻塞状态
如果一个线程执行了 sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：

- 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。
- 同步阻塞：线程在获取 synchronized 同步锁失败（因为同步锁被其他线程占用）。
- 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

### 5、死亡状态
个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

### 参考：
- [Java 多线程编程](https://www.runoob.com/java/java-multithreading.html)

## 多线程的使用
实现多线程有三种方式：
- 实现 Runnable 接口
- 实现 Callable 接口
- 继承 Thread 类

实现 Runnable 接口和 Callable 接口的类并没有 start() 方法，不是真正意义上的线程，也不可能直接调用 run() 方法，所以必须使用 new Thread(xxx).start() 的方式开启线程，这两种方式可以理解为任务是由线程驱动而执行的。

### 1、实现 Runnable 接口
实现 Runnable 接口，然后实现 run 方法。
```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName() + ", Hello MyRunnable");
    }

    public static void main(String[] args) {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName());
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
    }
}
```
```
Thread ID: 1, Thread Name: main
Thread ID: 11, Thread Name: Thread-0, Hello MyRunnable
```
### 2、实现 Callable 接口
实现 Callable 接口，重写 call 方法并且指定返回值类型，但是在启动时要注意，需要使用 FutureTask 包装一下，然后新建线程启动，因为 Thread 的构造方法接收的是 Runnable，而 FutureTask 间接实现了 Runnable 接口，最后线程执行的结果可以通过 FutureTask 的 get 方法获取到。

Thread 构造方法：
```java
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```
FutureTask 类关系：
![](http://images.intflag.com/concurrent01.png)

案例：
```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws InterruptedException {
        String res = "Hello MyCallable";
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName() + ", " + res);
        Thread.sleep(1000);
        return res + " Return";
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName());
        MyCallable myCallable = new MyCallable();
        FutureTask<String> futureTask = new FutureTask<>(myCallable);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName() + ", " + futureTask.get());
    }
}
```
```
Thread ID: 1, Thread Name: main
Thread ID: 11, Thread Name: Thread-0, Hello MyCallable
Thread ID: 1, Thread Name: main, Hello MyCallable Return
```


### 3、继承 Thread 类
继承 Thread 类，然后重写 run 方法。
```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread ID: " + this.getId() + ", Thread Name: " + this.getName() + ", Hello MyThread");
    }

    public static void main(String[] args) {
        System.out.println("Thread ID: " + Thread.currentThread().getId() + ", Thread Name: " + Thread.currentThread().getName());
        MyThread myThread = new MyThread();
        myThread.start();
    }
}
```
```
Thread ID: 1, Thread Name: main
Thread ID: 13, Thread Name: Thread-0, Hello MyThread
```
### 4、实现接口方式与继承方式对比
- Java 是单继承，采用实现 Runnable、Callable 接口的方式后还可以继承其他类，而继承了 Thread 类后无法再继承其他类。
- 如果要访问当前线程，采用实现 Runnable、Callable 接口的方式只能使用 `Thread.currentThread()` 得到当前线程，而继承 Thread 后直接使用 `this` 关键字即可得到当前线程。

## Executor框架
### 1、概述
从 JDK5 开始，把工作单元与执行机制分离开来，工作单元包括 Runnable 和 Callable，而执行机制由 Executor 框架提供。`java.util.concurrent.Executor`，`java.util.concurrent.ExecutorService`，`java.util.concurrent.Executors` 这三者均是 JAVA Executor 框架的一部分，用来提供线程池的功能。因为创建于销毁线程消耗太多计算机资源，而且操作系统通常对线程数有限制，所以建议使用线程池来管理线程和并发执行任务，这样不用每次请求过来时就创建一个线程。从线程池免去创建销毁线程的资源(包括时间)损耗，利用这点提高了应用的响应处理速度，还可避免 "java.lang.OutOfMemoryError:unable to create new native thread" 之类的错误。

### 2、Executor框架的两级调度模型
HotSpot VM 的线程模型中，Java线程 (java.lang.Thread) 被一对一的映射为本地操作系统的线程。Java 线程的启动与销毁都与本地线程同步。操作系统会调度所有线程并将它们分配给可用的 CPU。

在上层，Java使用多线程的程序，通常会将应用分解为若干任务，然后使用用户级别的调度器 (Executor框架) 将这些任务映射为对应数量的线程；

底层，操作系统会将这些线程映射到硬件处理器上，下层硬件的调度并不受应用程序的控制。

![](http://images.intflag.com/executor01.png)

### 3、Executor
Executor 是一个抽象层接口
```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```

Executor VS Thread

- Executor 是接口，Thread 是类。
- Executor 本质上是对并行计算的抽象，允许并发代码以可管理的方式运行，也就是说 Executor 任务与执行解耦；而 Thread 是一种具体的并行运行代码的方式，任务和执行都是紧密耦合的。
- Executor 可以执行任意数量的任务，而 Thread 只能执行一个任务。
- Executor 框架负责创建和启动线程，而 Thread 由开发者负责创建和启动线程。

### 4、ExecutorService
ExecutorService 接口是 Executor 接口的子接口，增加了一些常见的对线程的控制方法，如提供提交线程任务，返回 Future 对象，终止、关闭线程池等方法。当调用 shutdown 方法时，线程池会停止接受新任务，但会完成正在 pending 中的任务。

Future 对象提供了异步执行，意味着无需等待任务执行完成，提交需要执行的任务后，它立即返回继续往下执行 ，然后在需要时检查 Future 是否有结果了，如果任务已执行完毕，通过 Future.get() 方法获取执行结果，因为 Future.get() 是阻塞方法，如果考虑可能会因不明原因而导致 Future.get() 方法阻塞下去，Future 还提供了设置获取超时方法 Future.get(long timeout, TimeUnit unit)。

![](http://images.intflag.com/executor02.png)

### 5、Executors
Executors 是一个工具类，类似于 Collections。 提供工厂方法来创建不同类型的线程池，例如 FixedThreadPool 或者 CacheThreadPool。

![](http://images.intflag.com/executor03.png)

### 参考：
- [漫谈JAVA之Executor框架(1)](https://www.jianshu.com/p/e2053d455ef3)
- [Difference between a Thread and an Executor in Java](https://javarevisited.blogspot.com/2016/12/difference-between-thread-and-executor.html)

## Thread 类详解
### 1、start()
start() 用来启动一个线程，当调用 start 方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源。

### 2、run()
run() 方法是不需要用户来调用的，当通过 start 方法启动一个线程之后，当线程获得了 CPU 执行时间，便进入 run 方法体去执行具体的任务。注意，继承 Thread 类必须重写 run 方法，在 run 方法中定义具体要执行的任务。
### 3、sleep()
sleep方法有两个重载版本：
```
sleep(long millis)                      //参数为毫秒
sleep(long millis,int nanoseconds)      //第一参数为毫秒，第二个参数为纳秒
```
sleep 相当于让线程睡眠，交出 CPU，让 CPU 去执行其他的任务。

如果需要让当前正在执行的线程暂停一段时间，并进入阻塞状态，则可以通过调用Thread类的静态 sleep() 方法来实现。

当当前线程调用 sleep() 方法进入阻塞状态后，在其睡眠时间内，该线程不会获得执行机会，即使系统中没有其他可执行线程，处于 sleep() 中的线程也不会执行，因此 sleep() 方法常用来暂停程序的执行

但是有一点要非常注意，sleep 方法不会释放锁，也就是说如果当前线程持有对某个对象的锁，则即使调用 sleep 方法，其他线程也无法访问这个对象。
```java
public class SleepTest {

    private int i = 10;

    private Object object = new Object();

    class SleepThread extends Thread {
        @Override
        public void run() {
            synchronized (object) {
                try {
                    i++;
                    System.out.println("Thread Name: " + this.getName() + ", i = " + i);
                    System.out.println("Thread Name: " + this.getName() + ", Sleep Start");
                    sleep(5000);
                    System.out.println("Thread Name: " + this.getName() + ", Sleep End");
                    i++;
                    System.out.println("Thread Name: " + this.getName() + ", i = " + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        SleepTest test = new SleepTest();
        SleepThread s1 = test.new SleepThread();
        s1.start();
        SleepThread s2 = test.new SleepThread();
        s2.start();
    }
}
```
```
Thread Name: Thread-0, i = 11
Thread Name: Thread-0, Sleep Start
Thread Name: Thread-0, Sleep End
Thread Name: Thread-0, i = 12
Thread Name: Thread-1, i = 13
Thread Name: Thread-1, Sleep Start
Thread Name: Thread-1, Sleep End
Thread Name: Thread-1, i = 14
```
如果调用了sleep方法，必须捕获 InterruptedException 异常或者将该异常向上层抛出。当线程睡眠时间满后，不一定会立即得到执行，因为此时可能 CPU 正在执行其他的任务。所以说调用 sleep 方法相当于让线程进入阻塞状态。

### 4、yield()
yield() 方法和 sleep() 方法有点相似，它也是 Thread 类提供的一个静态方法，它也可以让当前正在执行的线程暂停，但它不会阻塞该线程，它只是将该线程转入到就绪状态。即让当前线程暂停一下，让系统的线程调度器重新调度一次，完全可能的情况是：当某个线程调用了 yield() 方法暂停之后，线程调度器又将其调度出来重新执行。

调用 yield 方法会让当前线程交出 CPU 权限，让 CPU 去执行其他的线程。它跟 sleep 方法类似，同样不会释放锁。但是 yield 不能控制具体的交出 CPU 的时间，另外，当某个线程调用了 yield() 方法之后，只有优先级与当前线程相同或者比当前线程更高的处于就绪状态的线程才会获得执行机会。

注意，调用 yield 方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取 CPU 执行时间，这一点是和 sleep 方法不一样的。
### 5、join()
在线程中调用另一个线程的 join() 方法，会将当前线程挂起，直到目标线程结束。

join方法有三个重载版本：
```
join()
join(long millis)                    //参数为毫秒
join(long millis,int nanoseconds)    //第一参数为毫秒，第二个参数为纳秒
```
假如在 main 线程中，调用 thread.join 方法，则 main 方法会等待 thread 线程执行完毕或者等待一定的时间。如果调用的是无参 join 方法，则等待 thread 执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的事件。

```java
public class JoinTest {
    class JoinThread extends Thread {
        @Override
        public void run() {
            try {
                System.out.println("Thread Name: " + this.getName() + ", Sleep Start");
                sleep(5000);
                System.out.println("Thread Name: " + this.getName() + ", Sleep End");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main Start");
        JoinTest test = new JoinTest();
        JoinTest.JoinThread s1 = test.new JoinThread();
        s1.start();
        //s1.join();        //挂起 main 线程，等待 JoinThread 执行完毕再接着执行 main 线程
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main End");
    }
}
```
不调用 s1.join() 的执行结果：
```
Thread Name: main, Main Start
Thread Name: main, Main End
Thread Name: Thread-0, Sleep Start
Thread Name: Thread-0, Sleep End
```

调用 s1.join() 的执行结果：
```
Thread Name: main, Main Start
Thread Name: Thread-0, Sleep Start
Thread Name: Thread-0, Sleep End
Thread Name: main, Main End
```
可以看出，当调用 s1.join() 方法后，main 线程会进入等待，然后等待 s1 执行完之后再继续执行。

实际上调用 join 方法是调用了 Object 的 wait 方法，这个可以通过查看源码得知：
```java
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
```
wait 方法会让线程进入阻塞状态，并且会释放线程占有的锁，并交出 CPU 执行权限。

由于 wait 方法会让线程释放对象锁，所以 join 方法同样会让线程释放对一个对象持有的锁。

### 6、interrupt()
interrupt 方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；另外，通过 interrupt 方法和 isInterrupted() 方法来停止正在运行的线程。
```java
public class InterruptTest {
    class InterruptThread extends Thread {
        @Override
        public void run() {
            System.out.println("Thread Name: " + this.getName() + ", Run Start");
            try {
                System.out.println("Thread Name: " + this.getName() + ", Sleep Start");
                sleep(5000);
                System.out.println("Thread Name: " + this.getName() + ", Sleep End");
            } catch (InterruptedException e) {
                System.out.println("Thread Name: " + this.getName() + ", Interrupted Exception");
            }
            System.out.println("Thread Name: " + this.getName() + ", Run End");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main Start");
        InterruptTest test = new InterruptTest();
        InterruptTest.InterruptThread s1 = test.new InterruptThread();
        s1.start();
        Thread.sleep(1000);
        s1.interrupt();     //中断 s1 线程
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main End");
    }
}
```
```
Thread Name: main, Main Start
Thread Name: Thread-0, Run Start
Thread Name: Thread-0, Sleep Start
Thread Name: main, Main End
Thread Name: Thread-0, Interrupted Exception
Thread Name: Thread-0, Run End
```
interrupt() 只能中断出于阻塞状态的线程，不能中断正在运行的线程。
```java
public class InterruptTest {
    class InterruptThread extends Thread {
        @Override
        public void run() {
            System.out.println("Thread Name: " + this.getName() + ", Sleep Start");
            int i = 0;
            while (i < Integer.MAX_VALUE) {
                i++;
            }
            System.out.println("Thread Name: " + this.getName() + ", Sleep End");
            System.out.println("Thread Name: " + this.getName() + ", i = " + i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main Start");
        InterruptTest test = new InterruptTest();
        InterruptTest.InterruptThread s1 = test.new InterruptThread();
        s1.start();
        Thread.sleep(1000);
        s1.interrupt();     //中断 s1 线程
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main End");
    }
}
```
```
Thread Name: main, Main Start
Thread Name: Thread-0, Sleep Start
Thread Name: Thread-0, Sleep End
Thread Name: Thread-0, i = 2147483647
Thread Name: main, Main End
```

运行该程序会发现，while 循环会一直运行直到变量 i 的值超出 Integer.MAX_VALUE 。所以说直接调用 interrupt 方法不能中断正在运行中的线程。

但是如果配合 isInterrupted() 能够中断正在运行的线程，因为调用 interrupt 方法相当于将中断标志位置为 true，那么可以通过调用 isInterrupted() 判断中断标志是否被置位来中断线程的执行。
```java
public class InterruptTest {
    class InterruptThread extends Thread {
        @Override
        public void run() {
            System.out.println("Thread Name: " + this.getName() + ", Sleep Start");
            int i = 0;
            while (!isInterrupted() && i < Integer.MAX_VALUE) {
                i++;
            }
            System.out.println("Thread Name: " + this.getName() + ", Sleep End");
            System.out.println("Thread Name: " + this.getName() + ", i = " + i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main Start");
        InterruptTest test = new InterruptTest();
        InterruptTest.InterruptThread s1 = test.new InterruptThread();
        s1.start();
        Thread.sleep(1000);
        s1.interrupt();     //中断 s1 线程
        System.out.println("Thread Name: " + Thread.currentThread().getName() + ", Main End");
    }
}
```
```
Thread Name: main, Main Start
Thread Name: Thread-0, Sleep Start
Thread Name: main, Main End
Thread Name: Thread-0, Sleep End
Thread Name: Thread-0, i = 939166599
```
运行会发现，i 累加到某个值，while 循环就停止了。
### 7、interrupted()
interrupted() 函数是 Thread 静态方法，用来检测当前线程的 interrupt 状态，检测完成后，状态清空。通过下面的 interrupted 源码我们能够知道，此方法首先调用 isInterrupted 方法，而 isInterrupted 方法是一个重载的 native 方法 private native boolean isInterrupted(boolean ClearInterrupted) 通过方法的注释能够知道，用来测试线程是否已经中断，参数用来决定是否重置中断标志。

### 8、stop()
stop 方法已经是一个废弃的方法，它是一个不安全的方法。因为调用 stop 方法会直接终止 run 方法的调用，并且会抛出一个 ThreadDeath 错误，如果线程持有某个对象锁的话，会完全释放锁，导致对象状态不一致。所以 stop 方法基本是不会被用到的。
### 9、destroy()
destroy 方法也是废弃的方法。基本不会被使用到。

### 参考：
- [java.lang.Thread类详解](https://www.cnblogs.com/albertrui/p/8391447.html)
