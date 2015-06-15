title: HashMap
date: 2015-06-15 10:36:35
categories: Study Notes
tags: [HashMap]
---
HashMap 和 HashSet 是 Java Collection Framework 的两个重要成员，其中 HashMap 是 Map 接口的常用实现类，HashSet 是 Set 接口的常用实现类。虽然 HashMap 和 HashSet 实现的接口规范不同，但它们底层的 Hash 存储机制完全一样，甚至 HashSet 本身就采用 HashMap 来实现的。 
# Hash 存储机制
实际上，HashSet 和 HashMap 之间有很多相似之处，对于 HashSet 而言，系统采用 Hash 算法决定集合元素的存储位置，这样可以保证能快速存、取集合元素；对于 HashMap 而言，系统 key-value 当成一个整体进行处理，系统总是根据 Hash 算法来计算 key-value 的存储位置，这样可以保证能快速存、取 Map 的 key-value 对。

在介绍集合存储之前需要指出一点：虽然集合号称存储的是 Java 对象，但实际上并不会真正将 Java 对象放入 Set 集合中，只是在 Set 集合中保留这些对象的引用而言。也就是说：Java 集合实际上是多个引用变量所组成的集合，这些引用变量指向实际的 Java 对象。 
## 集合和引用
就像引用类型的数组一样，当我们把 Java 对象放入数组之时，并不是真正的把 Java 对象放入数组中，只是把对象的引用放入数组中，每个数组元素都是一个引用变量。 
## HashMap 的存储实现
当程序试图将多个 key-value 放入 HashMap 中时，以如下代码片段为例：
 
	HashMap<String , Double> map = new HashMap<String , Double>();   
	map.put("语文" , 80.0);   
	map.put("数学" , 89.0);   
	map.put("英语" , 78.2); 
HashMap 采用一种所谓的“Hash 算法”来决定每个元素的存储位置。 

当程序执行 map.put("语文" , 80.0); 时，系统将调用"语文"的 hashCode() 方法得到其 hashCode 值——每个 Java 对象都有 hashCode() 方法，都可通过该方法获得它的 hashCode 值。得到这个对象的 hashCode 值之后，系统会根据该 hashCode 值来决定该元素的存储位置。 

我们可以看 HashMap 类的 put(K key , V value) 方法的源代码： 

	public V put(K key, V value)   
	{   
	 // 如果 key 为 null，调用 putForNullKey 方法进行处理  
	 if (key == null)   
	     return putForNullKey(value);   
	 // 根据 key 的 keyCode 计算 Hash 值  
	 int hash = hash(key.hashCode());   
	 // 搜索指定 hash 值在对应 table 中的索引  
	     int i = indexFor(hash, table.length);  
	 // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素  
	 for (Entry<K,V> e = table[i]; e != null; e = e.next)   
	 {   
	     Object k;   
	     // 找到指定 key 与需要放入的 key 相等（hash 值相同  
	     // 通过 equals 比较放回 true）  
	     if (e.hash == hash && ((k = e.key) == key   
	         || key.equals(k)))   
	     {   
	         V oldValue = e.value;   
	         e.value = value;   
	         e.recordAccess(this);   
	         return oldValue;   
	     }   
	 }   
	 // 如果 i 索引处的 Entry 为 null，表明此处还没有 Entry   
	 modCount++;   
	 // 将 key、value 添加到 i 索引处  
	 addEntry(hash, key, value, i);   
	 return null;   
	}   
上面程序中用到了一个重要的内部接口：Map.Entry，每个 Map.Entry 其实就是一个 key-value 对。从上面程序中可以看出：当系统决定存储 HashMap 中的 key-value 对时，完全没有考虑 Entry 中的 value，仅仅只是根据 key 来计算并决定每个 Entry 的存储位置。这也说明了前面的结论：我们完全可以把 Map 集合中的 value 当成 key 的附属，当系统决定了 key 的存储位置之后，value 随之保存在那里即可。 

上面方法提供了一个根据 hashCode() 返回值来计算 Hash 码的方法：hash()，这个方法是一个纯粹的数学计算，其方法如下： 

	static int hash(int h)   
	{   
	    h ^= (h >>> 20) ^ (h >>> 12);   
	    return h ^ (h >>> 7) ^ (h >>> 4);   
	}  
对于任意给定的对象，只要它的 hashCode() 返回值相同，那么程序调用 hash(int h) 方法所计算得到的 Hash 码值总是相同的。接下来程序会调用 indexFor(int h, int length) 方法来计算该对象应该保存在 table 数组的哪个索引处。indexFor(int h, int length) 方法的代码如下： 

	static int indexFor(int h, int length)   
	{   
	    return h & (length-1);   
	} 
这个方法非常巧妙，它总是通过 h &(table.length -1) 来得到该对象的保存位置——而 HashMap 底层数组的长度总是 2 的 n 次方，这一点可参看后面关于 HashMap 构造器的介绍。 

