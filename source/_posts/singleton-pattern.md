title: 单例模式
date: 2015-06-17 10:22:41
categories: Design Pattern
tags: [singleton]
---

二次世界大战的时候，我国有一个著名的战役叫“长沙保卫战”，中国军队指挥官薛岳将军率领第 9 战区十余万将士，通过所谓的“焦土”战术 4 次瓦解日军的大规模进攻，给对当时的国民党政府打了一针强心剂。这四次战役中最让人我难忘的一幕是，面对单兵战斗力是中国军队 5 倍的日军，人数上虽然占据一定优势，但是只有第 10 军和第 74 军两只军队装备了现代化的军械，其余军队都是“汉阳造”的落后装备。薛将军命令第 10 军反复在湘北、赣北多处出阵地来回穿插，面对东西方向出现的多路敌军，帮助装备落后的部队一起防守阵地，让敌人误以为是多支部队，其实薛岳将军只是调动了同一支部队，正是这一单一实例的对象 (第 10 军) 在各个战场均发挥出了显著的作用，为第二次长沙战役的全面获胜起了至关重要的作用。

回到我们的主题。考虑这样一个应用，读取配置文件的内容。很多应用项目，都有与应用相关的配置文件，这些配置文件很多是由项目开发人员自定义的，在里面定义一些应用重要的参数数据。当然，在实际的项目中，这种配置文件多数采用 xml 格式，也有采用 properties 格式的，我们这里假设创建了一个名为 AppConfig 的类，它专门用来读取配置文件内的信息。客户端通过 new 一个 AppConfig 的实例来得到一个操作配置文件内容的对象。如果在系统运行中，有很多地方都需要使用配置文件的内容，也就是说很多地方都需要创建 AppConfig 对象的实例。换句话说，在系统运行期间，系统中会存在很多个 AppConfig 的实例对象，这里读者有没有发现有什么问题存在？当然有问题了，试想一下，每一个 AppConfig 实例对象里面都封装着配置文件的内容，系统中有多个 AppConfig 实例对象，也就是说系统中会同时存在多份配置文件的内容，这样会严重浪费内存资源。如果配置文件内容越多，对于系统资源的浪费程度就越大。事实上，对于 AppConfig 这样的类，在运行期间只需要一个实例对象就足够了。

从专业化来说，单例模式是一种对象创建模式，它用于产生一个对象的具体实例，它可以确保系统中一个类只产生一个实例。Java 里面实现的单例是一个虚拟机的范围，因为装载类的功能是虚拟机的，所以一个虚拟机在通过自己的 ClassLoad 装载实现单例类的时候就会创建一个类的实例。在 Java 语言中，这样的行为能带来两大好处：

1. 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
2. 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

## 最简单的实现
首先单例类必须要有一个 private 访问级别的构造函数，只有这样，才能确保单例不会在系统中的其他代码内被实例化，；其次，instance 成员变量和 getInstance 方法必须是 static 的。

	public class SingletonClass { 
	
	  private static final SingletonClass instance = new SingletonClass(); 
	    
	  public static SingletonClass getInstance() { 
	    return instance; 
	  } 
	    
	  private SingletonClass() { 
	     
	  } 
    
	}

## 性能优化——lazy loaded
上述代码唯一的不足是无法对 instance 实例做延时加载，例如单例的创建过程很慢，而由于 instance 成员变量是 static 定义的，因此在 JVM 加载单例类时，单例对象就会被建立，如果此时这个单例类在系统中还扮演其他角色，那么在任何使用这个单例类的地方都会初始化这个单例变量，而不管是否会被用到。

	public class SingletonClass { 
	
	  private static SingletonClass instance = null; 
	    
	  public static SingletonClass getInstance() { 
	    if(instance == null) { 
	      instance = new SingletonClass(); 
	    } 
	    return instance; 
	  } 
	    
	  private SingletonClass() { 
	     
	  } 
	    
	}

首先对于静态成员变量 instance 初始化赋值 null，确保系统启动时没有额外的负载；其次，在 getInstance() 工厂方法中，判断当前单例是否已经存在，若存在则返回，不存在则再建立单例。这里尤其要注意的是，getInstance() 方法必须是同步的，否则在多线程环境下，当线程 1 正新建单例时，完成赋值操作前，线程 2 可能判断 instance 为 null，故线程 2 也将启动新建单例的程序，而导致多个实例被创建，故同步关键字是必须的。由于引入了同步关键字，导致多线程环境下耗时明显增加。
## 同步
线程A希望使用SingletonClass，调用getInstance()方法。因为是第一次调用，A就发现instance是null的，于是它开始创建实例，就在这个时候，CPU发生时间片切换，线程B开始执行，它要使用SingletonClass，调用getInstance()方法，同样检测到instance是null——注意，这是在A检测完之后切换的，也就是说A并没有来得及创建对象——因此B开始创建。B创建完成后，切换到A继续执行，因为它已经检测完了，所以A不会再检测一遍，它会直接创建对象。这样，线程A和B各自拥有一个SingletonClass的对象——单例失败！

