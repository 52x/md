title: Singleton--Java实现方式
date: 2016/06/20
tags: [Java, 单例模式]

---

Java 中的单例模式是很简单的，从名字就可以很容易知道这种模式的特点：**对外只提供一个且唯一一个实例对象**。但是你知道这种设计模式有多少中实现方式吗？哪种实现方式最高效？当有多个线程访问这个单例时，如何保证进程间的同步？当要把单例对象序列化时，如何保证反序列化该对象时不会创建新的对象？

以上问题就是这次博客主要说明的内容。
<!-- more -->

###单例模式实现方式###

####version 1.0####
构造方法设置为 `private`， 仅被调用一次，将对象定义为公有 `final` 静态的。由于缺少公有的构造函数，从而保证创建的对象唯一。

```
//sigleton with public final field
public class Singleton{
	public static final Singleton INSTANCE = new Singleton();
	private Singleton(){ ... }
	
	public void leaveTheBuilding(){ ... }
}
```

客户端的任何行为都不会改变这一点，但是有一点要注意： 享有特权的客户端可以借助 `AccessibleObject.setAccessible` 方法，通过**反射机制**调用私有构造器。如果要抵御这种攻击，可以通过修改构造器，让它在被要求创建第二个实例时抛出异常。如下所示：

```
public class Singleton{
	public static final Singleton INSTANCE = new Singleton();
	private Singleton(){
		if(INSTANCE != null)
			throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
	}
	
	public void leaveTheBuilding(){ ... }
}
```

####version 1.1####
和上一种的区别在于将实例设置为私有的，同时提供一个公有的静态工厂方法：

```
public class Singleton{
	private static final Singleton INSTANCE = new Singleton();  //instantial pre
	private Singleton(){
		if(INSTANCE != null)
			throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
	}
	public static Singleton getIntance(){
		return INSTANCE;
	}
	public void leaveTheBuilding(){ ... }
}
```

与这种不同的另外一种实现方式是：延迟单例对象的初始化。

```
public class Singleton{
	private static Singleton INSTANCE = null;  //instantial wait till getInstance()
	private Singleton(){
		if(INSTANCE != null)
			throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
	}
	public static Singleton getIntance(){
		if(INSTANCE == null)
			INSTANCE = new Singleton();  // do the instantial here
		return INSTANCE;
	}
	public void leaveTheBuilding(){ ... }
}
```

在工厂方法中判断单例是否已经被初始化，如果没有就初始化对象。

但是上面这种常用的实现方式有个问题，因为实例是全局的，所以，在多线程的情况下这种方式存在问题，有可能会初始化多个对象。例如，有可能多个线程都同时通过了 `INSTANCE == null` 条件判断，于是就会生成多个对象，造成内存泄漏。于是为了实现线程之间的同步互斥访问临界区，有如下的改进版本：

####version 1.2####

```
public class Singleton{
	private static Singleton INSTANCE = null;  //instantial wait till getInstance()
	private Singleton(){
		if(INSTANCE != null)
			throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
	}
	public static Singleton getIntance(){
		if(INSTANCE == null)
			synchronized(Singleton.class){
				INSTANCE = new Singleton();  // do the instantial here
			}
			
		return INSTANCE;
	}
	public void leaveTheBuilding(){ ... }
}
```

>插一句： `synchronized(Singleton.class)` 和 `synchronized(Singleton.this)` 是有区别的。Singleton.class 只有一个，是属于类的，所有实例共享。第一种写法保证了只有一个线程可访问临界区，直到它释放锁。第二种方式是对它的每个实例只能有一个线程可同时进入临界区。

上面的做法看似好像可以了，但是实际还是有问题的。因为，对于多个线程，他们可以同时通过 `INSTANCE == null` 这个条件的判断，接着串行的执行同步块，同样可以创建多个对象。所以这种方式还是不行。

继续修改。
####version 1.3####

```
public class Singleton{
	private static Singleton INSTANCE = null;  //instantial wait till getInstance()
	private Singleton(){
		if(INSTANCE != null)
			throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
	}
	public static Singleton getIntance(){
		synchronized(Singleton.class){
			if(INSTANCE == null)
				INSTANCE = new Singleton();  // do the instantial here
		}
			
		return INSTANCE;
	}
	public void leaveTheBuilding(){ ... }
}```

这种在判断条件前就进行同步，理论上应该可以了。它保证只有一个对象可以创建实例了。但是这种方式会有个问题，就是多个线程执行时只能串行访问该代码，尽管创建代码只有一次，后面的都得同步。这种做法比较极端，同时性能上会大打折扣。

