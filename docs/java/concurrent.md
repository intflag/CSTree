# Java 并发
> 进程是资源分配的最小单位，线程是 CPU 调度的最小单位。

一个简单比喻：进程 = 火车，线程 = 车厢

- 线程在进程下行进（单纯的车厢无法运行）
- 一个进程可以包含多个线程（一辆火车可以有多个车厢）
- 不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘），同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）
- 进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）
- 进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）
- 进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）
- 进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"
- 进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”

参考
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

### 参考
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
实现 Callable 接口，实现 call 方法并且指定返回值类型，但是在启动时要注意，需要使用 FutureTask 包装一下，然后新建线程启动，因为 Thread 的构造方法接收的是 Runnable，而 FutureTask 间接实现了 Runnable 接口，最后线程执行的结果可以通过 FutureTask 的 get 方法获取到。

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

如果需要让当前正在执行的线程暂停一段时间，并进入阻塞状态，则可以通过调用 Thread 类的静态 sleep() 方法来实现。

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
假如在 main 线程中，调用 thread.join 方法，则 main 方法会等待 thread 线程执行完毕或者等待一定的时间。如果调用的是无参 join 方法，则等待 thread 执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的时间。

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

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```
```java
/**
* Tests if some Thread has been interrupted.  The interrupted state
* is reset or not based on the value of ClearInterrupted that is
* passed.
*/
private native boolean isInterrupted(boolean ClearInterrupted);
```


### 8、stop()
stop 方法已经是一个废弃的方法，它是一个不安全的方法。因为调用 stop 方法会直接终止 run 方法的调用，并且会抛出一个 ThreadDeath 错误，如果线程持有某个对象锁的话，会完全释放锁，导致对象状态不一致。所以 stop 方法基本是不会被用到的。
### 9、destroy()
destroy 方法也是废弃的方法。基本不会被使用到。

### 参考
- [java.lang.Thread类详解](https://www.cnblogs.com/albertrui/p/8391447.html)

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
Executors 是一个工具类，类似于 Collections。 提供工厂方法来创建不同类型的线程池。

![](http://images.intflag.com/executor03.png)

**1）Executors 四种线程池**

| 线程池              | 功能                                                         |
| ----------------------- | ------------------------------------------------------------ |
| newCachedThreadPool     | 可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。 |
| newFixedThreadPool      | 定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。 |
| newScheduledThreadPool  | 定长线程池，支持定时及周期性任务执行。                       |
| newSingleThreadExecutor | 单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序（FIFO, LIFO 优先级）执行。 |

**2）生产实践**

在《阿里巴巴Java开发手册》中强制规定：
- 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。因为线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换” 的问题。
- 线程池不允许使用 Executors 去创建，而是通过 `ThreadPoolExecutor` 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。并且 Executors 返回的线程池对象具有弊端，FixedThreadPool 和 SingleThreadPool 允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM，而CachedThreadPool 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，也会导致 OOM。

### 6、Executor 的中断操作
调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
```
```java
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```
如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

### 参考
- [漫谈JAVA之Executor框架(1)](https://www.jianshu.com/p/e2053d455ef3)
- [Difference between a Thread and an Executor in Java](https://javarevisited.blogspot.com/2016/12/difference-between-thread-and-executor.html)

## 互斥同步
Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

### 1、synchronized
**1）同步代码块**
```java
public void fun() {
    synchronized (this) {
        // ...
    }
}
```
它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedTest {

    public void fun1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.println("Thread Name: " + Thread.currentThread().getName() + ", i = " + i);
            }
        }
    }

    public static void main(String[] args) {
        SynchronizedTest sync1 = new SynchronizedTest();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> sync1.fun1());
        executorService.execute(() -> sync1.fun1());
    }
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```
对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。
```java
public static void main(String[] args) {
    SynchronizedTest sync1 = new SynchronizedTest();
    SynchronizedTest sync2 = new SynchronizedTest();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> sync1.fun1());
    executorService.execute(() -> sync2.fun1());
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```

**2）同步方法**
```java
public synchronized void fun () {
    // ...
}
```
它和同步代码块一样，作用于同一个对象。

**3）同步类**
```java
public void fun() {
    synchronized (SynchronizedTest.class) {
        // ...
    }
}
```
作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。
```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```
```java
public static void main(String[] args) {
    SynchronizedTest sync1 = new SynchronizedTest();
    SynchronizedTest sync2 = new SynchronizedTest();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> sync1.fun2());
    executorService.execute(() -> sync2.fun2());
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```
**4）同步静态方法**
```java
public synchronized static void fun() {
    // ...
}
```
作用于整个类。

### 2、ReentrantLock
ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```java
public class ReentrantLockTest {

