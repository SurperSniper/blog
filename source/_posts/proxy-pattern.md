title: 代理模式
date: 2015-06-17 14:25:51
categories: Design Pattern
tags: [proxy]
---
东汉末年，大将军何进引董卓入京，想借西北王的军队对抗阉党，无奈自己先被阉党做掉，而后造成巨变，导致诸侯并起，最终形成三国鼎立局面。汉献帝即位后，初平三年（公元 192 年），治中从事毛玠向曹操建议“奉天子以令不臣”，曹操采纳了他的建议，迎接汉献帝来到许昌。汉献帝刘协在许都没有实际的权利，曹操不断地诛除公卿大臣，不断地集军政大权于一身。建安元年八月，曹操进驻洛阳，立刻趁张杨、杨奉兵众在外，赶跑了韩暹，接着做了三件事：杀侍中台崇、尚书冯硕等，谓“讨有罪”；封董承、伏完等，谓“赏有功”；追赐射声校尉沮俊，谓“矜死节”。然后在第九天趁他人尚未来得及反应的情况下，迁帝都许，使皇帝摆脱其他势力的控制。此后，他还加紧步伐剪除异己，提高自己的权势。他首先向最有影响力的三公发难，罢免太尉杨彪、司空张喜；其次诛杀议郎赵彦；再次是发兵征讨杨奉，解除近兵之忧；最后是一方面以天子名义谴责袁绍，打击其气焰，另一方面将大将军让予袁绍，稳定大敌。这就是历史上著名的“挟天子以令诸侯”。汉献帝与曹操的关系，是历史上两位伟大的政治家的联手，稳定了东汉政权，最终平稳交接给曹魏政权，也间接映射了我们本文要讲解的“代理模式”。
# 代理模式
代理模式使用代理对象完成用户请求，屏蔽用户对真实对象的访问。现实世界的代理人被授权执行当事人的一些事宜，无需当事人出面，从第三方的角度看，似乎当事人并不存在，因为他只和代理人通信。而事实上代理人是要有当事人的授权，并且在核心问题上还需要请示当事人。

在软件设计中，使用代理模式的意图也很多，比如因为安全原因需要屏蔽客户端直接访问真实对象，或者在远程调用中需要使用代理类处理远程方法调用的技术细节 (如 RMI)，也可能为了提升系统性能，对真实对象进行封装，从而达到延迟加载的目的。

代理模式角色分为 4 种：
1. 主题接口：定义代理类和真实主题的公共对外方法，也是代理类代理真实主题的方法；
2. 真实主题：真正实现业务逻辑的类；
3. 代理类：用来代理和封装真实主题；
4. Main：客户端，使用代理类和主题接口完成一些工作。

# 延迟加载
以一个简单的示例来阐述使用代理模式实现延迟加载的方法及其意义。假设某客户端软件有根据用户请求去数据库查询数据的功能。在查询数据前，需要获得数据库连接，软件开启时初始化系统的所有类，此时尝试获得数据库连接。当系统有大量的类似操作存在时 (比如 XML 解析等)，所有这些初始化操作的叠加会使得系统的启动速度变得非常缓慢。为此，使用代理模式的代理类封装对数据库查询中的初始化操作，当系统启动时，初始化这个代理类，而非真实的数据库查询类，而代理类什么都没有做。因此，它的构造是相当迅速的。

在系统启动时，将消耗资源最多的方法都使用代理模式分离，可以加快系统的启动速度，减少用户的等待时间。而在用户真正做查询操作时再由代理类单独去加载真实的数据库查询类，完成用户的请求。这个过程就是使用代理模式实现了延迟加载。

延迟加载的核心思想是：如果当前并没有使用这个组件，则不需要真正地初始化它，使用一个代理对象替代它的原有的位置，只要在真正需要的时候才对它进行加载。使用代理模式的延迟加载是非常有意义的，首先，它可以在时间轴上分散系统压力，尤其在系统启动时，不必完成所有的初始化工作，从而加速启动时间；其次，对很多真实主题而言，在软件启动直到被关闭的整个过程中，可能根本不会被调用，初始化这些数据无疑是一种资源浪费。例如使用代理类封装数据库查询类后，系统的启动过程这个例子。若系统不使用代理模式，则在启动时就要初始化 DBQuery 对象，而使用代理模式后，启动时只需要初始化一个轻量级的对象 DBQueryProxy。

