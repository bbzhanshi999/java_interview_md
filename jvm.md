# JVM虚拟机相关

![](F:\teachingNote\java面试\img\jdk体系结构.jpg)

![](F:\teachingNote\java面试\img\helloworld例子.png)

![](F:\teachingNote\java面试\img\jvm内存模型.png)

```java
public class Math{
    
    public static int initData = 666;
    public static User user = new User();
    
    public int compute(){
        int a = 1;
        int b = 2;
        int c = (a+b)*20;
        return c;
    }
    
    public static void main(String[] args){
        Math math = new Math();
        math.compute();
    }
    
    
}
```

![](F:\teachingNote\java面试\img\栈内存结构.png)

## 程序计数器

1.每一个线程独享一个程序计数器

2.程序计数器记录了当前线程运行的每一个指令的行号(内存地址指针)





## 对象头

https://www.jianshu.com/p/3d38cba67f8b

![](F:\teachingNote\java面试\img\对象组成.png)

![](F:\teachingNote\java面试\img\内存模型总.png)

## 动态链接的作用

将标记引用转换成直接引用

## 可达性分析以及GC Roots的对象

![](F:\teachingNote\java面试\img\gcroot.png)

可达性分析算法的基本思路就是通过一系列称为“GC Roots”的对象作为起点，从这些结点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（就是从GC Roots到这个对象不可达）时，则证明此对象是不可用的，为可回收对象。 图中，右边部分都是不可达的对象，都是可回收对象。

在Java语言中，可以作为GC Roots的对象包括以下几种：

1. 虚拟机栈（栈帧中的本地变量表）中引用的对象
2.方法区中类静态属性引用的对象
3.方法区中常量变量引用的对象
4.本地方法栈中JNI（即一般说的Native方法）引用的对象
5.活跃线程的引用对象

## 老年代常见数据

1.spring的bean2

2.连接池，线程池

3.静态变量

4.初始化缓存数据

等等



## 多核并发缓存架构

![](F:\teachingNote\java面试\img\多核并发缓存架构.png)







## JMM线程内存模型

![](F:\teachingNote\java面试\img\jmm模型.png)

### java数据原子操作

![](F:\teachingNote\java面试\img\java内存原子性操作.png)

![](F:\teachingNote\java面试\img\jmm内存模型.png)

## volatile实现方式

![](F:\teachingNote\java面试\img\volatile实现方式.png)

![](F:\teachingNote\java面试\img\volatile底层原理.png)

![](F:\teachingNote\java面试\img\汇编指令.png)

![](F:\teachingNote\java面试\img\volitale可见性.png)

## 线程的理解

![](F:\teachingNote\java面试\img\线程理解.png)

![](F:\teachingNote\java面试\img\内核线程.png)

![](F:\teachingNote\java面试\img\两种线程.png)

![1588818251854](C:\Users\bbzha\AppData\Roaming\Typora\typora-user-images\1588818251854.png)

![1588819218601](C:\Users\bbzha\AppData\Roaming\Typora\typora-user-images\1588819218601.png)

