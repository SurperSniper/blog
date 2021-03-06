title: Java内存模型
date: 2015-06-18 15:24:57
categories: Study Notes
tags: [Java]
---
# Java内存模型（Java Memory Model）
Java内存模型（JMM），不同于Java运行时数据区，JMM的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中读取数据这样的底层细节。JMM规定了所有的变量都存储在主内存中，但每个线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝。线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的变量，工作内存是线程之间独立的，线程之间变量值的传递均需要通过主内存来完成。

当Java程序将变量同步到线程所在的内存，这时候会操作工作内存中的变量，而线程中变量的值何时同步回主内存是不可预期的。但同时Java内存模型又告诉我们通过使用关键词“synchronized”或“volatile”可以让Java保证某些约束：

> 1. “volatile” — 保证读写的都是主内存的变量
2. “synchronized” — 保证在块开始时都同步主内存的值到工作内存，而块结束时将变量同步回主内存

其实java的多线程并发问题最终都会反映在java的内存模型上，**所谓线程安全无非是要控制多个线程对某个资源的有序访问或修改**。总结java的内存模型，要解决两个主要的问题：可见性和有序性。

我们都知道计算机有高速缓存的存在，处理器并不是每次处理数据都是取内存的。JVM定义了自己的内存模型，屏蔽了底层平台内存管理细节，对于java开发人员，要清楚在jvm内存模型的基础上，如果解决多线程的可见性和有序性。

那么，**何谓可见性？ 多个线程之间是不能互相传递数据通信的，它们之间的沟通只能通过共享变量来进行**。Java内存模型（JMM）规定了jvm有主内存，**主内存是多个线程共享的**。当new一个对象的时候，也是被分配在主内存中，每个线程都有自己的工作内存，工作内存存储了主存的某些对象的副本，当然线程的工作内存大小是有限制的。当线程操作某个对象时，执行顺序如下：

1. 从主存复制变量到当前工作内存 (read and load)
2. 执行代码，改变共享变量值 (use and assign)
3. 用工作内存数据刷新主存相关内容 (store and write)

JVM规范定义了线程对主存的操作指令：read，load，use，assign，store，write。当一个共享变量在多个线程的工作内存中都有副本时，如果一个线程修改了这个共享变量，那么其他线程应该能够看到这个被修改后的值，这就是多线程的可见性问题。

 那么，**什么是有序性呢** ？线程在引用变量时不能直接从主内存中引用,如果线程工作内存中没有该变量,则会从主内存中拷贝一个副本到工作内存中,这个过程为read-load,完成后线程会引用该副本。当同一线程再度引用该字段时,有可能重新从主存中获取变量副本(read-load-use),也有可能直接引用原来的副本 (use),也就是说 read,load,use顺序可以由JVM实现系统决定。

线程不能直接为主存中中字段赋值，它会将值指定给工作内存中的变量副本(assign),完成后这个变量副本会同步到主存储区(store- write)，至于何时同步过去，根据JVM实现系统决定.

## synchronized关键字 
java用synchronized关键字做为多线程并发环境的执行有序性的保证手段之一。当一段代码会修改共享变量，这一段代码成为互斥区或临界区，为了保证共享变量的正确性，synchronized标示了临界区。典型的用法如下：
*Java代码 *

	synchronized(锁){   
	     临界区代码   
	}   

 理论上，每个对象都可以做为锁，但一个对象做为锁时，应该被多个线程共享，这样才显得有意义，在并发环境下，一个没有共享的对象作为锁是没有意义的。

每个锁对象都有两个队列，一个是就绪队列，一个是阻塞队列，就绪队列存储了将要获得锁的线程，阻塞队列存储了被阻塞的线程，当一个被线程被唤醒 (notify)后，才会进入到就绪队列，等待cpu的调度。当一开始线程a第一次执行account.add方法时，jvm会检查锁对象account 的就绪队列是否已经有线程在等待，如果有则表明account的锁已经被占用了，由于是第一次运行，account的就绪队列为空，所以线程a获得了锁，执行account.add方法。如果恰好在这个时候，线程b要执行account.withdraw方法，因为线程a已经获得了锁还没有释放，所以线程 b要进入account的就绪队列，等到得到锁后才可以执行。

*一个线程执行临界区代码过程如下：*

> 1. 获得同步锁
> 2. 清空工作内存
> 3. 从主存拷贝变量副本到工作内存
> 4. 对这些变量计算
> 5. 将变量从工作内存写回到主存
> 6. 释放锁

可见，synchronized既保证了多线程的并发有序性，又保证了多线程的内存可见性。

## volatile关键字 
volatile是java提供的一种同步手段，只不过它是轻量级的同步，为什么这么说，因为volatile只能保证多线程的内存可见性，不能保证多线程的执行有序性。而最彻底的同步要保证有序性和可见性，例如synchronized。任何被volatile修饰的变量，都不拷贝副本到工作内存，任何修改都及时写在主存。因此对于Valatile修饰的变量的修改，所有线程马上就能看到，但是volatile不能保证对变量的修改是有序的.
*Java代码 *

	public class VolatileTest{   
	  public volatile int a;   
	  public void add(int count){   
	       a=a+count;   
	  }   
	} 