当 length 总是 2 的倍数时，h & (length-1) 将是一个非常巧妙的设计：假设 h=5,length=16, 那么 h & length - 1 将得到 5；如果 h=6,length=16, 那么 h & length - 1 将得到 6 ……如果 h=15,length=16, 那么 h & length - 1 将得到 15；但是当 h=16 时 , length=16 时，那么 h & length - 1 将得到 0 了；当 h=17 时 , length=16 时，那么 h & length - 1 将得到 1 了……这样保证计算得到的索引值总是位于 table 数组的索引之内。 

根据上面 put 方法的源代码可以看出，当程序试图将一个 key-value 对放入 HashMap 中时，程序首先根据该 key 的 hashCode() 返回值决定该 Entry 的存储位置：如果两个 Entry 的 key 的 hashCode() 返回值相同，那它们的存储位置相同。如果这两个 Entry 的 key 通过 equals 比较返回 true，新添加 Entry 的 value 将覆盖集合中原有 Entry 的 value，但 key 不会覆盖。如果这两个 Entry 的 key 通过 equals 比较返回 false，新添加的 Entry 将与集合中原有 Entry 形成 Entry 链，而且新添加的 Entry 位于 Entry 链的头部——具体说明继续看 addEntry() 方法的说明。 

当向 HashMap 中添加 key-value 对，由其 key 的 hashCode() 返回值决定该 key-value 对（就是 Entry 对象）的存储位置。当两个 Entry 对象的 key 的 hashCode() 返回值相同时，将由 key 通过 eqauls() 比较值决定是采用覆盖行为（返回 true），还是产生 Entry 链（返回 false）。 

上面程序中还调用了 addEntry(hash, key, value, i); 代码，其中 addEntry 是 HashMap 提供的一个包访问权限的方法，该方法仅用于添加一个 key-value 对。下面是该方法的代码： 

	void addEntry(int hash, K key, V value, int bucketIndex)   
	{   
	    // 获取指定 bucketIndex 索引处的 Entry   
	    Entry<K,V> e = table[bucketIndex];     // ①  
	    // 将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry   
	    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);   
	    // 如果 Map 中的 key-value 对的数量超过了极限  
	    if (size++ >= threshold)   
	        // 把 table 对象的长度扩充到 2 倍。  
	        resize(2 * table.length);    // ②  
	}   
上面方法的代码很简单，但其中包含了一个非常优雅的设计：系统总是将新添加的 Entry 对象放入 table 数组的 bucketIndex 索引处——如果 bucketIndex 索引处已经有了一个 Entry 对象，那新添加的 Entry 对象指向原有的 Entry 对象（产生一个 Entry 链），如果 bucketIndex 索引处没有 Entry 对象，也就是上面程序①号代码的 e 变量是 null，也就是新放入的 Entry 对象指向 null，也就是没有产生 Entry 链。 
## Hash 算法的性能选项 
根据上面代码可以看出，在同一个 bucket 存储 Entry 链的情况下，新放入的 Entry 总是位于 bucket 中，而最早放入该 bucket 中的 Entry 则位于这个 Entry 链的最末端。 

上面程序中还有这样两个变量： 


> - size：该变量保存了该 HashMap 中所包含的 key-value 对的数量。 
- threshold：该变量包含了 HashMap 能容纳的 key-value 对的极限，它的值等于 HashMap 的容量乘以负载因子（load factor）。 

从上面程序中②号代码可以看出，当 size++ >= threshold 时，HashMap 会自动调用 resize 方法扩充 HashMap 的容量。每扩充一次，HashMap 的容量就增大一倍。 

上面程序中使用的 table 其实就是一个普通数组，每个数组都有一个固定的长度，这个数组的长度就是 HashMap 的容量。HashMap 包含如下几个构造器：

> - HashMap()：构建一个初始容量为 16，负载因子为 0.75 的 HashMap。 
> - HashMap(int initialCapacity)：构建一个初始容量为 initialCapacity，负载因子为 0.75 的 HashMap。 
> - HashMap(int initialCapacity, float loadFactor)：以指定初始容量、指定的负载因子创建一个 HashMap。 

当创建一个 HashMap 时，系统会自动创建一个 table 数组来保存 HashMap 中的 Entry，下面是 HashMap 中一个构造器的代码：

	// 以指定初始化容量、负载因子创建 HashMap   
	 public HashMap(int initialCapacity, float loadFactor)   
	 {   
	     // 初始容量不能为负数  
	     if (initialCapacity < 0)   
	         throw new IllegalArgumentException(   
	        "Illegal initial capacity: " +   
	             initialCapacity);   
	     // 如果初始容量大于最大容量，让出示容量  
	     if (initialCapacity > MAXIMUM_CAPACITY)   
	         initialCapacity = MAXIMUM_CAPACITY;   
	     // 负载因子必须大于 0 的数值  
	     if (loadFactor <= 0 || Float.isNaN(loadFactor))   
	         throw new IllegalArgumentException(   
	         loadFactor);   
	     // 计算出大于 initialCapacity 的最小的 2 的 n 次方值。  
	     int capacity = 1;   
	     while (capacity < initialCapacity)   
	         capacity <<= 1;   
	     this.loadFactor = loadFactor;   
	     // 设置容量极限等于容量 * 负载因子  
	     threshold = (int)(capacity * loadFactor);   
	     // 初始化 table 数组  
	     table = new Entry[capacity];            // ①  
	     init();   
	 }   

