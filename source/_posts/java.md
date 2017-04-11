title: 面试准备
date: 2016/06/06
tags: [面试, Java, Mysql]

---
> 回答问题时，先回答是什么，再回答有什么用和要注意什么。这里给人的感觉做事有条理，理解东西比较透彻。

知识点总结：
1. 基础知识
2. io相关
3. 多线程
4. 集合
5. JVM原理
6. Mysql数据库原理及常用优化方法
7. Hadoop，MapReduce相关
8. 大型分布式系统设计与开发，web缓存，消息队列技术原理

<!-- more -->

###Java基础知识###
1. 一个** “.java”** 文件中可以包含多个类(不是内部类)，但是只能有一个public的类，并且public的类名必须与文件名相一致。
2. & 和 && 的区别： 他们都可以用作逻辑与的运算符，表示逻辑与，当两边都为ture，真个运算才为true；否则为false。不同点在于：&&具有短路的功能，若第一个表达式为false，第二个就不会计算，另一方面，&可以进行位运算。
3. 跳出多重循环：在内层循环中设置标志位，外层循环检测标志的状态。
4. **switch**语句中的表达式是一个整数表达式或者枚举常量，对于byte，short，char类型都可以隐含的转换为int，所以这些类型可以作为switch后面括号中的表达式类型，而对于long，string类型则不行。
5. **final**关键字修饰一个变量时，是指引用变量不能变。引用变量所指向的对象中的内容还是可以改变的。子类不能覆盖父类的final方法，final类不能有子类。
6. 不可以在非**static**方法内部调用非static方法。
7. ** Override(覆盖)，Overload(重载)**。子类覆盖父类的方法是，只能比父类抛出更少的异常，或者抛出父类抛出的异常的子类。子类的访问权限只能比父类的更大，不能更小。**父类的private在子类中不可见**。构造函数可以重载，但是不能被覆盖。
8. 写**clone()**方法时，通常都有一行代码，缺省为：**super.clone();**首先复制好父类的对象，再是子类的。
9. abstract类和接口的区别：abstract类不必含有抽象方法，但是含有抽象方法必须定义为abstract类。**abstract类不能实例对象。**接口中所有的方法必须是抽象的，方法默认定义为**public abstract**类型，成员变量默认为**public static final**类型。区别如下：
	1. 抽象类可以有构造方法，接口不能有构造方法。
	2. 抽象类中可以有普通成员变量，接口中没有普通成员变量。
	3. 抽象类中可以有非抽象的方法，接口中的所有方法必须都是抽象的
	4. 抽象类中的访问权限可以是public和protected，但是接口的访问必须是public
	5. 抽象类中可以包含静态方法，接口中不能包含静态方法
	6. 一个类可以实现多个接口，但是只能继承一个抽象类。

10. abstract 的method不可是static的，因为抽象的方法要被子类实现。**native方法表示该方法要用另外一种依赖平台的编程语言实现，不存在子类实现的问题，不能是abstract。**例如，FileOutputStream类要和硬件打交道，底层的实现是用操作系统相关的api实习。
11. 在try中的return语句和finally的关系：try 中的 return 将要返回的结果放在返回栈中，然后执行finally中的语句，最后返回栈中的结果。
12. final，finally，finalize的区别：final用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，类不可继承。**内部类要访问局部变量，局部变量要定义为final类型。**finally是异常处理的一部分，表示总是执行。finalize是Object的一个方法，在垃圾回收机制执行的时候会被调用回收对象的此方法，可以覆盖此方法提供垃圾收集
时的其他的资源回收。JVM不保证此方法总是被调用。
13. 异常与错误：他们都是继承自Throwable类。exception分为运行期异常和非运行期异常。
	1. Error一般指java虚拟接相关的问题，如系统崩溃，虚拟机出错，动态连接失败等，这种错误无法恢复或不可能捕获，将导致应用程序中断，通常应用程序无法处理这些错误，因此应用不应捕获Error对象，也无需使用throws任何Error对象。
	2. 运行期异常是RuntimeException类及其子类异常，如NullPointerException等。这些异常是不检查异常，程序中可以选择捕获处理也可以不处理。这些异常一般是由于程序逻辑错误引起。出现运行期异常后，要么线程中止，要么就是主线程终止。在异常退出前应该把异常数据处理掉，然后记录日志。**非运行期异常**是RuntimeException以外的异常，类型上都属于Exception类及其子类，如IOEexception，SQLException等以及用户自定义的异常。对于这类异常，**JAVA编译器强制要求我们对出现的这些异常进行catch并处理，否则程序就不能编译通过。**
	3. 几个常见的运行期异常：NullPointerException, ArrayIndexOutOfBoundsException, ClassCastException, IllegalArgumentException, NumberFormateException.
	
###多线程###

14. Java中实现线程的方式：
	1. java5以前：new Thread(){}.start();另外一种方式是new Thread(new Runnable(){}).start();
	2. java5以后，可以通过线程池创建多线程的方式：
