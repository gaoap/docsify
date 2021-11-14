# Effective Java学习笔记（十）：并发

## 1、同步访问共享的可变数据

示例：错误示例：

```java
public class StopThread {
    /**
     *  stopRequested虽然是院子变量，但是因为没有同步synchronized的限制，
     *  stopRequested变更后，什么时候，线程能够观察到确是未知的
     */
   
    private static boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        //这里虽然设置了线程终止，但是因为没有同步synchronized的缘故，线程会一直执行下去，而不会终止。
        stopRequested = true;
    }
}
```

修正错误，但是开销会比较大：

```java
public class StopThread {
    //boolean是原子变量
    private static boolean stopRequested;
    //增加synchronized后，会按照预想执行，并终止程序。但是开销比较大
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    //都要增加synchronized。
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        //会如期结束线程
        requestStop();
    }
}
```

减少线程线程同步开销的方法：

```java
public class StopThread {
    /**
     * volatile保证线程看到最新刚刚修改的变化
     * volatile不会执行互斥访问
     */
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}

```

注意，这种方式也是错误错的。如：

```java
  private static volatile int nextNumber;
  //nextNumber是原子的。但是++不是。所以 getNext()非线程安全
  public static int getNext(){
      return nextNumber++;
  }
改正：
  private static final AtomicLong nextNumber=new AtomicLong();
    public static long getNext(){
        return nextNumber.getAndIncrement();
    }
```

## 2、避免过度同步

补充知识：

一、CopyOnWriteArrayList介绍
①、CopyOnWriteArrayList，写数组的拷贝，支持高效率并发且是线程安全的,读操作无锁的ArrayList。所有可变操作都是通过对底层数组进行一次新的复制来实现。
②、CopyOnWriteArrayList适合使用在读操作远远大于写操作的场景里，比如缓存。它不存在扩容的概念，每次写操作都要复制一个副本，在副本的基础上修改后改变Array引用。CopyOnWriteArrayList中写操作需要大面积复制数组，所以性能肯定很差。
③、CopyOnWriteArrayList 合适读多写少的场景，不过这类慎用 ，因为谁也没法保证CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。

使用示例：

```java
原代码：
    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }
调整后：
  private final List<SetObserver<E>> observers =
            new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        observers.add(observer);
    }

    public boolean removeObserver(SetObserver<E> observer) {
        return observers.remove(observer);
    }
```

## 3、executor、task和stream优先于线程

## 4、并发工具优先于wait和notify

并发map:

```java
public class Intern {
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();
//    效率较慢
//    public static String intern(String s) {
//        String previousValue = map.putIfAbsent(s, s);
//        return previousValue == null ? s : previousValue;
//    }

    // 先判断，这样使用效率更高
    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null)
                result = s;
        }
        return result;
    }
}
```

最常用的同步器是 CountDownLatch 和 Semaphore 。 较不常用的是 CyclicBarrier 和Exchanger 。 功能最强大的同步器是 Phaser 。  

**CountDownLatch使用举例：**

