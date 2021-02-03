JAVA四种引用主要是强引用，软引用，弱引用，虚引用

​    （1）强引用是指对象和字符串，只要某个对象与强引用有关，那么，JVM必定不会回收这个对象，即使在内存不足的情况下，JVM宁愿抛出OutOfMemory,也不会回收这种对象。如果想中断强引用和某个对象之间的关系，那么可以显示的将引用赋值为NULL,这样JVM就可以将该对象进行回收了。

​      （2）软引用是指用来描述一些有用但是不是必需的对象，在JAVA中用java.lang.ref.SoftReference类来表示，对于软引用关联着的对象，只有在内存不足的时候JVM才会回收该对象，因此，这一点很好的解决了OOM的问题，并且这个特性很适合用来实现缓存；软引用可以和一个引用队列联合使用，如果软引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中，

```java
import java.lang.ref.SoftReference;



  



public class Main {



    public static void main(String[] args) {



      



        SoftReference<String> sr = new SoftReference<String>(new String("hello"));



          



        System.out.println(sr.get());



        System.gc();               //通知JVM的gc进行垃圾回收



        System.out.println(sr.get());



    }



}



//输出还存在，只有在内存不足的时候JVM才会回收该对象。
```

​     (3)弱引用是指用来描述非必需对象，当JVM进行垃圾回收的时候，无论内存是否充足，都会回收被弱引用关联的对象，在java中，用java.lang.ref.WeakReference类来表示。

​       （4）虚引用并不影响对象的声明周期，在Java中用java.lang.ref.PhantomReference类表示，如果一个对象虚引用关联，则根没有引用与之关联一样，在任何时候都可能被垃圾回收。要注意的是，虚引用必须和引用对列关联使用，当垃圾回收器准备一个对象时，如果发现他还有虚引用，就会把这个虚引用加入到与之关联的引用队列中，成相互可以通过判断引用队列中是否已经加入到引用队列中，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

被软引用关联的对象只有在内存不足时才会被回收，而被弱引用关联的对象在JVM进行垃圾回收时总会被回收。针对上面的特性，软引用适合用来进行缓存，当内存不够时能让JVM回收内存，弱引用能用来在回调函数中防止内存泄露。





https://www.cnblogs.com/mfrank/p/9837070.html

> 虚引用是使用PhantomReference创建的引用，虚引用也称为幽灵引用或者幻影引用，是所有引用类型中最弱的一个。一个对象是否有虚引用的存在，完全不会对其生命周期构成影响，也无法通过虚引用获得一个对象实例。

**0**|**1****说明**

虚引用，正如其名，对一个对象而言，这个引用形同虚设，有和没有一样。

> 如果一个对象与GC Roots之间仅存在虚引用，则称这个对象为`虚可达（phantom reachable）`对象。

当试图通过虚引用的get()方法取得强引用时，总是会返回null，并且，虚引用必须和引用队列一起使用。既然这么虚，那么它出现的意义何在？？

别慌别慌，自然有它的用处。它的作用在于跟踪垃圾回收过程，在对象被收集器回收时收到一个系统通知。 当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，将这个虚引用加入引用队列，在其关联的虚引用出队前，不会彻底销毁该对象。 所以可以通过检查引用队列中是否有相应的虚引用来判断对象是否已经被回收了。

如果一个对象没有强引用和软引用，对于垃圾回收器而言便是可以被清除的，在清除之前，会调用其finalize方法，如果一个对象已经被调用过finalize方法但是还没有被释放，它就变成了一个虚可达对象。

与软引用和弱引用不同，显式使用虚引用可以阻止对象被清除，只有在程序中显式或者隐式移除这个虚引用时，这个已经执行过finalize方法的对象才会被清除。想要显式的移除虚引用的话，只需要将其从引用队列中取出然后扔掉（置为null）即可。

同样来看一个栗子：

```
public class PhantomReferenceTest {
    private static final List<Object> TEST_DATA = new LinkedList<>();
    private static final ReferenceQueue<TestClass> QUEUE = new ReferenceQueue<>();

    public static void main(String[] args) {
        TestClass obj = new TestClass("Test");
        PhantomReference<TestClass> phantomReference = new PhantomReference<>(obj, QUEUE);

        // 该线程不断读取这个虚引用，并不断往列表里插入数据，以促使系统早点进行GC
        new Thread(() -> {
            while (true) {
                TEST_DATA.add(new byte[1024 * 100]);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    Thread.currentThread().interrupt();
                }
                System.out.println(phantomReference.get());
            }
        }).start();

        // 这个线程不断读取引用队列，当弱引用指向的对象呗回收时，该引用就会被加入到引用队列中
        new Thread(() -> {
            while (true) {
                Reference<? extends TestClass> poll = QUEUE.poll();
                if (poll != null) {
                    System.out.println("--- 虚引用对象被jvm回收了 ---- " + poll);
                    System.out.println("--- 回收对象 ---- " + poll.get());
                }
            }
        }).start();

        obj = null;

        try {
            Thread.currentThread().join();
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.exit(1);
        }
    }

    static class TestClass {
        private String name;

        public TestClass(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "TestClass - " + name;
        }
    }
}
```

使用的虚拟机设置如下：

```
-verbose:gc -Xms4m -Xmx4m -Xmn2m
```

运行结果如下：

```
[GC (Allocation Failure)  1024K->432K(3584K), 0.0113386 secs]
[GC (Allocation Failure)  1455K->520K(3584K), 0.0133610 secs]
[GC (Allocation Failure)  1544K->648K(3584K), 0.0008654 secs]
null
null
null
[GC (Allocation Failure)  1655K->973K(3584K), 0.0008111 secs]
null
...省略几个null的输出
[GC (Allocation Failure)  1980K->1997K(3584K), 0.0009289 secs]
[Full GC (Ergonomics)  1997K->1870K(3584K), 0.0048483 secs]
--- 弱引用对象被jvm回收了 ---- java.lang.ref.PhantomReference@74cbe23d
--- 回收对象 ---- null
null
...省略几个null和几次Full GC的输出
[Full GC (Ergonomics)  2971K->2971K(3584K), 0.0024850 secs]
[Full GC (Allocation Failure)  2971K->2971K(3584K), 0.0022460 secs]
Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
	at weakhashmap.PhantomReferenceTest.lambda$main$0(PhantomReferenceTest.java:20)
	at weakhashmap.PhantomReferenceTest$$Lambda$1/2065951873.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
```

因为设置的虚拟机堆大小比较小，所以创建一个100k的对象时直接进入了老年代，等到发生Full GC时才会被扫描然后回收。

**0**|**1****适用场景**

使用虚引用的目的就是为了得知对象被GC的时机，所以可以利用虚引用来进行销毁前的一些操作，比如说资源释放等。这个虚引用对于对象而言完全是无感知的，有没有完全一样，但是对于虚引用的使用者而言，就像是待观察的对象的把脉线，可以通过它来观察对象是否已经被回收，从而进行相应的处理。

事实上，虚引用有一个很重要的用途就是用来做堆外内存的释放，DirectByteBuffer就是通过虚引用来实现堆外内存的释放的。

**0**|**1****小结**

- 虚引用是最弱的引用
- 虚引用对对象而言是无感知的，对象有虚引用跟没有是完全一样的
- 虚引用不会影响对象的生命周期
- 虚引用可以用来做为对象是否存活的监控