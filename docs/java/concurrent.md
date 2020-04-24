# Java 并发
## 多线程的使用
实现多线程有三种方式：
- 实现 Runnable 接口
- 实现 Callable 接口
- 继承 Thread 类

实现 Runnable 接口和 Callable 接口的类并没有 start() 方法，不是真正意义上的线程，也不可能直接调用 run() 方法，所以必须使用 new Thread(xxx).start() 的方式开启线程，这两种方式可以理解为任务是由线程驱动而执行的。

### 1、实现 Runnable 接口
实现 Runnable 接口，然后实现 run 方法。
```
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
```
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```
FutureTask 类关系：
![](http://images.intflag.com/concurrent01.png)

案例：
```
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
```
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
```
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

参考：[漫谈JAVA之Executor框架(1)](https://www.jianshu.com/p/e2053d455ef3)