```java
/**
 * 主线程等待子线程执行完成再执行
 */
public class CountdownLatchTest1 {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(3);
        final CountDownLatch latch = new CountDownLatch(3);
        for (int i = 0; i < 3; i++) {
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    try {
                        System.out.println("子线程" + Thread.currentThread().getName() + "开始执行");
                        Thread.sleep((long) (Math.random() * 10000));
                        System.out.println("子线程"+Thread.currentThread().getName()+"执行完成");
                        latch.countDown();//当前线程调用此方法，则计数减一
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            service.execute(runnable);
        }

        try {
            System.out.println("主线程"+Thread.currentThread().getName()+"等待子线程执行完成...");
            latch.await();//阻塞当前线程，直到计数器的值为0
            System.out.println("主线程"+Thread.currentThread().getName()+"开始执行...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class CountdownLatchTest2 {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        final CountDownLatch cdOrder = new CountDownLatch(1);
        final CountDownLatch cdAnswer = new CountDownLatch(4);
        for (int i = 0; i < 4; i++) {
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    try {
                        System.out.println("选手" + Thread.currentThread().getName() + "正在等待裁判发布口令");
                        cdOrder.await();
                        System.out.println("选手" + Thread.currentThread().getName() + "已接受裁判口令");
                        Thread.sleep((long) (Math.random() * 10000));
                        System.out.println("选手" + Thread.currentThread().getName() + "到达终点");
                        cdAnswer.countDown();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            service.execute(runnable);
        }
        try {
            Thread.sleep((long) (Math.random() * 10000));
            System.out.println("裁判"+Thread.currentThread().getName()+"即将发布口令");
            cdOrder.countDown();
            System.out.println("裁判"+Thread.currentThread().getName()+"已发送口令，正在等待所有选手到达终点");
            cdAnswer.await();
            System.out.println("所有选手都到达终点");
            System.out.println("裁判"+Thread.currentThread().getName()+"汇总成绩排名");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        service.shutdown();
    }
}
```



**Semaphore** 

Semaphore 字面意思是信号量的意思，它的作用是控制访问特定资源的线程数目。例如：可用于流量控制，限制最大的并发访问数。

**构造方法**

```java
public Semaphore(int permits)
public Semaphore(int permits, boolean fair)
```

**解析：**

- permits 表示许可线程的数量
- fair 表示公平性，如果这个设为 true 的话，下次执行的线程会是等待最久的线程

**重要方法**

```java
public void acquire() throws InterruptedException
public void release()
```

**解析：**

- acquire() 表示阻塞并获取许可
- release() 表示释放许可

**代码实现：**

多个线程同时执行，但是限制同时执行的线程数量为 2 个。



```java
public class SemaphoreDemo {

    static class TaskThread extends Thread {

        Semaphore semaphore;

        public TaskThread(Semaphore semaphore) {
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println(getName() + " acquire");
                Thread.sleep(1000);
                semaphore.release();
                System.out.println(getName() + " release ");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        int threadNum = 5;
        Semaphore semaphore = new Semaphore(2);
        for (int i = 0; i < threadNum; i++) {
            new TaskThread(semaphore).start();
        }
    }

}
```

**打印结果：**



```undefined
Thread-1 acquire
Thread-0 acquire
Thread-0 release 
Thread-1 release 
Thread-2 acquire
Thread-3 acquire
Thread-2 release 
Thread-4 acquire
Thread-3 release 
Thread-4 release 
```

从打印结果可以看出，一次只有两个线程执行 acquire()，只有线程进行 release() 方法后才会有别的线程执行 acquire()。

需要注意的是 Semaphore 只是对资源并发访问的线程数进行监控，并不会保证线程安全。

**CyclicBarrier是java提供的同步辅助类。**
一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)，才得以继续执行。阻塞子线程，当阻塞数量到达定义的参与线程数后，才可继续向下执行。

```java
public class BarrierMain {

    public static void main(String[] args) {
        ThreadPoolExecutor executor=new ThreadPoolExecutor(5,5,1, TimeUnit.SECONDS
                ,new ArrayBlockingQueue<Runnable>(10)){
            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                super.afterExecute(r, t);
            }
        };
        CyclicBarrier cyclicBarrier=new CyclicBarrier(3, new Runnable() {
            @Override
            public void run() {
                System.out.println("=====当前阶段已完成");
            }
        });
        executor.submit(new BarrierDemo(cyclicBarrier));
        executor.submit(new BarrierDemo(cyclicBarrier));
        executor.submit(new BarrierDemo(cyclicBarrier));
        System.out.println("====主线程执行完毕");
        executor.shutdown();
    }
}

public class BarrierDemo implements Runnable {

    private final CyclicBarrier barrier;


    public BarrierDemo(CyclicBarrier barrier) {
        this.barrier = barrier;
    }

    @Override
    public void run() {
        try {
            System.out.println(Thread.currentThread().getName()+"到达现场");
            Thread.sleep(5000);
            //阻塞子线程
            barrier.await();
            //继续执行
            System.out.println(Thread.currentThread().getName()+"开始表演");
            Thread.sleep(5000);
            barrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}

输出结果：
====主线程执行完毕
pool-1-thread-3到达现场
pool-1-thread-2到达现场
pool-1-thread-1到达现场
=====当前阶段已完成
pool-1-thread-3开始表演
pool-1-thread-2开始表演
pool-1-thread-1开始表演
=====当前阶段已完成
```

