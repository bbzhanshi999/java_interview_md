​	信号量Semaphore是一个并发工具类，用来控制可同时并发的线程数，其内部维护了一组虚拟许可，通过构造器指定许可的数量，每次线程执行操作时先通过acquire方法获得许可，执行完毕再通过release方法释放许可。如果无可用许可，那么acquire方法将一直阻塞，直到其它线程释放许可。

　　线程池用来控制实际工作的线程数量，通过线程复用的方式来减小内存开销。线程池可同时工作的线程数量是一定的，超过该数量的线程需进入线程队列等待，直到有可用的工作线程来执行任务。

　　使用Seamphore，你创建了多少线程，实际就会有多少线程进行执行，只是可同时执行的线程数量会受到限制。但使用线程池，你创建的线程只是作为任务提交给线程池执行，实际工作的线程由线程池创建，并且实际工作的线程数量由线程池自己管理。

简单来说，线程池实际工作的线程是work线程，不是你自己创建的，是由线程池创建的，并由线程池自动控制实际并发的work线程数量。而Seamphore相当于一个信号灯，作用是对线程做限流，Seamphore可以对你自己创建的的线程做限流（也可以对线程池的work线程做限流），Seamphore的限流必须通过手动acquire和release来实现。

区别就是两点：

1、实际工作的线程是谁创建的？

使用线程池，实际工作线程由线程池创建；使用Seamphore，实际工作的线程由你自己创建。

2、限流是否自动实现？

线程池自动，Seamphore手动。

案例

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreTest {
 public static void main(String[] args) {
     ExecutorService exec = Executors.newFixedThreadPool(20);
     Semaphore semp = new Semaphore(5);
     for (int i = 0;i<20;i++){
         exec.submit(new MyTask(semp));
     }
     exec.shutdown();
 }

 private static class MyTask implements Runnable{
     private Semaphore semaphore;
     public MyTask(Semaphore semaphore){
         this.semaphore = semaphore;
     }
     @Override
     public void run() {
         try {
             semaphore.acquire();
             Thread.sleep(2000);     //模拟2S的线程操作
             System.out.println(System.currentTimeMillis() +  ":done!");
             semaphore.release();
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
}
```