解决办法，getInstance()加上同步锁，一个线程必须等待另外一个线程创建完成后才能使用这个方法，这就保证了单例的唯一性：

	public class SingletonClass { 
	
	  private static SingletonClass instance = null; 
	    
	  public synchronized static SingletonClass getInstance() { 
	    if(instance == null) { 
	      instance = new SingletonClass(); 
	    } 
	    return instance; 
	  } 
	    
	  private SingletonClass() { 
	     
	  } 
	    
	}

## 同步性能问题
synchronized修饰的同步块要比一般的代码段慢上几倍的。

采用double-checked locking设计实现单例模式

	public class SingletonClass { 
	
	  private static SingletonClass instance = null; 
	
	  public static SingletonClass getInstance() { 
	    if (instance == null) { 
	      synchronized (SingletonClass.class) { 
	        if (instance == null) { 
	          instance = new SingletonClass(); 
	        } 
	      } 
	    } 
	    return instance; 
	  } 
	
	  private SingletonClass() { 
	
	  } 
	
	}

### 为何要使用双重检查锁定？
考虑这样一种情况，就是有两个线程同时到达，即同时调用 GetInstance（），此时由于 singleton == null ，所以很明显，两个线程都可以通过第一重的 singleton == null ，进入第一重 if 语句后，由于存在锁机制，所以会有一个线程进入synchronized语句并进入第二重 singleton == null，而另外的一个线程则会在synchronized语句的外面等待。而当第一个线程执行完 new  Singleto（）语句后，便会退出锁定区域，此时，第二个线程便可以进入synchronized语句块，此时，如果没有第二重 singleton == null 的话，那么第二个线程还是可以调用 new  Singleton（）语句，这样第二个线程也会创建一个Singleton实例，这样也还是违背了单例模式的初衷的，所以这里必须要使用双重检查锁定。

细心的朋友一定会发现，如果我去掉第一重 singleton == null，程序还是可以在多线程下完好的运行的，考虑在没有第一重 singleton == null的情况下，当有两个线程同时到达，此时，由于synchronized机制的存在，第一个线程会进入synchronized语句块，并且可以顺利执行 new Singleton（），当第一个线程退出synchronized语句块时，singleton这个静态变量已不为 null 了，所以当第二个线程进入synchronized时，还是会被第二重 singleton == null 挡在外面，而无法执行 new Singleton（）。

**所以在没有第一重 singleton == null 的情况下，也是可以实现单例模式的？那么为什么需要第一重 singleton == null 呢？**

这里就涉及一个性能问题了，因为对于单例模式的话，new Singleton（）只需要执行一次就 OK 了，而如果没有第一重 singleton == null 的话，每一次有线程进入 GetInstance（）时，均会执行锁定操作来实现线程同步，这是非常耗费性能的，而如果我加上第一重 singleton == null 的话，那么就只有在第一次，也就是 singleton ==null 成立时的情况下执行一次锁定以实现线程同步，而以后的话，便只要直接返回 Singleton 实例就 OK 了而根本无需再进入 lock 语句块了，这样就可以解决由线程同步带来的性能问题了。

## 解决同步关键字低效率

	public class SingletonClass { 
	    
	  private static class SingletonClassInstance { 
	    private static final SingletonClass instance = new SingletonClass(); 
	  } 
	
	  public static SingletonClass getInstance() { 
	    return SingletonClassInstance.instance; 
	  } 
	
	  private SingletonClass() { 
	
	  } 
	    
	}

使用了Java的静态内部类。这一技术是被JVM明确说明了的，因此不存在任何二义性。在这段代码中，因为SingletonClass没有static的属性，因此并不会被初始化。直到调用getInstance()的时候，会首先加载SingletonClassInstance类，这个类有一个static的SingletonClass实例，因此需要调用SingletonClass的构造方法，然后getInstance()将把这个内部类的instance返回给使用者。由于这个instance是static的，因此并不会构造多次。

由于SingletonClassInstance是私有静态内部类，所以不会被其他类知道，同样，static语义也要求不会有多个实例存在。并且，JSL规范定义，类的构造必须是原子性的，非并发的，因此不需要加同步块。同样，由于这个构造是非并发的，所以getInstance()也并不需要加同步。

Reference:[https://www.ibm.com/developerworks/cn/java/j-lo-Singleton/](https://www.ibm.com/developerworks/cn/java/j-lo-Singleton/)