CountDownLatch与CyclicBarrier:
CountDownLatch是一个同步的辅助类，允许一个或多个线程，等待其他一组线程完成操作，被等待线程（例如主线程）再继续执行。
CyclicBarrier是一个同步的辅助类，允许一组线程相互之间等待，达到一个共同点，子线程再继续执行。CyclicBarrier可以被重用，比如有三个线程，执行逻辑到达同步点阻塞，到齐后被唤醒，又再次执行逻辑，到达下一个同步点，到齐后再被唤醒
区别:

1. CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次
2. CyclicBarrier还提供getNumberWaiting(可以获得CyclicBarrier阻塞的线程数量)、isBroken(用来知道阻塞的线程是否被中断)等方法。
3. CountDownLatch会阻塞主线程，CyclicBarrier不会阻塞主线程，只会阻塞子线程。

**Exchanger 作用**

使两个线程之间进行数据传递。（对是两个之间而不是三个或者更多个线程之间）

**常用方法**

Exchanger()：无参构造方法。

V exchange(V v)：等待另一个线程到达此交换点（除非当前线程被中断），然后将给定的对象传送给该线程，并接收该线程的对象。

V exchange(V v, long timeout, TimeUnit unit)：等待另一个线程到达此交换点（除非当前线程被中断或超出了指定的等待时间），然后将给定的对象传送给该线程，并接收该线程的对象。

**例子**

```java
public class Test {
    static class Producer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;
        Producer(String name, Exchanger<Integer> exchanger) {
            super("Producer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            for (int i=1; i<5; i++) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = i;
                    System.out.println(getName()+" 交换前:" + data);
                    data = exchanger.exchange(data);
                    System.out.println(getName()+" 交换后:" + data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class Consumer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;
        Consumer(String name, Exchanger<Integer> exchanger) {
            super("Consumer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            while (true) {
                data = 0;
                System.out.println(getName()+" 交换前:" + data);
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = exchanger.exchange(data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName()+" 交换后:" + data);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Exchanger<Integer> exchanger = new Exchanger<Integer>();
        new Producer("", exchanger).start();
        new Consumer("", exchanger).start();
        TimeUnit.SECONDS.sleep(7);
        System.exit(-1);
    }
}
结果：
Consumer- 交换前:0
Producer- 交换前:1
Consumer- 交换后:1
Consumer- 交换前:0
Producer- 交换后:0
Producer- 交换前:2
Producer- 交换后:0
Consumer- 交换后:2
Consumer- 交换前:0
Producer- 交换前:3
Producer- 交换后:0
Consumer- 交换后:3
Consumer- 交换前:0
Producer- 交换前:4
Producer- 交换后:0
Consumer- 交换后:4
Consumer- 交换前:0
```

又如：