下面代码 IDBQuery 是主题接口，定义代理类和真实类需要对外提供的服务，定义了实现数据库查询的公共方法 request() 函数。DBQuery 是真实主题，负责实际的业务操作，DBQueryProxy 是 DBQuery 的代理类。
*清单 1. 延迟加载代理*

	public interface IDBQuery {
	 String request();
	}
	 public class DBQuery implements IDBQuery{
	 public DBQuery(){
	 try{
	 Thread.sleep(1000);//假设数据库连接等耗时操作
	 }catch(InterruptedException ex){
	 ex.printStackTrace();
	 }
	 }
	
	@Override
	public String request() {
	// TODO Auto-generated method stub
	return "request string";
	}
	 
	 
	}
	 public class DBQueryProxy implements IDBQuery{
	 private DBQuery real = null;
	 
	@Override
	public String request() {
	// TODO Auto-generated method stub
	//在真正需要的时候才能创建真实对象，创建过程可能很慢
	if(real==null){
	real = new DBQuery();
	}//在多线程环境下，这里返回一个虚假类，类似于 Future 模式
	return real.request();
	}
	 
	}
	 public class Main {
	 public static void main(String[] args){
	 IDBQuery q = new DBQueryProxy(); //使用代里
	 q.request(); //在真正使用时才创建真实对象
	 }
	}

# 动态代理
动态代理是指在运行时动态生成代理类。即，代理类的字节码将在运行时生成并载入当前代理的 ClassLoader。与静态处理类相比，动态类有诸多好处。首先，不需要为真实主题写一个形式上完全一样的封装类，假如主题接口中的方法很多，为每一个接口写一个代理方法也很麻烦。如果接口有变动，则真实主题和代理类都要修改，不利于系统维护；其次，使用一些动态代理的生成方法甚至可以在运行时制定代理类的执行逻辑，从而大大提升系统的灵活性。

动态代理类使用字节码动态生成加载技术，在运行时生成加载类。生成动态代理类的方法很多，如，JDK 自带的动态处理、CGLIB、Javassist 或者 ASM 库。JDK 的动态代理使用简单，它内置在 JDK 中，因此不需要引入第三方 Jar 包，但相对功能比较弱。CGLIB 和 Javassist 都是高级的字节码生成库，总体性能比 JDK 自带的动态代理好，而且功能十分强大。ASM 是低级的字节码生成工具，使用 ASM 已经近乎于在使用 Java bytecode 编程，对开发人员要求最高，当然，也是性能最好的一种动态代理生成工具。但 ASM 的使用很繁琐，而且性能也没有数量级的提升，与 CGLIB 等高级字节码生成工具相比，ASM 程序的维护性较差，如果不是在对性能有苛刻要求的场合，还是推荐 CGLIB 或者 Javassist。

以清单 1 所示代码中的 DBQueryProxy 为例，使用动态代理生成动态类，替换上例中的 DBQueryProxy。首先，使用 JDK 的动态代理生成代理对象。JDK 的动态代理需要实现一个处理方法调用的 Handler，用于实现代理方法的内部逻辑。
*清单 2. 动态代理*

	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	
	
	public class DBQueryHandler implements InvocationHandler{
	IDBQuery realQuery = null;//定义主题接口
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args)
	throws Throwable {
	// TODO Auto-generated method stub
	//如果第一次调用，生成真实主题
	if(realQuery == null){
	realQuery = new DBQuery();
	}
	//返回真实主题完成实际的操作
	return realQuery.request();
	}
	}

以上代码实现了一个 Handler，可以看到，它的内部逻辑和 DBQueryProxy 是类似的。在调用真实主题的方法前，先尝试生成真实主题对象。接着，需要使用这个 Handler 生成动态代理对象。代码如清单 3 所示。

*清单 3. 生成动态代理对象*

	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	
	
	public class DBQueryHandler implements InvocationHandler{
	IDBQuery realQuery = null;//定义主题接口
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args)
	throws Throwable {
	// TODO Auto-generated method stub
	//如果第一次调用，生成真实主题
	if(realQuery == null){
	realQuery = new DBQuery();
	}
	//返回真实主题完成实际的操作
	return realQuery.request();
	}
	
	public static IDBQuery createProxy(){
	IDBQuery proxy = (IDBQuery)Proxy.newProxyInstance(
	ClassLoader.getSystemClassLoader(), new Class[]{IDBQuery.class}, new DBQueryHandler()
	);
	return proxy;
	}
	}

以上代码生成了一个实现了 IDBQuery 接口的代理类，代理类的内部逻辑由 DBQueryHandler 决定。生成代理类后，由 newProxyInstance() 方法返回该代理类的一个实例。至此，一个完整的动态代理完成了。