    private ReentrantLock lock = new ReentrantLock();

    public void fun1() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.println("Thread Name: " + Thread.currentThread().getName() + ", i = " + i);
            }
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ReentrantLockTest sync1 = new ReentrantLockTest();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(()->sync1.fun1());
        executorService.execute(()->sync1.fun1());
    }
}
```
```
Thread Name: pool-1-thread-1, i = 0
Thread Name: pool-1-thread-1, i = 1
Thread Name: pool-1-thread-1, i = 2
Thread Name: pool-1-thread-1, i = 3
Thread Name: pool-1-thread-1, i = 4
Thread Name: pool-1-thread-1, i = 5
Thread Name: pool-1-thread-1, i = 6
Thread Name: pool-1-thread-1, i = 7
Thread Name: pool-1-thread-1, i = 8
Thread Name: pool-1-thread-1, i = 9
Thread Name: pool-1-thread-2, i = 0
Thread Name: pool-1-thread-2, i = 1
Thread Name: pool-1-thread-2, i = 2
Thread Name: pool-1-thread-2, i = 3
Thread Name: pool-1-thread-2, i = 4
Thread Name: pool-1-thread-2, i = 5
Thread Name: pool-1-thread-2, i = 6
Thread Name: pool-1-thread-2, i = 7
Thread Name: pool-1-thread-2, i = 8
Thread Name: pool-1-thread-2, i = 9
```

### 3、synchronized 与 ReentrantLock 比较
**1）锁的实现**

synchronized 是 JVM 实现的，ReentrantLock 是 JDK 实现的。

**2）性能**

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**3）等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4）公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5）锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

### 4、使用选择
除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## 线程之间的协作
当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

### 1、join
在线程中调用另一个线程的 join() 方法，会将当前线程挂起，知道目标线程结束，该方法是 Thread 类的方法。

### 2、wait() notify() notifyAll()
调用 wait() 使线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使这个条件满足时，其他线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

他们都属于 Object 的一部分，不属于 Thread。

只能用在同步方法或者同步代码块中，否则会在运行时抛出 IllegalMonitorStateException 非法监视状态异常。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其他线程就无法进入对象的同步方法或者同步代码块中，也就无法执行 notify() 或者 notifyAll() 方法来唤醒挂起的线程，从而造成死锁。

```java
public class WaitTest {