```java
public class Worker extends Thread {

    private final String name;
    Exchanger<String> exchanger;

    public Worker(String name, Exchanger<String> exchanger) {
        this.name = name;
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " : " +
                System.currentTimeMillis() +
                " " + name +
                " waiting message.");

        try {
            String exchange = exchanger.exchange("I'm " + name);
            System.out.println(name + " get message : " + exchange);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }


    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();

        for (int i = 1; i <= 4; i++) {
            new Worker("people-" + i, exchanger)
                    .start();
        }
    }
}

结果如下：
Thread-2 : 1636869362701 people-3 waiting message.
Thread-1 : 1636869362701 people-2 waiting message.
Thread-3 : 1636869362717 people-4 waiting message.
Thread-0 : 1636869362701 people-1 waiting message.
people-1 get message : I'm people-3
people-4 get message : I'm people-2
people-2 get message : I'm people-4
people-3 get message : I'm people-1
```

**什么是Phaser？** 原文地址：https://blog.csdn.net/liuyu973971883/article/details/107917079
Phaser又称“阶段器”，用来解决控制多个线程分阶段共同完成任务的情景问题。它与CountDownLatch和CyclicBarrier类似，都是等待一组线程完成工作后再执行下一步，协调线程的工作。但在CountDownLatch和CyclicBarrier中我们都不可以动态的配置parties，而Phaser可以动态注册需要协调的线程，相比CountDownLatch和CyclicBarrier就会变得更加灵活。

**Phaser的常用方法**
1、register方法：动态添加一个parties

int register()
2、bulkRegister方法：动态添加多个parties

parties：需要添加的个数
int bulkRegister(int parties)
3、getRegisteredParties方法：获取当前的parties数

int getRegisteredParties()

4、arriveAndAwaitAdvance方法：到达并等待其他线程到达

int arriveAndAwaitAdvance()
5、arriveAndDeregister方法：到达并注销该parties，这个方法不会使线程阻塞

int arriveAndDeregister()
6、arrive方法：到达，但不会使线程阻塞

int arrive()
7、awaitAdvance方法：
等待前行，可阻塞也可不阻塞，判断条件为传入的phase是否为当前phaser的phase。如果相等则阻塞，反之不进行阻塞

phase：阶段数值
int awaitAdvance(int phase)

8、awaitAdvanceInterruptibly方法：该方法与awaitAdvance类似，唯一不一样的就是它可以进行打断。

phase：阶段数值
timeout：超时时间
unit：时间单位
int awaitAdvanceInterruptibly(int phase)
int awaitAdvanceInterruptibly(int phase, long timeout, TimeUnit unit)
9、getArrivedParties方法：获取当前到达的parties数

int getArrivedParties()

10、getUnarrivedParties方法：获取当前未到达的parties数

int getUnarrivedParties()

11、getPhase方法：获取当前属于第几阶段，默认从0开始，最大为integer的最大值

int getPhase()

12、isTerminated方法：判断当前phaser是否关闭

boolean isTerminated()

13、forceTermination方法：强制关闭当前phaser

void forceTermination()

**案例**
1、使用Phaser动态注册parties

```java

public class PhaserExample {
    private static Random random = new Random(System.currentTimeMillis());
    public static void main(String[] args) {
        Phaser phaser = new Phaser();
        //创建5个任务
        for (int i=0;i<5;i++){
            new Task(phaser).start();
        }
        //动态注册
        phaser.register();
        //等待其他线程完成工作
        phaser.arriveAndAwaitAdvance();
        System.out.println("All of worker finished the task");
    }

    private static class Task extends Thread{
        private Phaser phaser;

        public Task(Phaser phaser) {
            this.phaser = phaser;
            //动态注册任务
            this.phaser.register();
        }

        @Override
        public void run() {
            try {
                System.out.println("The thread ["+getName()+"] is working");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("The thread ["+getName()+"] work finished");
            //等待其他线程完成工作
            phaser.arriveAndAwaitAdvance();
        }
    }
}
输出结果：
The thread [Thread-2] is working
The thread [Thread-4] is working
The thread [Thread-1] is working
The thread [Thread-3] is working
The thread [Thread-0] is working
The thread [Thread-1] work finished
The thread [Thread-3] work finished
The thread [Thread-0] work finished
The thread [Thread-4] work finished
The thread [Thread-2] work finished
All of worker finished the task
```