当一个VolatileTest对象被多个线程共享，a的值不一定是正确的，因为a=a+count包含了好几步操作，而此时多个线程的执行是无序的，因为没有任何机制来保证多个线程的执行有序性和原子性。volatile存在的意义是，**任何线程对a的修改，都会马上被其他线程读取到，因为直接操作主存，没有线程对工作内存和主存的同步**。所以，volatile的使用场景是有限的，在有限的一些情形下可以使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全,必须同时满足下面两个条件:

1. 对变量的写操作不依赖于当前值。
2. 该变量没有包含在具有其他变量的不变式中 

volatile只保证了可见性，所以Volatile适合直接赋值的场景，如
* Java代码 *

	public class VolatileTest{   
	  public volatile int a;   
	  public void setA(int a){   
	      this.a=a;   
	  }   
	}  

在没有volatile声明时，多线程环境下，a的最终值不一定是正确的，因为this.a=a;涉及到给a赋值和将a同步回主存的步骤，这个顺序可能被打乱。如果用volatile声明了，读取主存副本到工作内存和同步a到主存的步骤，相当于是一个原子操作。所以简单来说，volatile适合这种场景：**一个变量被多个线程共享，线程直接给这个变量赋值。这是一种很简单的同步场景，这时候使用volatile的开销将会非常小。所谓线程的“工作内存”到底是个什么东西？有的人认为是线程的栈，其实这种理解是不正确的。看看JLS（java语言规范）对线程工作内存的描述，线程的working memory只是cpu的寄存器和高速缓存的抽象描述**。

cpu在计算的时候，并不总是从内存读取数据，它的数据读取顺序优先级是：寄存器－高速缓存－内存。线程耗费的是CPU，线程计算的时候，原始的数据来自内存，在计算过程中，有些数据可能被频繁读取，这些数据被存储在寄存器和高速缓存中，当线程计算完后，这些缓存的数据在适当的时候应该写回内存。当多个线程同时读写某个内存数据时，就会产生多线程并发问题，涉及到三个特性：原子性，有序性，可见性。支持多线程的平台都会面临这种问题，运行在多线程平台上支持多线程的语言应该提供解决该问题的方案。

JVM是一个虚拟的计算机，它也会面临多线程并发问题，java程序运行在java虚拟机平台上，java程序员不可能直接去控制底层线程对寄存器高速缓存内存之间的同步，那么java从语法层面，应该给开发人员提供一种解决方案，这个方案就是诸如 synchronized, volatile,锁机制（如同步块，就绪队列，阻塞队列）等等。这些方案只是语法层面的，但我们要从本质上去理解它，不能仅仅知道一个 synchronized 可以保证同步就完了。在这里说的是jvm的内存模型，是动态的，面向多线程并发的，沿袭JSL的“working memory”的说法,工作内存指的是寄存器和高速缓存的抽象描述.

JVM的静态内存储模型只是一种对内存的物理划分而已，它只局限在内存，而且只局限在JVM的内存.

1.程序计数器

> 每一个Java线程都有一个程序计数器来用于保存程序执行到当前方法的哪一个指令。

2.线程栈

> 线程的每个方法被执行的时候，都会同时创建一个帧（Frame）用于存储本地变量表、操作栈、动态链接、方法出入口等信息。每一个方法的调用至完成，就意味着一个帧在VM栈中的入栈至出栈的过程。如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；如果VM栈可以动态扩展（VM Spec中允许固定长度的VM栈），当扩展时无法申请到足够内存则抛出OutOfMemoryError异常。

3.本地方法栈

4.堆

> 每个线程的栈都是该线程私有的，堆则是所有线程共享的。当我们new一个对象时，该对象就被分配到了堆中。但是堆，并不是一个简单的概念，堆区又划分了很多区域，为什么堆划分成这么多区域，这是为了JVM的内存垃圾收集，似乎越扯越远了，扯到垃圾收集了，现在的jvm的gc都是按代收集，堆区大致被分为三大块：新生代，旧生代，持久代（虚拟的）；新生代又分为eden区，s0区，s1区。新建一个对象时，基本小的对象，生命周期短的对象都会放在新生代的eden区中，eden区满时，有一个小范围的gc（minor gc），整个新生代满时，会有一个大范围的gc（major gc），将新生代里的部分对象转到旧生代里。

5.方法区 

> 其实就是永久代（Permanent Generation），方法区中存放了每个Class的结构信息，包括常量池、字段描述、方法描述等等。VM Space描述中对这个区域的限制非常宽松，除了和Java堆一样不需要连续的内存，也可以选择固定大小或者可扩展外，甚至可以选择不实现垃圾收集。相对来说，垃圾收集行为在这个区域是相对比较少发生的，但并不是某些描述那样永久代不会发生GC（至 少对当前主流的商业JVM实现来说是如此），这里的GC主要是对常量池的回收和对类的卸载，虽然回收的“成绩”一般也比较差强人意，尤其是类卸载，条件相当苛刻。

6.常量池
	
> Class文件中除了有类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量表(constant_pool table)，用于存放编译期已可知的常量，这部分内容将在类加载后进入方法区（永久代）存放。但是Java语言并不要求常量一定只有编译期预置入Class的常量表的内容才能进入方法区常量池，运行期间也可将新内容放入常量池（最典型的String.intern()方法）。  