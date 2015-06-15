title: 垃圾收集
date: 2015-06-09 23:44:47
categories: Study Notes
tags: [Java]
---
# 垃圾收集

- **垃圾收集并不等于“破坏”！**

	Java可用垃圾收集器回收由不再使用的对象占据的内存。现在考虑一种非常特殊且不多见的情况。假定我们的对象分配了一个“特殊”内存区域，没有使用new。垃圾收集器只知道释放那些由new分配的内存，所以不知道如何释放对象的“特殊”内存。为解决这个问题，Java提供了一个名为finalize()的方法，可为我们的类定义它。在理想情况下，它的工作原理应该是这样的：一旦垃圾收集器准备好释放对象占用的存储空间，它首先调用finalize()，而且只有在下一次垃圾收集过程中，才会真正回收对象的内存。所以如果使用finalize()，就可以在垃圾收集期间进行一些重要的清除或清扫工作。

	但也是一个潜在的编程陷阱，因为有些程序员（特别是在C++开发背景的）刚开始可能会错误认为它就是在C++中为“破坏器”（Destructor）使用的finalize()——破坏（清除）一个对象的时候，肯定会调用这个函数。但在这里有必要区分一下C++和Java的区别，因为C++的对象肯定会被清除（排开编程错误的因素），而Java对象并非肯定能作为垃圾被“收集”去
- **我们的对象可能不会当作垃圾被收掉！**
	
	有时可能发现一个对象的存储空间永远都不会释放，因为自己的程序永远都接近于用光空间的临界点。若程序执行结束，而且垃圾收集器一直都没有释放我们创建的任何对象的存储空间，则随着程序的退出，那些资源会返回给操作系统。这是一件好事情，因为垃圾收集本身也要消耗一些开销。如永远都不用它，那么永远也不用支出这部分开销。
- **垃圾收集只跟内存有关！**

	垃圾收集器存在的唯一原因是为了回收程序不再使用的内存。所以对于与垃圾收集有关的任何活动来说，其中最值得注意的是finalize()方法，它们也必须同内存以及它的回收有关。
	但这是否意味着假如对象包含了其他对象，finalize()就应该明确释放那些对象呢？答案是否定的——垃圾收集器会负责释放所有对象占据的内存，无论这些对象是如何创建的。它将对finalize()的需求限制到特殊的情况。在这种情况下，我们的对象可采用与创建对象时不同的方法分配一些存储空间。但大家或许会注意到，Java中的所有东西都是对象，所以这到底是怎么一回事呢？

	之所以要使用finalize()，看起来似乎是由于有时需要采取与Java的普通方法不同的一种方法，通过分配内存来做一些具有C风格的事情。这主要可以通过“固有方法”来进行，它是从Java里调用非Java方法的一种方式（固有方法的问题在附录A讨论）。C和C++是目前唯一获得固有方法支持的语言。但由于它们能调用通过其他语言编写的子程序，所以能够有效地调用任何东西。在非Java代码内部，也许能调用C的malloc()系列函数，用它分配存储空间。而且除非调用了free()，否则存储空间不会得到释放，从而造成内存“漏洞”的出现。当然，free()是一个C和C++函数，所以我们需要在finalize()内部的一个固有方法中调用它。
	
	读完上述文字后，大家或许已弄清楚了自己不必过多地使用finalize()。这个思想是正确的；它并不是进行普通清除工作的理想场所。
	> finalize()最有用处的地方之一是观察垃圾收集的过程

# 初始化

----------

## 初始化顺序
在一个类里，初始化的顺序是由变量在类内的定义顺序决定的。即使变量定义大量遍布于方法定义的中间，那些变量仍会在调用任何方法之前得到初始化——甚至在构建器调用之前。

    //: OrderOfInitialization.java
    // Demonstrates initialization order.
    // When the constructor is called, to create a
    // Tag object, you'll see a message:
    class Tag {
    Tag(int marker) {
    System.out.println("Tag(" + marker + ")");
    }
    }
    class Card {
    Tag t1 = new Tag(1); // Before constructor
    Card() {
    // Indicate we're in the constructor:
    System.out.println("Card()");
    t3 = new Tag(33); // Re-initialize t3
    }
    Tag t2 = new Tag(2); // After constructor
    void f() {
    System.out.println("f()");
    }
    Tag t3 = new Tag(3); // At end
    }
    public class OrderOfInitialization {
    public static void main(String[] args) {
    Card t = new Card();
    t.f(); // Shows that construction is done
    }}
	//输出结果
    Tag(1)
    Tag(2)
    Tag(3)
    Card()
    Tag(33)
    f()