    public synchronized void before() {
        System.out.println("Thread Name: " + Thread.currentThread().getName()+" before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
            System.out.println("Thread Name: " + Thread.currentThread().getName()+" after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        WaitTest waitTest = new WaitTest();
        executorService.execute(()->waitTest.after());
        executorService.execute(()->waitTest.before());

    }
}
```
```
Thread Name: pool-1-thread-2 before
Thread Name: pool-1-thread-1 after
```

wait() 和 sleep() 的区别
- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法。
- wait() 会释放锁，sleep() 不会释放锁。

### 3、await() signal() signalAll()
java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其他线程上调用 signal() 或者 signalAll() 方法唤醒等待的线程。

相比与 wait() 方法，await() 方法可以指定等待的条件，因此更加灵活。

```java
public class AwaitTest {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("Thread Name: " + Thread.currentThread().getName() + " before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("Thread Name: " + Thread.currentThread().getName() + " after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        AwaitTest awaitTest = new AwaitTest();
        executorService.execute(() -> awaitTest.after());
        executorService.execute(() -> awaitTest.before());
    }
}
```
```java
Thread Name: pool-1-thread-2 before
Thread Name: pool-1-thread-1 after
```

## J.U.C - AQS
AQS：AbstractQuenedSynchronizer抽象的队列式同步器。是除了java自带的synchronized关键字之外的锁机制。

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。

### 1、CountDownLatch
用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

```java
public class CountDownLatchTest {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.println("Thread Name: " + Thread.currentThread().getName() + " Run...");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("Thread Name: " + Thread.currentThread().getName() + " End");
    }
}
```
```
Thread Name: pool-1-thread-1 Run...
Thread Name: pool-1-thread-2 Run...
Thread Name: pool-1-thread-3 Run...
Thread Name: pool-1-thread-4 Run...
Thread Name: pool-1-thread-5 Run...
Thread Name: pool-1-thread-6 Run...
Thread Name: pool-1-thread-7 Run...
Thread Name: pool-1-thread-8 Run...
Thread Name: pool-1-thread-9 Run...
Thread Name: pool-1-thread-10 Run...
Thread Name: main End
```

### 2、CyclicBarrier
用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

```java
public class CyclicBarrierTest {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(()->{
                System.out.println("Thread Name: " + Thread.currentThread().getName() + " brfore...");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread Name: " + Thread.currentThread().getName() + " after...");
            });
        }
    }
}
```
```
Thread Name: pool-1-thread-1 brfore...
Thread Name: pool-1-thread-2 brfore...
Thread Name: pool-1-thread-3 brfore...
Thread Name: pool-1-thread-4 brfore...
Thread Name: pool-1-thread-5 brfore...
Thread Name: pool-1-thread-6 brfore...
Thread Name: pool-1-thread-7 brfore...
Thread Name: pool-1-thread-8 brfore...
Thread Name: pool-1-thread-9 brfore...
Thread Name: pool-1-thread-10 brfore...
Thread Name: pool-1-thread-10 after...
Thread Name: pool-1-thread-1 after...
Thread Name: pool-1-thread-2 after...
Thread Name: pool-1-thread-3 after...
Thread Name: pool-1-thread-4 after...
Thread Name: pool-1-thread-5 after...
Thread Name: pool-1-thread-6 after...
Thread Name: pool-1-thread-7 after...
Thread Name: pool-1-thread-9 after...
Thread Name: pool-1-thread-8 after...
```

### 3、Semaphore
Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

以下代码模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。

```java
public class SemaphoreTest {

    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        ExecutorService executorService = Executors.newCachedThreadPool();
        Semaphore semaphore = new Semaphore(clientCount);
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    System.out.println("Thread Name: " + Thread.currentThread().getName() + " "+semaphore.availablePermits()+" Run...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
    }
}
```
```
Thread Name: pool-1-thread-1 2 Run...
Thread Name: pool-1-thread-2 2 Run...
Thread Name: pool-1-thread-3 2 Run...
Thread Name: pool-1-thread-4 1 Run...
Thread Name: pool-1-thread-5 2 Run...
Thread Name: pool-1-thread-6 2 Run...
Thread Name: pool-1-thread-7 1 Run...
Thread Name: pool-1-thread-8 2 Run...
Thread Name: pool-1-thread-9 2 Run...
Thread Name: pool-1-thread-10 2 Run...
```

## J.U.C - 其它组件
### 1、FutureTask
在介绍 Callable 时我们知道它可以有返回值，返回值通过 Future 进行封装。FutureTask 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。

FutureTask 可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

### 2、BlockingQueue
java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

FIFO 队列 ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
优先级队列 ：PriorityBlockingQueue
提供了阻塞的 take() 和 put() 方法：如果队列为空 take() 将阻塞，直到队列中有内容；如果队列为满 put() 将阻塞，直到队列有空闲位置。

**使用 BlockingQueue 实现生产者消费者问题**

```java
public class BlockQueueTest {