####version 1.4####
这里使用的是双重检查机制(`double checked locking`)。代码如下：

	public class Singleton{
		private static Singleton INSTANCE = null;  //instantial wait till getInstance()
		private Singleton(){
			if(INSTANCE != null)
				throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
		}
		public static Singleton getIntance(){
			//double check locking	
			if(INSTANCE == null){
				synchronized(Singleton.class){
					if(INSTANCE == null)
						INSTANCE = new Singleton();  // do the instantial here
				}
			}			
			return INSTANCE;
		}
		public void leaveTheBuilding(){ ... }
	}
	 

双重检验指的是：

1. 第一次的 `if` 判断该对象是否已经被初始化，如果已经被创建了就不需要同步了，直接返回即可；
2. 否则，进行同步创建对象； 
3. 第二个 `if` 确保只有一个线程进入，但是要再次判断其他的线程是否已经创建了对象，这个就不存在上面说的所有的对象依次串行创建新的对象。

这种有些复杂的方式在一定程度上基本上解决我们上面说的问题。但是还有需要注意的地方： `INSTANCE=new Singleton();` 这句不是原子操作，它在创建对象时会做如下的三件事：

1. 给 `INSTANCE` 分配内存；
2. 调用构造函数来初始化成员变量，形成实例；
3. 将 `INSTANCE` 对象指向分配的内存空间

但是在 `JVM` 的即时编译器(`Just In Time`)中存在指令重排的优化。(是不是要崩溃了，特么的，只是编译器问题，不是我本身程序问题)也就是说上面三步的执行顺序不确定，有可能是 1-2-3 也有可能是 1-3-2。如果是后者，在语句 3 执行后，语句 2 还未来得及执行，执行权被另外一个线程占领，那么之后就会一直报错。

解决办法就是把 `INSTANCE` 变量声明为 `volatile`。如下

	public class Singleton{
		private volatile static Singleton INSTANCE = null;  //instantial wait till getInstance()
		private Singleton(){
			if(INSTANCE != null)
				throw new IllegalStateException("Alreadly instantiated");   //protected from reflection mechanism 
		}
		public static Singleton getIntance(){
			//double check locking	
			if(INSTANCE == null){
				synchronized(Singleton.class){
					if(INSTANCE == null)
						INSTANCE = new Singleton();  // do the instantial here
				}
			}			
			return INSTANCE;
		}
		public void leaveTheBuilding(){ ... }
	}
 

使用 `volatile` 有两个作用：

1. 这个变量在多线程的情况下不会存在复本，直接从内存中读取。
2. 这个关键字会禁止指令重排序优化。也就是说，在 `volatile` 变量的赋值操作后会有一个内存屏障(汇编代码保护)，读操作不会被重排到内存屏障之前。


####version 2.0####
比上面 version 1.x 更简洁的版本如下：

	public class Singleton{
		private volatile static Singleton INSTANCE = new Singleton();
		private Singleton(){ ... }
		public static Singleton getInstance(){
			return INSTANCE;
		}
	}


这种方式存在一种缺陷就是实例是在类加载的时候就创建了，对于一些需要依赖外部信息来创建相应对象的情况这种方式就不适合了。

还有一种简洁的方式是使用内部类：


	public class Singleton{
		private class SingletonHolder{
			private volatile static Singleton INSTANCE = new Singleton();	
		}
		
		private Singleton(){ ... }
		public static Singleton getInstance(){
			return SingletonHolder.INSTANCE;
		}
	}


####version 3.0####

上面的所有版本针对与将对象序列化时都会存在问题。将该单例对象序列存储后，再进行反序列化时，每次都会生成一个新的对象，从而违背单实例。例如：


	public class Singleton implements Serializable{
		private static final long serialVersionUID = 1L;
		private static transient Singleton INSTANCE = null;
		
		//Rest of the things are same as above
		
		//no more fear of serialization
		@SuppressWarnings("unused")
		private Singleton readResolve(){
			return INSTANCE;
		}
	}


关键字 `transient` 和 `readResolve()` 方法保证了只会返回一个唯一的对象实例。

最优雅的方式是使用枚举类型来实现。

```
public enum Singleton{
	INSTANCE;
	public static Singleton getInstance(){
		return INSTANCE；
	}
}
```

枚举类型不用担心多线程，冗长的代码，对象序列化的问题。



###参考文献###
1. 《Effective Java (2nd)》
2. [what is an efficient way to implement a singleton pattern in java (stackoverflow)](http://stackoverflow.com/questions/70689/what-is-an-efficient-way-to-implement-a-singleton-pattern-in-java)