```	
	ExecutorService pool = Executors.newFixedPool(3);
		for(int i=0;i<=ThreadSize;i++){
			pool.execute(new Runnable(){public void run(){}});
		}
		Executors.newCachedThreadPool().execute(new Runnable(){public void run(){}});
		Executors.newSingleThreadExecutor().execute(new Runable(){public void run(){}});
```
15. stop() 和 suspend() 不推荐使用。stop() 是因为它不安全。它会解除由线程获取的所有锁定，而且如果对象处于一种不连贯状态，那么其他线程能在那种状态下检查和修改它们。suspend()方法容易发生死锁。调用suspend()的时候，目标线程会停下来，但任然持有在这之前获得的锁定。此时，其他任何线程都不能访问锁定的资源，除非“挂起”的线程回复运行。

16. sleep() 和 wait(): sleep方法是线程类的方法，导致此线程暂停执行指定时间，给其他线程执行的机会，但依然持有自己的资源，不会释放对象锁。wait是Object类的方法，对此对象调用wait方法导致本线程放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象发出的notify方法(notifyAll)后本线程才进入对象锁争夺的状态。

17. 同步和异步：如果数据在线程间共享。例如正在写的数据以后可能被另外一个线程读，那么这些数据是共享的要进行同步，保证数据的一致性。当应用程序在对象上调用了一个需要花费很长时间来执行的方法，并且不希望让程序等待方法返回时，就应该使用异步编程。例如大量的计算和读写外存，执行读写的任务会异步的执行，当他们执行完后，发送中断请求，返回执行状态。

18. 线程状态：就绪，运行，阻塞，完成
19. synchronized和java.util.concurrent.locks.Lock异同：
	1. Lock能完成synchronized所实现的所有功能
	2. Lock有比synchronized更精确的线程语义和更好的性能。synchronized会自动释放锁，而Lock一定要求程序员手工释放，并且必须在finally从句中释放。Lock还有更强大的功能，例如，它的tryLock方法可以非阻塞方式去拿锁。

7. volatile是轻量级的synchronized，他在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量是，另外一个线程能读到这修改的值。

###集合###
20. Collection的认识。Java中封装了优良的接口和类组成了集合框架Collection，这些接口和类有很多抽象数据类型操作的API，而这是我们常用的且在数据结构中熟知的。例如Map，Set，List等。并且Java用面向对象设计的思想封装了这些数据结构和算法，这些极大的简化了程序员编程的负担。程序员也可以在这个集合框架上定义更高级别的数据抽象，从而满足自己的需要。
 ![collection](/images/Collection.bmp)

21. Collections 和 Arrays：注意看上图他们所处的位置。这两个类提供了封装器实现(Wrapper Implementations)、 数据结构算法和数组相关的应用。各种排序算法在Collections中有实现。

22. ArrayList和Vector的比较：两个都实现了List接口，他们都是有序集合，可以按照位置取出某个元素，并且元素允许重复。Vector是线程安全的，ArrayList是线程不安全。如果只有一个线程访问就用ArrayList，效率跟高。(类似于Hashtable是线程安全，HashMap线程不安全)；数据增长空间不一样，Vector增长一倍，ArrayList增长0.5倍。

23. HashMap和Hashtable：HashMap是Hashtable的轻量级实现(非线程安全的实现)，他们都实现了Map接口，主要区别在于HashMap允许key为空和value为空，效率高于Hashtable。

###IO流###
24. java中存在字节流和字符流。字节流继承InputStream和OutputStream，字符流继承自InputStramReader和OutputStreamWriter。

25. 字符流和字节流的主要区别：字符流使用了缓冲区，而字节流没有缓冲区；底层设备永远只接受字节数据；字符是字节通过不同的编码方式的包装；字符向字节转换时要注意编码方式。

26. java序列化：通过序列化可以将一个对象变成字节流传出去或者从字节流中恢复成一个对象，这样就可以通过网络传递对象，或者将对象写入磁盘。我们可以调用OutputStream的writeObject方法来写对象，对象必须首先实现Serializable接口。

27. java中直接缓冲区和非直接缓冲器有什么区别：

###JVM相关###
1. 64位JVM中，int 的长度任然为32位，与虚拟机本身无关。
2. Serial与Parallel GC之间的不同之处：他们在GC执行时都会引起stop-the-world。他们不同在于serial收集器是默认的复制收集器，执行GC的只有一个线程，而parallel使用多个GC线程执行。
3. WeakReference 和 SoftReference的 区别：他们有利于GC和内存的效率。但是WeakReference一旦失去最后一个强引用，就会被GC回收，而软引用虽然不能阻止被回收，但是可以延迟到JVM内存不足的时候。
4. JVM选项 -XX:+UseCompressedOops:当你将你的应用从 32 位的 JVM 迁移到 64 位的 JVM 时，由于对象的指针从 32 位增加到了 64 位，因此堆内存会突然增加，差不多要翻倍。这也会对 CPU 缓存（容量比内存小很多）的数据产生不利的影响。因为，迁移到 64 位的 JVM 主要动机在于可以指定最大堆大小，通过压缩 OOP 可以节省一定的内存。通过 -XX:+UseCompressedOops 选项，JVM 会使用 32 位的 OOP，而不是 64 位的 OOP。
5. JIT:just in time compliation，当代码执行的次数超过一定的阈值时，它会将Java字节码转换成本地代码，如，主要的热点代码会被转换为本地代码，这样有利于大幅度提高Java应用的性能。
6. 
###参考文献###

[Java面试精选](http://www.cnblogs.com/hnlshzx/p/3492197.html)
[Java集合框架](http://blog.csdn.net/u010736393/article/details/9125453)
[Java面试题列表](http://www.importnew.com/17232.html)