## 静态数据的初始化
若数据是静态的（static），那么同样的事情就会发生；如果它属于一个基本类型（主类型），而且未对其初始化，就会自动获得自己的标准基本类型初始值；如果它是指向一个对象的句柄，那么除非新建一个对象，并将句柄同它连接起来，否则就会得到一个空值（NULL）。
如果想在定义的同时进行初始化，采取的方法与非静态值表面看起来是相同的。但由于static值只有一个存储区域，所以无论创建多少个对象，都必然会遇到何时对那个存储区域进行初始化的问题。

    //: StaticInitialization.java
    // Specifying initial values in a
    // class definition.
    class Bowl {
    Bowl(int marker) {
    System.out.println("Bowl(" + marker + ")");
    }
    void f(int marker) {
    System.out.println("f(" + marker + ")");
    }
    }
    class Table {
    static Bowl b1 = new Bowl(1);
    Table() {
    System.out.println("Table()");
    b2.f(1);
    }
    void f2(int marker) {
    System.out.println("f2(" + marker + ")");
    }
    static Bowl b2 = new Bowl(2);
    }
    class Cupboard {
    Bowl b3 = new Bowl(3);
    113
    static Bowl b4 = new Bowl(4);
    Cupboard() {
    System.out.println("Cupboard()");
    b4.f(2);
    }
    void f3(int marker) {
    System.out.println("f3(" + marker + ")");
    }
    static Bowl b5 = new Bowl(5);
    }
    public class StaticInitialization {
    public static void main(String[] args) {
    System.out.println(
    "Creating new Cupboard() in main");
    new Cupboard();
    System.out.println(
    "Creating new Cupboard() in main");
    new Cupboard();
    t2.f2(1);
    t3.f3(1);
    }
    static Table t2 = new Table();
    static Cupboard t3 = new Cupboard();
    }
	//输出结果
    Bowl(1)
    Bowl(2)
    Table()
    f(1)
    Bowl(4)
    Bowl(5)
    Bowl(3)
    Cupboard()
    f(2)
    Creating new Cupboard() in main
    Bowl(3)
    Cupboard()
    f(2)
    Creating new Cupboard() in main
    Bowl(3)
    Cupboard()
    f(2)
    f2(1)
    f3(1)
> static 初始化只有在必要的时候才会进行。如果不创建一个Table 对象，而且永远都不引用Table.b1 或
Table.b2，那么static Bowl b1 和b2 永远都不会创建。然而，只有在创建了第一个Table 对象之后（或者
发生了第一次static 访问），它们才会创建。在那以后，static 对象不会重新初始化。

> 初始化的顺序是首先static（如果它们尚未由前一次对象创建过程初始化），接着是非static对象。大家可从输出结果中找到相应的证据。

## 对象的创建过程
请考虑一个名为Dog的类：

1.  类型为Dog的一个对象首次创建时，或者Dog类的static方法／static字段首次访问时，Java解释器必须找到Dog.class（在事先设好的类路径里搜索）。
2. 找到Dog.class后（它会创建一个Class对象），它的所有static初始化模块都会运行。因此，static初始化仅发生一次——在Class对象首次载入的时候。
3. 创建一个new Dog()时，Dog对象的构建进程首先会在内存堆（Heap）里为一个Dog对象分配足够多的存储空间。
4. 这种存储空间会清为零，将Dog中的所有基本类型设为它们的默认值（零用于数字，以及boolean和char的等价设定）。
5. 进行字段定义时发生的所有初始化都会执行。
6. 执行构建器。