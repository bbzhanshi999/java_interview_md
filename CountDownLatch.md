#### 1.背景：

- countDownLatch是在java1.5被引入，跟它一起被引入的工具类还有CyclicBarrier、Semaphore、concurrentHashMap和BlockingQueue。
- 存在于java.util.cucurrent包下。

#### 2.概念

- countDownLatch这个类使一个线程等待其他线程各自执行完毕后再执行。
- 是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。

#### 3.源码

- countDownLatch类中只提供了一个构造器：

```cpp
//参数count为计数值
public CountDownLatch(int count) {  };  
```

- 类中有三个方法是最重要的：

```java
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { };   
//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  
//将count值减1
public void countDown() { };  
```

#### 4.示例

*普通示例：*

```csharp
public class CountDownLatchTest {

    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(2);
        System.out.println("主线程开始执行…… ……");
        //第一个子线程执行
        ExecutorService es1 = Executors.newSingleThreadExecutor();
        es1.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                    System.out.println("子线程："+Thread.currentThread().getName()+"执行");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                latch.countDown();
            }
        });
        es1.shutdown();

        //第二个子线程执行
        ExecutorService es2 = Executors.newSingleThreadExecutor();
        es2.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("子线程："+Thread.currentThread().getName()+"执行");
                latch.countDown();
            }
        });
        es2.shutdown();
        System.out.println("等待两个线程执行完毕…… ……");
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("两个子线程都执行完毕，继续执行主线程");
    }
}
```

结果集：

```undefined
主线程开始执行…… ……
等待两个线程执行完毕…… ……
子线程：pool-1-thread-1执行
子线程：pool-2-thread-1执行
两个子线程都执行完毕，继续执行主线程
```

*模拟并发示例：*

```csharp
public class Parallellimit {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newCachedThreadPool();
        CountDownLatch cdl = new CountDownLatch(100);
        for (int i = 0; i < 100; i++) {
            CountRunnable runnable = new CountRunnable(cdl);
            pool.execute(runnable);
        }
    }
}

 class CountRunnable implements Runnable {
    private CountDownLatch countDownLatch;
    public CountRunnable(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }
    @Override
    public void run() {
        try {
            synchronized (countDownLatch) {
                /*** 每次减少一个容量*/
                countDownLatch.countDown();
                System.out.println("thread counts = " + (countDownLatch.getCount()));
            }
            countDownLatch.await();
            System.out.println("concurrency counts = " + (100 - countDownLatch.getCount()));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

> ***CountDownLatch和CyclicBarrier区别：**
>  1.countDownLatch是一个计数器，线程完成一个记录一个，计数器递减，只能只用一次
>  2.CyclicBarrier的计数器更像一个阀门，需要所有线程都到达，然后继续执行，计数器递增，提供reset功能，可以多次使用

作者：指尖架构141319

链接：https://www.jianshu.com/p/e233bb37d2e6

来源：简书

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 1. CountDownLatch（减一）

  CountDownLatch 是基于 **AQS** 机制实现的，在AQS底层维护了一个 volatile 修饰的状态信息 state，初始化为子线程的个数 n，首先主线程调用 **await 方法**进入阻塞状态，等待 n 个子线程执行完才能继续执行， 这 n 个子线程并行执行，每个线程执行完后都会调用一次 **countDown 方法**，该方法会通过 CAS 机制使 state 减一。等到所有子线程执行完后，state 就等于0了，然后主线程继续执行。
  

##### CountDownLatch在实现过程中有什么需要注意的?

- 各个线程的处理没有先后顺序，并发情况下无法保证顺序执行。
- 各子线程的countDown计数要确保一定执行，否则会导致 state 减不到0，主线程一直阻塞。
- 主线程的await要设置超时等待，出现异常及时抛出。

### 2. CyclicBarrier（加一）

  CyclicBarrier字面意思是可循环使用（Cyclic）的屏障（Barrier），它的原理是：线程调用 await 方法来进入阻塞状态，并且计数器加一，表示自己到达屏障了。当计数达到参与的线程总数时，也就意味着所有线程都到达屏障了，此时释放所有等待的线程，并且计数器置为0重新开始。
  

### 3. CountDownLatch与CyclicBarrier的区别

- CountDownLatch 调用 countDown 方法计数器减一，是减法计数，计数为0时释放所有等待线程，计数器只能使用一次，不能重复使用。（调用 await 方法只进行阻塞，不影响计数）
- CyclicBarrier 调用 await 方法进入阻塞状态，并且计数器加一，是加法计数，当计数达到线程总数时就释放所有等待线程，计数器置为0重新开始，计数器可重复使用。

（简单理解方式：对于CountDownLatch，重点是“一个或多个线程等待”，而其他的线程在完成“某件事情”之后，可以终止，也可以等待。而对于CyclicBarrier，重点是多个线程，在任意一个线程没有完成时，所有线程都必须等待）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191130205313584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RsNjc0NzU2MzIx,size_16,color_FFFFFF,t_70)
**应用场景：**

- CountDownLatch：启动一个服务时，主线程需要等待多个组件加载完毕，才能继续执行。英雄联盟，所有人都准备好了才能开始游戏。
- CyclicBarrier：可以用于多线程计算数据、最后合并计算结果的场景，比如计算一个人多个账户的平均银行流水，n个线程分别计算这个人的n个账户，只有它们都计算完成了，才能执行合并计算，得到用户的平均银行流水。

### 4. Semaphore（信号量）

   Synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。
  

### 5. 使用案例

CountDownLatch 的使用案例：

```java
/**
 * CountDownLatch 的使用案例
 */
public class test {
    public static void main(String[] args) throws InterruptedException {
        final int threadSize = 100;
        AtomicInteger count = new AtomicInteger();
        CountDownLatch cdl = new CountDownLatch(threadSize);
        ExecutorService e = Executors.newCachedThreadPool();
        for (int i = 0; i < threadSize; i++) {
            e.execute(() -> {
                count.getAndIncrement();
                cdl.countDown();
            });
        }
        cdl.await();
        e.shutdown();
        System.out.println(count.get()); //总是最后输出100
    }
}
```