在 Java 中，动态代理类的生成主要涉及对 ClassLoader 的使用。以 CGLIB 为例，使用 CGLIB 生成动态代理，首先需要生成 Enhancer 类实例，并指定用于处理代理业务的回调类。在 Enhancer.create() 方法中，会使用 DefaultGeneratorStrategy.Generate() 方法生成动态代理类的字节码，并保存在 byte 数组中。接着使用 ReflectUtils.defineClass() 方法，通过反射，调用 ClassLoader.defineClass() 方法，将字节码装载到 ClassLoader 中，完成类的加载。最后使用 ReflectUtils.newInstance() 方法，通过反射，生成动态类的实例，并返回该实例。基本流程是根据指定的回调类生成 Class 字节码—通过 defineClass() 将字节码定义为类—使用反射机制生成该类的实例。从清单 4 到清单 7 所示是使用 CGLIB 动态反射生成类的完整过程。
*清单 4. 定义接口*

	public interface BookProxy {
	public void addBook();
	}

*清单 5. 定义实现类*

	//该类并没有申明 BookProxy 接口
	 public class BookProxyImpl {
	public void addBook() { 
	System.out.println("增加图书的普通方法..."); 
	 } 
	}

*清单 6. 定义反射类及重载方法*

	import java.lang.reflect.Method; 
	
	import net.sf.cglib.proxy.Enhancer; 
	import net.sf.cglib.proxy.MethodInterceptor; 
	import net.sf.cglib.proxy.MethodProxy; 
	
	public class BookProxyLib implements MethodInterceptor {
	private Object target; 
	/** 
	 * 创建代理对象 
	 * 
	 * @param target 
	 * @return 
	*/ 
	public Object getInstance(Object target) { 
	 this.target = target; 
	 Enhancer enhancer = new Enhancer(); 
	 enhancer.setSuperclass(this.target.getClass()); 
	 // 回调方法 
	 enhancer.setCallback(this); 
	 // 创建代理对象 
	 return enhancer.create(); 
	} 
	 
	@Override 
	// 回调方法 
	public Object intercept(Object obj, Method method, Object[] args, 
	 MethodProxy proxy) throws Throwable { 
	 System.out.println("事物开始"); 
	 proxy.invokeSuper(obj, args); 
	 System.out.println("事物结束"); 
	 return null; 
	} 
	}

*清单 7. 运行程序*

	public class TestCglib { 
	 public static void main(String[] args) { 
	 BookProxyLib cglib=new BookProxyLib(); 
	 BookProxyImpl bookCglib=(BookProxyImpl)cglib.getInstance(new BookProxyImpl()); 
	 bookCglib.addBook(); 
	 } 
	}

*清单 8. 运行输出*

	事物开始
	增加图书的普通方法...
	事物结束

# 代理模式的应用场合
代理模式有多种应用场合，如下所述：

1. 远程代理，也就是为一个对象在不同的地址空间提供局部代表，这样可以隐藏一个对象存在于不同地址空间的事实。比如说 WebService，当我们在应用程序的项目中加入一个 Web 引用，引用一个 WebService，此时会在项目中声称一个 WebReference 的文件夹和一些文件，这个就是起代理作用的，这样可以让那个客户端程序调用代理解决远程访问的问题；
2. 虚拟代理，是根据需要创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象。这样就可以达到性能的最优化，比如打开一个网页，这个网页里面包含了大量的文字和图片，但我们可以很快看到文字，但是图片却是一张一张地下载后才能看到，那些未打开的图片框，就是通过虚拟代里来替换了真实的图片，此时代理存储了真实图片的路径和尺寸；
3. 安全代理，用来控制真实对象访问时的权限。一般用于对象应该有不同的访问权限的时候；
4. 指针引用，是指当调用真实的对象时，代理处理另外一些事。比如计算真实对象的引用次数，这样当该对象没有引用时，可以自动释放它，或当第一次引用一个持久对象时，将它装入内存，或是在访问一个实际对象前，检查是否已经释放它，以确保其他对象不能改变它。这些都是通过代理在访问一个对象时附加一些内务处理；
5. 延迟加载，用代理模式实现延迟加载的一个经典应用就在 Hibernate 框架里面。当 Hibernate 加载实体 bean 时，并不会一次性将数据库所有的数据都装载。默认情况下，它会采取延迟加载的机制，以提高系统的性能。Hibernate 中的延迟加载主要分为属性的延迟加载和关联表的延时加载两类。实现原理是使用代理拦截原有的 getter 方法，在真正使用对象数据时才去数据库或者其他第三方组件加载实际的数据，从而提升系统性能。

Reference:[http://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern](http://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern)