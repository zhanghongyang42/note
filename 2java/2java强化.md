# 序列化



# 反射





# 多线程

开始 cpu一个核心开一个线程。为了提高cpu处理能力，cpu上可以有**多个核**心，intel cpu 也可以**一个核心**开**2个线程**。



线程：cpu调度的最小单位。

进程：是操作系统分配资源的基本单位，一个进程可以有多个线程，线程之间资源共享。

并发：单核cpu运行多线程时，时间片进行很快的切换。线程轮流执行cpu

并行：多核cpu运行 多线程时，真正的在同一时刻运行



所以想要程序运行的快，**多线程**可以充分利用cpu多核资源。



### 线程使用

线程定义的三种方式

```java
// 继承Thread类,任务和线程合并在一起
class T extends Thread {
    @Override
    public void run() {
        log.info("我是继承Thread的任务");
    }
}

//实现Runnable接口,解耦了线程与任务，这里定义任务。
class R implements Runnable {
    @Override
    public void run() {
        log.info("我是实现Runnable的任务");
    }
}

//实现Callable接口，相比Runnable，多了抛出异常和返回值
class C implements Callable<String> {
    @Override
    public String call() throws Exception {
        log.info("我是实现Callable的任务");
        return "success";
    }
}
```

线程创建

```java
// 启动继承Thread类的任务
new T().start();

//  启动实现Runnable接口的任务
new Thread(new R()).start();

// 启动实现了Callable接口的任务 结合FutureTask 可以获取线程执行的结果
FutureTask<String> target = new FutureTask<>(new C());
new Thread(target).start();
log.info(target.get());
```

线程暂停和优先级

```java
class r1 implements Runnable {
    @Override
    public void run() {
        int count = 0;
        for (;;){
            log.info("---- 1>" + count++);}}}

class r2 implements Runnable {
    @Override
    public void run() {
        int count = 0;
        for (;;){
            //yield()会让线程暂停，并等待重新分配
            Thread.yield();	//public static native void yield(); 
            log.info("---- 2>" + count++);}}}

Thread t1 = new Thread(r1,"t1");
Thread t2 = new Thread(r2,"t2");
t1.setPriority(Thread.NORM_PRIORITY);	//设置线程优先级
t2.setPriority(Thread.MAX_PRIORITY);
t1.start();
t2.start();
```



### 线程状态

![img](picture/v2-6a0f728bfa4068edeb5d9dc29a1884b9_720w.jpg)

1. 初始状态：创建线程对象时的状态
2. 可运行状态(就绪状态)：调用start()方法后进入就绪状态，也就是准备好被cpu调度执行
3. 运行状态：线程获取到cpu的时间片，执行run()方法的逻辑
4. 阻塞状态: 线程被阻塞，放弃cpu的时间片，等待解除阻塞重新回到就绪状态争抢时间片
5. 终止状态: 线程执行完成或抛出异常后的状态



### 线程阻塞

BIO阻塞，即使用了阻塞式的io流

获得锁之后调用wait()方法 也会让线程进入阻塞状态  (同步锁章节细说)

LockSupport.park() 让线程进入阻塞状态  (同步锁章节细说)



sleep()

使线程休眠，会将运行中的线程进入阻塞状态。当休眠时间结束后，重新争抢cpu的时间片继续运行。

```java
try{
    Thread.sleep(2000);
}catch (InterruptedException异常 e) {
}
```



join()

join是指调用该方法的线程进入阻塞状态，等待某线程执行完成后恢复运行

```java
public class Parent {
    public static void main(String[] args) {
    Child child = new Child(); // 创建child线程对象
    child.start();
    child.join(); // 等待child线程运行完主线程再继续运行
    }
}
```



### 同步锁

线程安全

```
线程安全：多线程同时读写共享属性时，属性值一定不会发生错误，就是保证了线程安全。

线程安全的类中每一个独立的方法是线程安全的，但是方法的组合就不一定是线程安全的。
```



同步锁是保证线程安全的，同步锁是锁在对象上的，不同的对象就是不同的锁。

同一个时刻最多只有一个线程能持有对象锁，其他线程在想获取这个对象锁就会被阻塞。

```java
 // 加在方法上 实际是对this对象加锁
private synchronized void a() {
}

// 加在静态方法上 实际是对类对象加锁
private synchronized static void c() {
}
```

```java
private static int count = 0;
private static Object lock = new Object();

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            synchronized (lock) {
                count++;
            }
        }
    });
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 5000; i++) {
            synchronized (lock) {
                count--;
            }
        }
    });
 
    t1.start();
    t2.start();
    
    t1.join();
    t2.join();
    System.out.println(count);
}
```



### 线程池

线程池就是一种管理线程的工具，节约创建线程的时间，控制线程的数量。



线程池类

```java
public ThreadPoolExecutor(int corePoolSize,						//核心线程数
                          int maximumPoolSize,					//最大线程数 = 核心线程数 + 救急线程
                          long keepAliveTime,					//救急线程的空闲时间，哪个线程空闲了就是救急线程
                          TimeUnit unit,						//救急线程的空闲时间单位
                          BlockingQueue<Runnable> workQueue,	//阻塞队列
                          ThreadFactory threadFactory,			//线程工厂
                          RejectedExecutionHandler handler) {}	//拒绝策略
```

任务先来到阻塞队列，然后核心线程开始执行。

阻塞队列满后，救急线程开始执行。

所有线程和阻塞队列都满后，执行拒绝策略。

救急线程达到空闲时间时，还没有任务，会被回收。



线程参数配置

![preview](picture/v2-8ed95255ed320e9fa4b27c7c676c8f08_r.jpg)



https://zhuanlan.zhihu.com/p/257088648



# JVM优化

# 设计模式

可以单独一个文件



线程与并发

网络编程



MINA

Netty

NIO