2、使用Phaser设置多个阶段

- 这边使用的案例是运动员，模拟多个运动员参加多个项目

```java
public class PhaserExample2 {
    private static Random random = new Random(System.currentTimeMillis());
    public static void main(String[] args) {
        //初始化5个parties
        Phaser phaser = new Phaser(5);
        for (int i=1;i<6;i++){
            new Athlete(phaser,i).start();
        }
    }
    //创建运动员类
    private static class Athlete extends Thread{
        private Phaser phaser;
        private int no;//运动员编号

        public Athlete(Phaser phaser,int no) {
            this.phaser = phaser;
            this.no = no;
        }

        @Override
        public void run() {
            try {
                System.out.println(no+": 当前处于第："+phaser.getPhase()+"阶段");
                System.out.println(no+": start running");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
                System.out.println(no+": end running");
                //等待其他运动员完成跑步
                phaser.arriveAndAwaitAdvance();

                System.out.println(no+": 当前处于第："+phaser.getPhase()+"阶段");
                System.out.println(no+": start bicycle");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
                System.out.println(no+": end bicycle");
                //等待其他运动员完成骑行
                phaser.arriveAndAwaitAdvance();

                System.out.println(no+": 当前处于第："+phaser.getPhase()+"阶段");
                System.out.println(no+": start long jump");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
                System.out.println(no+": end long jump");
                //等待其他运动员完成跳远
                phaser.arriveAndAwaitAdvance();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
结果：
1: 当前处于第：0阶段
5: 当前处于第：0阶段
2: 当前处于第：0阶段
4: 当前处于第：0阶段
3: 当前处于第：0阶段
2: start running
4: start running
1: start running
5: start running
3: start running
2: end running
3: end running
5: end running
4: end running
1: end running
2: 当前处于第：1阶段
3: 当前处于第：1阶段
3: start bicycle
1: 当前处于第：1阶段
2: start bicycle
1: start bicycle
4: 当前处于第：1阶段
5: 当前处于第：1阶段
5: start bicycle
4: start bicycle
5: end bicycle
4: end bicycle
2: end bicycle
1: end bicycle
3: end bicycle
3: 当前处于第：2阶段
5: 当前处于第：2阶段
1: 当前处于第：2阶段
4: 当前处于第：2阶段
1: start long jump
4: start long jump
3: start long jump
5: start long jump
2: 当前处于第：2阶段
2: start long jump
5: end long jump
2: end long jump
3: end long jump
4: end long jump
1: end long jump
```

3、综合演示

```java
public class PhaserExample3 {
    private static Random random = new Random(System.currentTimeMillis());
    public static void main(String[] args) throws InterruptedException {
        //初始化5个parties
        Phaser phaser = new Phaser(5);

        //只有当全部线程通过时才会进入下一阶段，从0开始
        System.out.println("当前阶段数："+phaser.getPhase());

        //添加一个parties
        phaser.register();
        System.out.println("当前Parties数："+phaser.getRegisteredParties());
        //添加多个parties
        phaser.bulkRegister(4);
        System.out.println("当前Parties数："+phaser.getRegisteredParties());

        new Thread(new Runnable() {
            @Override
            public void run() {
                //到达并等待其他线程到达
                phaser.arriveAndAwaitAdvance();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                //到达后注销该parties，不等待其他线程
                phaser.arriveAndDeregister();
                System.out.println("go on");
            }
        }).start();
        TimeUnit.MILLISECONDS.sleep(100);
        System.out.println("当前Parties数："+phaser.getRegisteredParties());
        System.out.println("当前到达数："+phaser.getArrivedParties());
        System.out.println("当前未达数："+phaser.getUnarrivedParties());

        //何时会停止，只有当parties中的数量为0时或者调用forceTermination方法就会停止了，我们也可以重写phaser中的onAdvance，给他返回true就会使这个phaser停止了
        System.out.println("phaser是否结束："+phaser.isTerminated());
        phaser.forceTermination();
        System.out.println("phaser是否结束："+phaser.isTerminated());
    }

}
结果：
当前阶段数：0
当前Parties数：6
当前Parties数：10
go on
当前Parties数：9
当前到达数：1
当前未达数：8
phaser是否结束：false
phaser是否结束：true
```

