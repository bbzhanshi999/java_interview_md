Java1.5之前synchronized是一个重量级锁，Java 1.6对synchronized进行的各种优化后，synchronized并不会显得那么重了。我们先介绍重量级锁，然后介绍优化后的轻量级锁和偏向锁。

# 0.对象头以及Mark Word

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yMDYyNzI5LTlhNzhmN2VhNzY3MWEwMzEucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXAlN0NpbWFnZVZpZXcyLzIvdy84ODEvZm9ybWF0L3dlYnA)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMDUzNjI5LWY4Zjg0ODBmMDI2OTY4MWQucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXAlN0NpbWFnZVZpZXcyLzIvdy85OTkvZm9ybWF0L3dlYnA)

# 1重量级锁

重量级的synchronized有三个用法：

- 普通同步方法，锁的是当前实例对象。
- 静态同步方法，锁的是当前类的class对象。
- 同步代码块，锁的是synchronized括号里配置的对象。

上述这三种机制的切换是根据竞争激烈程度进行的：

- 在几乎无竞争的条件下， 会使用偏向锁
- 在轻度竞争的条件下， 会由偏向锁升级为轻量级锁
- 在重度竞争的情况下， 会由轻量级锁升级为重量级锁。

## 1.1普通同步方法用法及原理

用法：

```java
public class SynchronizedMethod {



    public synchronized void method() {



        System.out.println("Hello World!");



    }



}
```

方法调用如下图所示：

![img](https://img-blog.csdnimg.cn/20190805205154958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0NTk4MDU5,size_16,color_FFFFFF,t_70)

1.当前线程调用指令将会调用当前实例对象，去检查当前实例对象（**在堆中**）所对应的类方法（**在方法区中**）方法表中的**访问标志（ACC_SYNCHRONIZED）**是否被设置；

2.如果设置了，那么检查此同步对象有没有被锁定，即检查它的**Mark Word**的锁标志位是不是01，则虚拟机首先在当前线程的栈中创建我们称之为“**锁记录（Lock Record）**”的空间，用于存储锁对象的Mark Word的拷贝，官方把这个拷贝称为**Displaced Mark Word**。

3.执行线程将通过Displaced Mark Word中指向重量级锁的指针，找到**Monitor对象的起始地址**，获取到monitor对象，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 

## 1.2同步代码块用法及原理

用法：

```java
public class SynchronizedDemo {



    public void method() {



        synchronized (this) {



            System.out.println("Method 1 start");



        }



    }



}
```

原理同2.1.1。

## 1.3静态同步方法用法及原理

用法：

```java
public static class SynchronizedMethod {



    public synchronized void method() {



        System.out.println("Hello World!");



    }



}
```

在类中如果某方法或某代码块中有 synchronized，那么在生成一个该类实例后，该实例也就有一个单独的Monitor对象，而static synchronized则是所有该类的所有实例公用得一个Monitor对象。

# 2偏向锁

大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，偏向锁可以降低多次加锁解锁的开销。

偏向锁会偏向于第一个获得它的线程，如果在接下来的执行过程中，该锁没有被其他的线程获取，则持有偏向锁的线程将永远不需要同步。大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。

当有另外一个线程去尝试获取这个锁时，偏向模式就宣告结束。根据锁对象目前是否处于被锁定的状态，撤销偏向后恢复到未锁定或轻量级锁定状态。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMDUzNjI5LTJhY2RjNmE4Y2I2YTNiNzcucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXAlN0NpbWFnZVZpZXcyLzIvdy84NjUvZm9ybWF0L3dlYnA)

# 3轻量级锁

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMDUzNjI5LTYxYjlhOTIzNDRmNzY4ZDYucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXAlN0NpbWFnZVZpZXcyLzIvdy84NjUvZm9ybWF0L3dlYnA)

# 4总结

| **锁**       | **优点**                                                     | **缺点**                                         | **适用场景**                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------ |
| **偏向锁**   | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 | 适用于只有一个线程访问同步块场景。   |
| **轻量级锁** | 竞争的线程不会阻塞，提高了程序的响应速度。                   | 如果始终得不到锁竞争的线程使用自旋会消耗CPU。    | 追求响应时间。同步块执行速度非常快。 |
| **重量级锁** | 线程竞争不使用自旋，不会消耗CPU。                            | 线程阻塞，响应时间缓慢。                         | 追求吞吐量。同步块执行速度较长。     |

synchronized的执行过程： 
1. 检测Mark Word里面是不是当前线程的ID，如果是，表示当前线程处于偏向锁 
2. 如果不是，则使用CAS将当前线程的ID替换Mard Word，如果成功则表示当前线程获得偏向锁，置偏向标志位1 
3. 如果失败，则说明发生竞争，撤销偏向锁，进而升级为轻量级锁。 
4. 当前线程使用CAS将对象头的Mark Word替换为锁记录指针，如果成功，当前线程获得锁 
5. 如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。 
6. 如果自旋成功则依然处于轻量级状态。 
7. 如果自旋失败，则升级为重量级锁。

上面几种锁都是JVM自己内部实现，当我们执行synchronized同步块的时候jvm会根据启用的锁和当前线程的争用情况，决定如何执行同步操作；

在所有的锁都启用的情况下线程进入临界区时会先去获取偏向锁，如果已经存在偏向锁了，则会尝试获取轻量级锁，启用自旋锁，如果自旋也没有获取到锁，则使用重量级锁，没有获取到锁的线程阻塞挂起，直到持有锁的线程执行完同步块唤醒他们；

偏向锁是在无锁争用的情况下使用的，也就是同步开在当前线程没有执行完之前，没有其它线程会执行该同步块，一旦有了第二个线程的争用，偏向锁就会升级为轻量级锁，如果轻量级锁自旋到达阈值后，没有获取到锁，就会升级为重量级锁；

如果线程争用激烈，那么应该禁用偏向锁。

 

参考：

<https://www.jianshu.com/p/e62fa839aa41>

<https://www.jianshu.com/p/73b9a8466b9c>

<https://www.cnblogs.com/deltadeblog/p/9559035.html>

<https://www.cnblogs.com/linghu-java/p/8944784.html>