上面代码中粗体字代码包含了一个简洁的代码实现：找出大于 initialCapacity 的、最小的 2 的 n 次方值，并将其作为 HashMap 的实际容量（由 capacity 变量保存）。例如给定 initialCapacity 为 10，那么该 HashMap 的实际容量就是 16。 
程序①号代码处可以看到：table 的实质就是一个数组，一个长度为 capacity 的数组。 

对于 HashMap 及其子类而言，它们采用 Hash 算法来决定集合中元素的存储位置。当系统开始初始化 HashMap 时，系统会创建一个长度为 capacity 的 Entry 数组，这个数组里可以存储元素的位置被称为“桶（bucket）”，每个 bucket 都有其指定索引，系统可以根据其索引快速访问该 bucket 里存储的元素。 

无论何时，HashMap 的每个“桶”只存储一个元素（也就是一个 Entry），由于 Entry 对象可以包含一个引用变量（就是 Entry 构造器的的最后一个参数）用于指向下一个 Entry，因此可能出现的情况是：HashMap 的 bucket 中只有一个 Entry，但这个 Entry 指向另一个 Entry ——这就形成了一个 Entry 链。
## HashMap 的读取实现 
当 HashMap 的每个 bucket 里存储的 Entry 只是单个 Entry ——也就是没有通过指针产生 Entry 链时，此时的 HashMap 具有最好的性能：当程序通过 key 取出对应 value 时，系统只要先计算出该 key 的 hashCode() 返回值，在根据该 hashCode 返回值找出该 key 在 table 数组中的索引，然后取出该索引处的 Entry，最后返回该 key 对应的 value 即可。看 HashMap 类的 get(K key) 方法代码： 

	public V get(Object key)   
	{   
	 // 如果 key 是 null，调用 getForNullKey 取出对应的 value   
	 if (key == null)   
	     return getForNullKey();   
	 // 根据该 key 的 hashCode 值计算它的 hash 码  
	 int hash = hash(key.hashCode());   
	 // 直接取出 table 数组中指定索引处的值，  
	 for (Entry<K,V> e = table[indexFor(hash, table.length)];   
	     e != null;   
	     // 搜索该 Entry 链的下一个 Entr   
	     e = e.next)         // ①  
	 {   
	     Object k;   
	     // 如果该 Entry 的 key 与被搜索 key 相同  
	     if (e.hash == hash && ((k = e.key) == key   
	         || key.equals(k)))   
	         return e.value;   
	 }   
	 return null;   
	}   

从上面代码中可以看出，如果 HashMap 的每个 bucket 里只有一个 Entry 时，HashMap 可以根据索引、快速地取出该 bucket 里的 Entry；在发生“Hash 冲突”的情况下，单个 bucket 里存储的不是一个 Entry，而是一个 Entry 链，系统只能必须按顺序遍历每个 Entry，直到找到想搜索的 Entry 为止——如果恰好要搜索的 Entry 位于该 Entry 链的最末端（该 Entry 是最早放入该 bucket 中），那系统必须循环到最后才能找到该元素。 

归纳起来简单地说，HashMap 在底层将 key-value 当成一个整体进行处理，这个整体就是一个 Entry 对象。HashMap 底层采用一个 Entry[] 数组来保存所有的 key-value 对，当需要存储一个 Entry 对象时，会根据 Hash 算法来决定其存储位置；当需要取出一个 Entry 时，也会根据 Hash 算法找到其存储位置，直接取出该 Entry。由此可见：HashMap 之所以能快速存、取它所包含的 Entry，完全类似于现实生活中母亲从小教我们的：不同的东西要放在不同的位置，需要时才能快速找到它。 

当创建 HashMap 时，有一个默认的负载因子（load factor），其默认值为 0.75，这是时间和空间成本上一种折衷：增大负载因子可以减少 Hash 表（就是那个 Entry 数组）所占用的内存空间，但会增加查询数据的时间开销，而查询是最频繁的的操作（HashMap 的 get() 与 put() 方法都要用到查询）；减小负载因子会提高数据查询的性能，但会增加 Hash 表所占用的内存空间。 

掌握了上面知识之后，我们可以在创建 HashMap 时根据实际需要适当地调整 load factor 的值；如果程序比较关心空间开销、内存比较紧张，可以适当地增加负载因子；如果程序比较关心时间开销，内存比较宽裕则可以适当的减少负载因子。通常情况下，程序员无需改变负载因子的值。 

如果开始就知道 HashMap 会保存多个 key-value 对，可以在创建时就使用较大的初始化容量，如果 HashMap 中 Entry 的数量一直不会超过极限容量（capacity * load factor），HashMap 就无需调用 resize() 方法重新分配 table 数组，从而保证较好的性能。当然，开始就将初始容量设置太高可能会浪费空间（系统需要创建一个长度为 capacity 的 Entry 数组），因此创建 HashMap 时初始化容量设置也需要小心对待。