4、利用arrive只监听线程完成第一部分任务

```java
public class PhaserExample4 {
    private static Random random = new Random(System.currentTimeMillis());
    public static void main(String[] args) throws InterruptedException {
        //初始化6个parties
        Phaser phaser = new Phaser(6);
        //创建5个任务
        IntStream.rangeClosed(1,5).forEach(i->new ArrayTask(i,phaser).start());
        //等待5个任务的第一部分完成
        phaser.arriveAndAwaitAdvance();
        System.out.println("all work finished");
    }

    private static class ArrayTask extends Thread{
        private Phaser phaser;

        public ArrayTask(int name,Phaser phaser) {
            super(String.valueOf(name));
            this.phaser = phaser;
        }

        @Override
        public void run() {
            try {
                //模拟第一部分工作
                System.out.println(getName()+" start working");
                TimeUnit.SECONDS.sleep(random.nextInt(3));
                System.out.println(getName()+" end working");
                //该方法表示到达但不会使线程阻塞
                phaser.arrive();
                //模拟第二部分工作
                TimeUnit.SECONDS.sleep(random.nextInt(3));
                System.out.println(getName()+" do other thing");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
结果：
3 start working
1 start working
4 start working
2 start working
4 end working
1 end working
5 start working
2 end working
5 end working
4 do other thing
5 do other thing
2 do other thing
3 end working
1 do other thing
3 do other thing
all work finished
```

5、awaitAdvance演示

```java
public class PhaserExample5 {
    private static Random random = new Random(System.currentTimeMillis());
    public static void main(String[] args) {
        //初始化6个parties
        Phaser phaser = new Phaser(5);
        //创建5个任务
        IntStream.rangeClosed(1,5).forEach(i->new ArrayTask(i,phaser).start());
        System.out.println(phaser.getPhase());
        //当phaser中的当前阶段等于传入的阶段则该方法会阻塞，反之不会
        phaser.awaitAdvance(1);
        System.out.println("all work finished");
    }

    private static class ArrayTask extends Thread{
        private Phaser phaser;

        public ArrayTask(int name,Phaser phaser) {
            super(String.valueOf(name));
            this.phaser = phaser;
        }

        @Override
        public void run() {
            try {
                System.out.println(getName()+" start working");
               // phaser.getPhase();
                TimeUnit.SECONDS.sleep(random.nextInt(3));
                System.out.println(getName()+" end working");
                 phaser.arriveAndAwaitAdvance();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
结果：
0
all work finished
1 start working
3 start working
1 end working
2 start working
5 start working
4 start working
4 end working
3 end working
2 end working
5 end working
```

## 5、线程安全性文档

## 6、慎用延时加载

静态域延迟初始化：

```java
private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }

    private static FieldType getField() { return FieldHolder.field; }

```

双重检查，延迟初始化：

```java
 private volatile FieldType field4;
    private FieldType getField4() {
        FieldType result = field4;
        if (result != null)    // 没有加锁
            return result;

        synchronized(this) {
            if (field4 == null) // 加锁了
                field4 = computeFieldValue();
            return field4;
        }
    }
```

允许接收重复初始化的域：

```java
    private volatile FieldType field5;

    private FieldType getField5() {
        FieldType result = field5;
        if (result == null)
            field5 = result = computeFieldValue();
        return result;
    }
```



## 7、不要依赖线程调度器

任何依赖于线程调度器来达到正确性或者性能要求的程序，很有可能都是不可移植的。