    private static BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(5);

    public static class Producer extends Thread {
        @Override
        public void run() {
            try {
                blockingQueue.put(this.getName() + " produce");
                System.out.println(this.getName() + " produce...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static class Consumer extends Thread {
        @Override
        public void run() {
            try {
                String take = blockingQueue.take();
                System.out.println(this.getName() + " consume: " + take);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }
        for (int i = 0; i < 5; i++) {
            Consumer consumer = new Consumer();
            consumer.start();
        }
        for (int i = 0; i < 3; i++) {
            Producer producer = new Producer();
            producer.start();
        }
    }
}
```
```
Thread-0 produce...
Thread-1 produce...
Thread-2 consume: Thread-0 produce
Thread-3 consume: Thread-1 produce
Thread-7 produce...
Thread-4 consume: Thread-7 produce
Thread-5 consume: Thread-8 produce
Thread-6 consume: Thread-9 produce
Thread-8 produce...
Thread-9 produce...
```

### 3、ForkJoin
主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```java
public class ForkJoinTest extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinTest(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            System.out.println(Thread.currentThread().getName() + " first = " + first + " last = " + last);
            //任务范围小于临界值时直接进行计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            //否则拆分成小任务进行计算
            int middle = first + ((last - first) >> 1);
            ForkJoinTest leftForkJoinTest = new ForkJoinTest(first, middle);
            ForkJoinTest rightForkJoinTest = new ForkJoinTest(middle + 1, last);
            leftForkJoinTest.fork();
            rightForkJoinTest.fork();
            result = leftForkJoinTest.join() + rightForkJoinTest.join();
        }
        return result;
    }

    public static void main(String[] args) {
        ForkJoinTest forkJoinTest = new ForkJoinTest(1, 100);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        forkJoinPool.submit(forkJoinTest);
        try {
            System.out.println(forkJoinTest.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```
```
ForkJoinPool-1-worker-2 first = 1 last = 4
ForkJoinPool-1-worker-2 first = 5 last = 7
ForkJoinPool-1-worker-2 first = 8 last = 13
ForkJoinPool-1-worker-2 first = 14 last = 19
ForkJoinPool-1-worker-2 first = 20 last = 25
ForkJoinPool-1-worker-2 first = 26 last = 29
ForkJoinPool-1-worker-2 first = 30 last = 32
ForkJoinPool-1-worker-2 first = 33 last = 38
ForkJoinPool-1-worker-2 first = 39 last = 44
ForkJoinPool-1-worker-2 first = 45 last = 50
ForkJoinPool-1-worker-3 first = 51 last = 54
ForkJoinPool-1-worker-2 first = 76 last = 79
ForkJoinPool-1-worker-2 first = 80 last = 82
ForkJoinPool-1-worker-2 first = 83 last = 88
ForkJoinPool-1-worker-2 first = 89 last = 94
ForkJoinPool-1-worker-2 first = 95 last = 100
ForkJoinPool-1-worker-2 first = 64 last = 69
ForkJoinPool-1-worker-2 first = 70 last = 75
ForkJoinPool-1-worker-2 first = 58 last = 63
ForkJoinPool-1-worker-2 first = 55 last = 57
5050
```
ForkJoin 使用 ForkJoinPool 来启动，它是一个特殊的线程池，线程数量取决于 CPU 核数。
```java
public class ForkJoinPool extends AbstractExecutorService
```
ForkJoinPool 实现了工作窃取算法来提高 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。例如下图中，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。但是如果队列中只有一个任务时还是会发生竞争。

## Java 的内存模型
### 0、计算机中的木桶理论
计算机的 CPU、内存、I/O 设备的性能随着技术发展变得越来越快，但是一直有个核心矛盾存在，那就是这三者的速度差异，如果我们把 CPU 执行一条普通指令看做「天上一天」的话，那 CPU 读写内存的速度就是「地上一年」，而内存和 I/O 设备的速度差异更大，内存是「天上一天」，I/O 设备是「地上十年」。

根据「木桶理论」，程序的性能取决于最慢的操作，那就是读写 I/O 设备，也就是说单方面提高 CPU 的性能是无效的。

为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为：
- CPU 增加了缓存，以均衡与内存的速度差异；
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；
- 编译程序优化指令执行次序，使缓存能否得到更加合理地利用。

天下没有免费的午餐，并发程序很多诡异问题的根源也在这里。

### 1、主内存与工作内存
处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

![](http://images.intflag.com/concurrent02.png)

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![](http://images.intflag.com/concurrent03.png)

### 2、内存间交互操作
Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![](http://images.intflag.com/concurrent04.png)

- read（读取）：把一个变量的值从主内存传输到工作内存中；
- load（加载）：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中；
- use（使用）：把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时就会执行该操作；
- assign（赋值）：把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时就会执行该操作；
- store（存储）：把工作内存的一个变量的值传送到主内存中；
- write（写入）：在 store 之后执行，把 store 得到的值放入主内存的变量中；
- lock（锁定）：作用于主内存的变量，一个变量在同一时间只能一个线程锁定，该操作表示这条线成独占这个变量；
- unlock（解锁）：作用于主内存的变量，表示这个变量的状态由处于锁定状态被释放，这样其他线程才能对该变量进行锁定。

### 3、内存模型三大特性

**1）缓存导致的可见性问题**

> 一个线程对共享变量的修改，另外一个线程能够立即看到的特性，称为**可见性**。

在多核时代，每颗 CPU 都有自己的缓存，多个线程在不同 CPU 上执行时，操作的是不同的 CPU 缓存。

![多核 CPU 的缓存与内存关系图](http://images.intflag.com/mem01.png)

如上图所示，线程 A 操作的是 CPU-1 的缓存，而线程 B 操作的是 CPU-2 的缓存，这个时候线程 A 对变量 V 的操作对线程 B 而言就不具备可见性了。

验证多核场景下的可见性问题：
```java
public class VisibilityTest {
    public long count = 0;

    public void add10K() {
        int i = 0;
        while (i++ < 10000) {
            count += 1;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        VisibilityTest visibilityTest = new VisibilityTest();
        //创建两个线程，执行 add10K 操作
        Thread t1 = new Thread(() -> visibilityTest.add10K());
        Thread t2 = new Thread(() -> visibilityTest.add10K());
        //启动两个线程
        t1.start();
        t2.start();
        //等待两个程序结束
        t1.join();
        t2.join();
        System.out.println(visibilityTest.count);
    }
}
```
```
16438
```
累加一万次结果在1万到2万之间。

我们假设线程 A 和线程 B 同时开始执行，那么第一次都会将 count=0 读到各自的 CPU 缓存里，执行完 count+=1 之后，各自 CPU 缓存里的值都是 1，同时写入内存后，我们会发现内存中是 1，而不是我们期望的 2。之后由于各自的 CPU 缓存里都有了 count 的值，两个线程都是基于 CPU 缓存里的 count 值来计算，所以导致最终 count 的值都是小于 20000 的。这就是缓存的可见性问题。
```
100007570
```
改为累加一亿次效果更为明显，最终 count 的值接近 1 亿，而不是 2 亿。如果循环 10000 次，count 的值接近 20000，原因是两个线程不是同时启动的，有一个时差。

**避免这种问题：**

- **使用原子类**
```java
public class VisibilityTest {

    public AtomicLong count = new AtomicLong();

    public void add10K() {
        int i = 0;
        while (i++ < 100000000) {
            count.addAndGet(1);
        }
    }
}
```
- **使用 synchronized 互斥锁**
```java
public class VisibilityTest {

    public long count = 0;

    public synchronized void add10K() {
        int i = 0;
        while (i++ < 100000000) {
            count += 1;
        }
    }
}
```
- **使用 ReentrantLock 加锁**
```java
public class VisibilityTest {

    private long count = 0;

    private Lock lock = new ReentrantLock();

    public void add10K() {
        lock.lock();
        try {
            int i = 0;
            while (i++ < 100000000) {
                count += 1;
            }
        } finally {
            lock.unlock();
        }
    }
}
```
经过验证，使用原子类大概耗时 2400ms 左右，使用 synchronized 和 ReentrantLock 基本都在 90ms 左右。

**2）线程切换带来的原子性问题**

> 一个或者多个操作在 CPU 执行的过程中不被中断的特性，称为原子性。

Java 并发程序都是基于多线程的，自然也会涉及到任务切换，也许你想不到，任务切换竟然也是并发编程里诡异 Bug 的源头之一。任务切换的时机大多数是在时间片结束的时候，我们现在基本都使用高级语言编程，高级语言里一条语句往往需要多条 CPU 指令完成，例如上面代码中的count += 1，至少需要三条 CPU 指令。

- 指令 1：首先，需要把变量 count 从内存加载到 CPU 的寄存器；
- 指令 2：之后，在寄存器中执行 +1 操作；
- 指令 3：最后，将结果写入内存（缓存机制导致可能写入的是 CPU 缓存而不是内存）。

操作系统做任务切换，可以发生在任何一条 CPU 指令执行完，是的，是 CPU 指令，而不是高级语言里的一条语句。对于上面的三条指令来说，我们假设 count=0，如果线程 A 在指令 1 执行完后做线程切换，线程 A 和线程 B 按照下图的序列执行，那么我们会发现两个线程都执行了 count+=1 的操作，但是得到的结果不是我们期望的 2，而是 1。

![非原子操作的执行路径示意图](http://images.intflag.com/mem02.png)

我们潜意识里面觉得 count+=1 这个操作是一个不可分割的整体，就像一个原子一样，线程的切换可以发生在 count+=1 之前，也可以发生在 count+=1 之后，但就是不会发生在中间。我们把一个或者多个操作在 CPU 执行的过程中不被中断的特性称为原子性。CPU 能保证的原子操作是 CPU 指令级别的，而不是高级语言的操作符，这是违背我们直觉的地方。因此，很多时候我们需要在高级语言层面保证操作的原子性。

**3、编译优化带来的有序性问题**

> 程序按照代码的先后顺序执行，称为有序性。

编译器为了优化性能，有时候会改变程序中语句的先后顺序，例如程序中：“a=6；b=7；”编译器优化后可能变成“b=7；a=6；”，在这个例子中，编译器调整了语句的顺序，但是不影响程序的最终结果。不过有时候编译器及解释器的优化可能导致意想不到的 Bug。

在 Java 领域一个经典的案例就是利用双重检查创建单例对象，例如下面的代码：在获取实例 getInstance() 的方法中，我们首先判断 instance 是否为空，如果为空，则锁定 Singleton.class 并再次检查 instance 是否为空，如果还为空则创建 Singleton 的一个实例。

```java
public class Singleton {
  static Singleton instance;
  static Singleton getInstance(){
    if (instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    }
    return instance;
  }
}
```
假设有两个线程 A、B 同时调用 getInstance() 方法，他们会同时发现 instance == null ，于是同时对 Singleton.class 加锁，此时 JVM 保证只有一个线程能够加锁成功（假设是线程 A），另外一个线程则会处于等待状态（假设是线程 B）；线程 A 会创建一个 Singleton 实例，之后释放锁，锁释放后，线程 B 被唤醒，线程 B 再次尝试加锁，此时是可以加锁成功的，加锁成功后，线程 B 检查 instance == null 时会发现，已经创建过 Singleton 实例了，所以线程 B 不会再创建一个 Singleton 实例。

这看上去一切都很完美，无懈可击，但实际上这个 getInstance() 方法并不完美。问题出在哪里呢？出在 new 操作上，我们以为的 new 操作应该是：
- 分配一块内存 M；
- 在内存 M 上初始化 Singleton 对象；
- 然后 M 的地址赋值给 instance 变量。

但是实际上优化后的执行路径却是这样的：
- 分配一块内存 M；
- 将 M 的地址赋值给 instance 变量；
- 最后在内存 M 上初始化 Singleton 对象。

优化后会导致什么问题呢？我们假设线程 A 先执行 getInstance() 方法，当执行完指令 2 时恰好发生了线程切换，切换到了线程 B 上；如果此时线程 B 也执行 getInstance() 方法，那么线程 B 在执行第一个判断时会发现 instance != null ，所以直接返回 instance，而此时的 instance 是没有初始化过的，如果我们这个时候访问 instance 的成员变量就可能触发空指针异常。

![双重检查创建单例的异常执行路径](http://images.intflag.com/mem03.png)

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

### Happens-Before 原则
如何理解 Happens-Before 呢？如果望文生义（很多网文也都爱按字面意思翻译成“先行发生”），那就南辕北辙了，Happens-Before 并不是说前面一个操作发生在后续操作的前面，它真正要表达的是：`前面一个操作的结果对后续操作是可见的`。就像有心灵感应的两个人，虽然远隔千里，一个人心之所想，另一个人都看得到。Happens-Before 规则就是要保证线程之间的这种“心灵感应”。所以比较正式的说法是：Happens-Before 约束了编译器的优化行为，虽允许编译器优化，但是要求编译器优化后一定遵守 Happens-Before 规则。

**1）单一线程原则**

> Single Thread rule

在一个线程内，在程序前面的操作先行发生于后面的操作。

![](http://images.intflag.com/concurrent05.png)

**2）管程锁定规则**

> Monitor Lock Rule

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

![](http://images.intflag.com/concurrent06.png)

**3）volatile 变量规则**

> Volatile Variable Rule

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

![](http://images.intflag.com/concurrent07.png)

**4）线程启动规则**

> Thread Start Rule

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

![](http://images.intflag.com/concurrent08.png)

**5）线程加入规则**

> Thread Join Rule

Thread 对象的结束先行发生于 join() 方法返回。

![](http://images.intflag.com/concurrent09.png)

**6）线程中断规则**

> Thread Interruption Rule

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

**7）对象终结规则**

> Finalizer Rule

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

**8）传递性**

> Transitivity

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

## 线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

### 1、不可变
不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。

```java
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```
```java
Exception in thread "main" java.lang.UnsupportedOperationException
    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
    at ImmutableExample.main(ImmutableExample.java:9)
```
Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。
```java
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```
### 2、互斥同步
synchronized 和 ReentrantLock。
### 3、非阻塞同步
互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

**1）CAS**

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

**2）AtomicInteger**

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码使用了 AtomicInteger 执行了自增的操作。

```java
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.incrementAndGet();
}
```
以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。
```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```
以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。
```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

**3）ABA**

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

### 4、无同步方案
要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

**1）栈封闭**

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

```java
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}
```
```java
public static void main(String[] args) {
    StackClosedExample example = new StackClosedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}
```
```java
100
100
```

**2、线程本地存储（Thread Local Storage）**

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。

```java
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}
```
```java
1
```
为了理解 ThreadLocal，先看以下代码：
```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```
它所对应的底层结构图为：

![](http://images.intflag.com/concurrent10.png)

每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```
当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
get() 方法类似。
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

**3）可重入代码（Reentrant Code）**

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。



