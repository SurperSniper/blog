title: Java 程序优化：字符串操作、基本运算方法等优化策略
date: 2015-06-17 15:59:18
categories: Study Notes
tags: [Java]
---
# 字符串操作优化
**字符串对象**

字符串对象或者其等价对象 (如 char 数组)，在内存中总是占据最大的空间块，因此如何高效地处理字符串，是提高系统整体性能的关键。

String 对象可以认为是 char 数组的延伸和进一步封装，它主要由 3 部分组成：char 数组、偏移量和 String 的长度。char 数组表示 String 的内容，它是 String 对象所表示字符串的超集。String 的真实内容还需要由偏移量和长度在这个 char 数组中进行定位和截取。

String 有 3 个基本特点：
1. 不变性
2. 针对常量池的优化
3. 类的 final 定义

不变性指的是 String 对象一旦生成，则不能再对它进行改变。String 的这个特性可以泛化成不变 (immutable) 模式，即一个对象的状态在对象被创建之后就不再发生变化。不变模式的主要作用在于当一个对象需要被多线程共享，并且访问频繁时，可以省略同步和锁等待的时间，从而大幅提高系统性能。

针对常量池的优化指的是当两个 String 对象拥有相同的值时，它们只引用常量池中的同一个拷贝，当同一个字符串反复出现时，这个技术可以大幅度节省内存空间。

下面代码 str1、str2、str4 引用了相同的地址，但是 str3 却重新开辟了一块内存空间，虽然 str3 单独占用了堆空间，但是它所指向的实体和 str1 完全一样。代码如下清单 1 所示。
*清单 1. 示例代码*

	public class StringDemo {
	 public static void main(String[] args){
	 String str1 = "abc";
	 String str2 = "abc";
	 String str3 = new String("abc");
	 String str4 = str1;
	 System.out.println("is str1 = str2?"+(str1==str2));
	 System.out.println("is str1 = str3?"+(str1==str3));
	 System.out.println("is str1 refer to str3?"+(str1.intern()==str3.intern()));
	 System.out.println("is str1 = str4"+(str1==str4));
	 System.out.println("is str2 = str4"+(str2==str4));
	 System.out.println("is str4 refer to str3?"+(str4.intern()==str3.intern()));
	 }
	}

输出如清单 2 所示。

*清单 2. 输出结果*

	is str1 = str2?true
	is str1 = str3?false
	is str1 refer to str3?true
	is str1 = str4true
	is str2 = str4true
	is str4 refer to str3?true

**SubString 使用技巧**

String 的 substring 方法源码在最后一行新建了一个 String 对象，new String(offset+beginIndex,endIndex-beginIndex,value);该行代码的目的是为了能高效且快速地共享 String 内的 char 数组对象。但在这种通过偏移量来截取字符串的方法中，String 的原生内容 value 数组被复制到新的子字符串中。设想，如果原始字符串很大，截取的字符长度却很短，那么截取的子字符串中包含了原生字符串的所有内容，并占据了相应的内存空间，而仅仅通过偏移量和长度来决定自己的实际取值。这种算法提高了速度却浪费了空间。

下面代码演示了使用 substring 方法在一个很大的 string 独享里面截取一段很小的字符串，如果采用 string 的 substring 方法会造成内存溢出，如果采用反复创建新的 string 方法可以确保正常运行。
*清单 3.substring 方法演示*

	import java.util.ArrayList;
	import java.util.List;
	
	public class StringDemo {
	 public static void main(String[] args){
	 List<String> handler = new ArrayList<String>();
	 for(int i=0;i<1000;i++){
	 HugeStr h = new HugeStr();
	 ImprovedHugeStr h1 = new ImprovedHugeStr();
	 handler.add(h.getSubString(1, 5));
	 handler.add(h1.getSubString(1, 5));
	 }
	 }
	 
	 static class HugeStr{
	 private String str = new String(new char[800000]);
	 public String getSubString(int begin,int end){
	 return str.substring(begin, end);
	 }
	 }
	 
	 static class ImprovedHugeStr{
	 private String str = new String(new char[10000000]);
	 public String getSubString(int begin,int end){
	 return new String(str.substring(begin, end));
	 }
	 }
	}

输出结果如清单 4 所示。

*清单 4. 输出结果*

	Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Unknown Source)
	at java.lang.StringValue.from(Unknown Source)
	at java.lang.String.<init>(Unknown Source)
	at StringDemo$ImprovedHugeStr.<init>(StringDemo.java:23)
	at StringDemo.main(StringDemo.java:9)

ImprovedHugeStr 可以工作是因为它使用没有内存泄漏的 String 构造函数重新生成了 String 对象，使得由 substring() 方法返回的、存在内存泄漏问题的 String 对象失去所有的强引用，从而被垃圾回收器识别为垃圾对象进行回收，保证了系统内存的稳定。

String 的 split 方法支持传入正则表达式帮助处理字符串，但是简单的字符串分割时性能较差。

对比 split 方法和 StringTokenizer 类的处理字符串性能，代码如清单 5 所示。

**切分字符串方式讨论**

String 的 split 方法支持传入正则表达式帮助处理字符串，操作较为简单，但是缺点是它所依赖的算法在对简单的字符串分割时性能较差。清单 5 所示代码对比了 String 的 split 方法和调用 StringTokenizer 类来处理字符串时性能的差距。
*清单 5.String 的 split 方法演示*

	import java.util.StringTokenizer;
	
	
	public class splitandstringtokenizer {
	 public static void main(String[] args){
	 String orgStr = null;
	 StringBuffer sb = new StringBuffer();
	 for(int i=0;i<100000;i++){
	 sb.append(i);
	 sb.append(",");
	 }
	 orgStr = sb.toString();
	 long start = System.currentTimeMillis();
	 for(int i=0;i<100000;i++){
	 orgStr.split(",");
	 }
	 long end = System.currentTimeMillis();
	 System.out.println(end-start);
	 
	 start = System.currentTimeMillis();
	 String orgStr1 = sb.toString();
	 StringTokenizer st = new StringTokenizer(orgStr1,",");
	 for(int i=0;i<100000;i++){
	 st.nextToken();
	 }
	 st = new StringTokenizer(orgStr1,",");
	 end = System.currentTimeMillis();
	 System.out.println(end-start);
	 
	 start = System.currentTimeMillis();
	 String orgStr2 = sb.toString();
	 String temp = orgStr2;
	 while(true){
	 String splitStr = null;
	 int j=temp.indexOf(",");
	 if(j<0)break;
	 splitStr=temp.substring(0, j);
	 temp = temp.substring(j+1);
	 }
	 temp=orgStr2;
	 end = System.currentTimeMillis();
	 System.out.println(end-start);
	 }
	}

输出如清单 6 所示：
*清单 6. 运行输出结果*

	39015
	16
	15

当一个 StringTokenizer 对象生成后，通过它的 nextToken() 方法便可以得到下一个分割的字符串，通过 hasMoreToken 方法可以知道是否有更多的字符串需要处理。对比发现 split 的耗时非常的长，采用 StringTokenizer 对象处理速度很快。我们尝试自己实现字符串分割算法，使用 substring 方法和 indexOf 方法组合而成的字符串分割算法可以帮助很快切分字符串并替换内容。

由于 String 是不可变对象，因此，在需要对字符串进行修改操作时 (如字符串连接、替换)，String 对象会生成新的对象，所以其性能相对较差。但是 JVM 会对代码进行彻底的优化，将多个连接操作的字符串在编译时合成一个单独的长字符串。

以上实例运行结果差异较大的原因是 split 算法对每一个字符进行了对比，这样当字符串较大时，需要把整个字符串读入内存，逐一查找，找到符合条件的字符，这样做较为耗时。而 StringTokenizer 类允许一个应用程序进入一个令牌（tokens），StringTokenizer 类的对象在内部已经标识化的字符串中维持了当前位置。一些操作使得在现有位置上的字符串提前得到处理。 一个令牌的值是由获得其曾经创建 StringTokenizer 类对象的字串所返回的。
*清单 7.split 类源代码*

	import java.util.ArrayList;
	
	public class Split {
	public String[] split(CharSequence input, int limit) { 
	int index = 0; 
	boolean matchLimited = limit > 0; 
	ArrayList<String> matchList = new ArrayList<String>(); 
	Matcher m = matcher(input); 
	// Add segments before each match found 
	while(m.find()) { 
	if (!matchLimited || matchList.size() < limit - 1) { 
	String match = input.subSequence(index, m.start()).toString(); 
	matchList.add(match); 
	index = m.end(); 
	} else if (matchList.size() == limit - 1) { 
	// last one 
	String match = input.subSequence(index,input.length()).toString(); 
	matchList.add(match); 
	index = m.end(); 
	} 
	} 
	// If no match was found, return this 
	if (index == 0){ 
	return new String[] {input.toString()}; 
	}
	// Add remaining segment 
	if (!matchLimited || matchList.size() < limit){ 
	matchList.add(input.subSequence(index, input.length()).toString()); 
	}
	// Construct result 
	int resultSize = matchList.size(); 
	if (limit == 0){ 
	while (resultSize > 0 && matchList.get(resultSize-1).equals("")) 
	resultSize--; 
	 String[] result = new String[resultSize]; 
	 return matchList.subList(0, resultSize).toArray(result); 
	}
	}
	}

split 借助于数据对象及字符查找算法完成了数据分割，适用于数据量较少场景。

**合并字符串**

由于 String 是不可变对象，因此，在需要对字符串进行修改操作时 (如字符串连接、替换)，String 对象会生成新的对象，所以其性能相对较差。但是 JVM 会对代码进行彻底的优化，将多个连接操作的字符串在编译时合成一个单独的长字符串。针对超大的 String 对象，我们采用 String 对象连接、使用 concat 方法连接、使用 StringBuilder 类等多种方式，代码如清单 8 所示。
*清单 8. 处理超大 String 对象的示例代码*

	public class StringConcat {
	 public static void main(String[] args){
	 String str = null;
	 String result = "";
	 
	 long start = System.currentTimeMillis();
	 for(int i=0;i<10000;i++){
	 str = str + i;
	 }
	 long end = System.currentTimeMillis();
	 System.out.println(end-start);
	 
	 start = System.currentTimeMillis();
	 for(int i=0;i<10000;i++){
	 result = result.concat(String.valueOf(i));
	 }
	 end = System.currentTimeMillis();
	 System.out.println(end-start);
	 
	 start = System.currentTimeMillis();
	 StringBuilder sb = new StringBuilder();
	 for(int i=0;i<10000;i++){
	 sb.append(i);
	 }
	 end = System.currentTimeMillis();
	 System.out.println(end-start);
	 }
	}
输出如清单 9 所示。
*清单 9. 运行输出结果*

	375
	187
	0
虽然第一种方法编译器判断 String 的加法运行成 StringBuilder 实现，但是编译器没有做出足够聪明的判断，每次循环都生成了新的 StringBuilder 实例从而大大降低了系统性能。

StringBuffer 和 StringBuilder 都实现了 AbstractStringBuilder 抽象类，拥有几乎相同的对外借口，两者的最大不同在于 StringBuffer 对几乎所有的方法都做了同步，而 StringBuilder 并没有任何同步。由于方法同步需要消耗一定的系统资源，因此，StringBuilder 的效率也好于 StringBuffer。 但是，在多线程系统中，StringBuilder 无法保证线程安全，不能使用。代码如清单 10 所示。
*清单 10.StringBuilderVSStringBuffer*

	public class StringBufferandBuilder {
	public StringBuffer contents = new StringBuffer(); 
	public StringBuilder sbu = new StringBuilder();
	
	public void log(String message){ 
	for(int i=0;i<10;i++){ 
	/*
	contents.append(i); 
	contents.append(message); 
	contents.append("\n"); 
	*/
	contents.append(i);
	contents.append("\n");
	sbu.append(i);
	sbu.append("\n");
	} 
	} 
	public void getcontents(){ 
	//System.out.println(contents); 
	System.out.println("start print StringBuffer");
	System.out.println(contents); 
	System.out.println("end print StringBuffer");
	}
	public void getcontents1(){ 
	//System.out.println(contents); 
	System.out.println("start print StringBuilder");
	System.out.println(sbu); 
	System.out.println("end print StringBuilder");
	}
	
	 
	 
	 public static void main(String[] args) throws InterruptedException { 
	StringBufferandBuilder ss = new StringBufferandBuilder(); 
	runthread t1 = new runthread(ss,"love");
	runthread t2 = new runthread(ss,"apple");
	runthread t3 = new runthread(ss,"egg");
	t1.start();
	t2.start();
	t3.start();
	t1.join();
	t2.join();
	t3.join();
	}
	 
	}
	
	class runthread extends Thread{ 
	String message; 
	StringBufferandBuilder buffer; 
	public runthread(StringBufferandBuilder buffer,String message){ 
	this.buffer = buffer;
	this.message = message; 
	} 
	public void run(){ 
	while(true){ 
	buffer.log(message); 
	//buffer.getcontents();
	buffer.getcontents1();
	try {
	sleep(5000000);
	} catch (InterruptedException e) {
	// TODO Auto-generated catch block
	e.printStackTrace();
	}
	} 
	} 
	
	
	}
输出结果如清单 11 所示。
*清单 11. 运行结果*

	start print StringBuffer
	0123456789
	end print StringBuffer
	start print StringBuffer
	start print StringBuilder
	01234567890123456789
	end print StringBuffer
	start print StringBuilder
	01234567890123456789
	01234567890123456789
	end print StringBuilder
	end print StringBuilder
	start print StringBuffer
	012345678901234567890123456789
	end print StringBuffer
	start print StringBuilder
	012345678901234567890123456789
	end print StringBuilder

StringBuilder 数据并没有按照预想的方式进行操作。StringBuilder 和 StringBuffer 的扩充策略是将原有的容量大小翻倍，以新的容量申请内存空间，建立新的 char 数组，然后将原数组中的内容复制到这个新的数组中。因此，对于大对象的扩容会涉及大量的内存复制操作。如果能够预先评估大小，会提高性能。

# 数据定义、运算逻辑优化
## 使用局部变量
调用方法时传递的参数以及在调用中创建的临时变量都保存在栈 (Stack) 里面，读写速度较快。其他变量，如静态变量、实例变量等，都在堆 (heap) 中创建，读写速度较慢。清单 12 所示代码演示了使用局部变量和静态变量的操作时间对比。
*清单 12. 局部变量 VS 静态变量*

	public class variableCompare {
	public static int b = 0;
	 public static void main(String[] args){
	 int a = 0;
	 long starttime = System.currentTimeMillis();
	 for(int i=0;i<1000000;i++){
	 a++;//在函数体内定义局部变量
	 }
	 System.out.println(System.currentTimeMillis() - starttime);
	 
	 starttime = System.currentTimeMillis();
	 for(int i=0;i<1000000;i++){
	 b++;//在函数体内定义局部变量
	 }
	 System.out.println(System.currentTimeMillis() - starttime);
	 }
	}
运行后输出如清单 13 所示。
*清单 13. 运行结果*

	0
	15

以上两段代码的运行时间分别为 0ms 和 15ms。由此可见，局部变量的访问速度远远高于类的成员变量。
##位运算代替乘除法
位运算是所有的运算中最为高效的。因此，可以尝试使用位运算代替部分算数运算，来提高系统的运行速度。最典型的就是对于整数的乘除运算优化。清单 14 所示代码是一段使用算数运算的实现。
*清单 14. 算数运算*

	public class yunsuan {
	 public static void main(String args[]){
	 long start = System.currentTimeMillis();
	 long a=1000;
	 for(int i=0;i<10000000;i++){
	 a*=2;
	 a/=2;
	 }
	 System.out.println(a);
	 System.out.println(System.currentTimeMillis() - start);
	 start = System.currentTimeMillis();
	 for(int i=0;i<10000000;i++){
	 a<<=1;
	 a>>=1;
	 }
	 System.out.println(a);
	 System.out.println(System.currentTimeMillis() - start);
	 }
	}
运行输出如清单 15 所示。
*清单 15. 运行结果*

	1000
	546
	1000
	63

两段代码执行了完全相同的功能，在每次循环中，整数 1000 乘以 2，然后除以 2。第一个循环耗时 546ms，第二个循环耗时 63ms。

## 替换 switch
关键字 switch 语句用于多条件判断，switch 语句的功能类似于 if-else 语句，两者的性能差不多。但是 switch 语句有性能提升空间。清单 16 所示代码演示了 Switch 与 if-else 之间的对比。
*清单 16.Switch 示例*

	public class switchCompareIf {
	
	public static int switchTest(int value){
	int i = value%10+1;
	switch(i){
	case 1:return 10;
	case 2:return 11;
	case 3:return 12;
	case 4:return 13;
	case 5:return 14;
	case 6:return 15;
	case 7:return 16;
	case 8:return 17;
	case 9:return 18;
	default:return -1;
	}
	}
	
	public static int arrayTest(int[] value,int key){
	int i = key%10+1;
	if(i>9 || i<1){
	return -1;
	}else{
	return value[i];
	}
	}
	
	 public static void main(String[] args){
	 int chk = 0;
	 long start=System.currentTimeMillis();
	 for(int i=0;i<10000000;i++){
	 chk = switchTest(i);
	 }
	 System.out.println(System.currentTimeMillis()-start);
	 chk = 0;
	 start=System.currentTimeMillis();
	 int[] value=new int[]{0,10,11,12,13,14,15,16,17,18};
	 for(int i=0;i<10000000;i++){
	 chk = arrayTest(value,i);
	 }
	 System.out.println(System.currentTimeMillis()-start);
	 }
	}

运行输出如清单 17 所示。
*清单 17. 运行结果*

	172
	93
使用一个连续的数组代替 switch 语句，由于对数据的随机访问非常快，至少好于 switch 的分支判断，从上面例子可以看到比较的效率差距近乎 1 倍，switch 方法耗时 172ms，if-else 方法耗时 93ms。
## 一维数组代替二维数组
JDK 很多类库是采用数组方式实现的数据存储，比如 ArrayList、Vector 等，数组的优点是随机访问性能非常好。一维数组和二维数组的访问速度不一样，一维数组的访问速度要优于二维数组。在性能敏感的系统中要使用二维数组，尽量将二维数组转化为一维数组再进行处理，以提高系统的响应速度。
*清单 18. 数组方式对比*

	public class arrayTest {
	 public static void main(String[] args){
	 long start = System.currentTimeMillis();
	 int[] arraySingle = new int[1000000];
	 int chk = 0;
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arraySingle.length;j++){
	 arraySingle[j] = j;
	 }
	 }
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arraySingle.length;j++){
	 chk = arraySingle[j];
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 
	 start = System.currentTimeMillis();
	 int[][] arrayDouble = new int[1000][1000];
	 chk = 0;
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arrayDouble.length;j++){
	 for(int k=0;k<arrayDouble[0].length;k++){
	 arrayDouble[i][j]=j;
	 }
	 }
	 }
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arrayDouble.length;j++){
	 for(int k=0;k<arrayDouble[0].length;k++){
	 chk = arrayDouble[i][j];
	 }
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 
	 start = System.currentTimeMillis();
	 arraySingle = new int[1000000];
	 int arraySingleSize = arraySingle.length;
	 chk = 0;
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arraySingleSize;j++){
	 arraySingle[j] = j;
	 }
	 }
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arraySingleSize;j++){
	 chk = arraySingle[j];
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 
	 start = System.currentTimeMillis();
	 arrayDouble = new int[1000][1000];
	 int arrayDoubleSize = arrayDouble.length;
	 int firstSize = arrayDouble[0].length;
	 chk = 0;
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arrayDoubleSize;j++){
	 for(int k=0;k<firstSize;k++){
	 arrayDouble[i][j]=j;
	 }
	 }
	 }
	 for(int i=0;i<100;i++){
	 for(int j=0;j<arrayDoubleSize;j++){
	 for(int k=0;k<firstSize;k++){
	 chk = arrayDouble[i][j];
	 }
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 }
	}

*清单 19. 运行结果*

	343
	624
	287
	390
第一段代码操作的是一维数组的赋值、取值过程，第二段代码操作的是二维数组的赋值、取值过程。可以看到一维数组方式比二维数组方式快接近一半时间。而对于数组内如果可以减少赋值运算，则可以进一步减少运算耗时，加快程序运行速度。
## 提取表达式
大部分情况下，代码的重复劳动由于计算机的高速运行，并不会对性能构成太大的威胁，但若希望将系统性能发挥到极致，还是有很多地方可以优化的。
*清单 20. 提取表达式*

	public class duplicatedCode {
	 public static void beforeTuning(){
	 long start = System.currentTimeMillis();
	 double a1 = Math.random();
	 double a2 = Math.random();
	 double a3 = Math.random();
	 double a4 = Math.random();
	 double b1,b2;
	 for(int i=0;i<10000000;i++){
	 b1 = a1*a2*a4/3*4*a3*a4;
	 b2 = a1*a2*a3/3*4*a3*a4;
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 }
	 
	 public static void afterTuning(){
	 long start = System.currentTimeMillis();
	 double a1 = Math.random();
	 double a2 = Math.random();
	 double a3 = Math.random();
	 double a4 = Math.random();
	 double combine,b1,b2;
	 for(int i=0;i<10000000;i++){
	 combine = a1*a2/3*4*a3*a4;
	 b1 = combine*a4;
	 b2 = combine*a3;
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 }
	 
	 public static void main(String[] args){
	 duplicatedCode.beforeTuning();
	 duplicatedCode.afterTuning();
	 }
	}
运行输出如清单 21 所示。
*清单 21. 运行结果*

	202
	110
两段代码的差别是提取了重复的公式，使得这个公式的每次循环计算只执行一次。分别耗时 202ms 和 110ms，可见，提取复杂的重复操作是相当具有意义的。这个例子告诉我们，在循环体内，如果能够提取到循环体外的计算公式，最好提取出来，尽可能让程序少做重复的计算。

## 优化循环
当性能问题成为系统的主要矛盾时，可以尝试优化循环，例如减少循环次数，这样也许可以加快程序运行速度。
*清单 22. 减少循环次数*

	public class reduceLoop {
	public static void beforeTuning(){
	 long start = System.currentTimeMillis();
	 int[] array = new int[9999999];
	 for(int i=0;i<9999999;i++){
	 array[i] = i;
	 }
	 System.out.println(System.currentTimeMillis() - start);
	}
	
	public static void afterTuning(){
	 long start = System.currentTimeMillis();
	 int[] array = new int[9999999];
	 for(int i=0;i<9999999;i+=3){
	 array[i] = i;
	 array[i+1] = i+1;
	 array[i+2] = i+2;
	 }
	 System.out.println(System.currentTimeMillis() - start);
	}
	
	public static void main(String[] args){
	reduceLoop.beforeTuning();
	reduceLoop.afterTuning();
	}
	}
运行输出如清单 23 所示。
*清单 23. 运行结果*

	265
	31

这个例子可以看出，通过减少循环次数，耗时缩短为原来的 1/8。

## 布尔运算代替位运算
虽然位运算的速度远远高于算术运算，但是在条件判断时，使用位运算替代布尔运算确实是非常错误的选择。在条件判断时，Java 会对布尔运算做相当充分的优化。假设有表达式 a、b、c 进行布尔运算“a&&b&&c”，根据逻辑与的特点，只要在整个布尔表达式中有一项返回 false，整个表达式就返回 false，因此，当表达式 a 为 false 时，该表达式将立即返回 false，而不会再去计算表达式 b 和 c。若此时，表达式 a、b、c 需要消耗大量的系统资源，这种处理方式可以节省这些计算资源。同理，当计算表达式“a||b||c”时，只要 a、b 或 c，3 个表达式其中任意一个计算结果为 true 时，整体表达式立即返回 true，而不去计算剩余表达式。简单地说，在布尔表达式的计算中，只要表达式的值可以确定，就会立即返回，而跳过剩余子表达式的计算。若使用位运算 (按位与、按位或) 代替逻辑与和逻辑或，虽然位运算本身没有性能问题，但是位运算总是要将所有的子表达式全部计算完成后，再给出最终结果。因此，从这个角度看，使用位运算替代布尔运算会使系统进行很多无效计算。
*清单 24. 运算方式对比*

	public class OperationCompare {
	 public static void booleanOperate(){
	 long start = System.currentTimeMillis();
	 boolean a = false;
	 boolean b = true;
	 int c = 0;
	 //下面循环开始进行位运算，表达式里面的所有计算因子都会被用来计算
	 for(int i=0;i<1000000;i++){
	 if(a&b&"Test_123".contains("123")){
	 c = 1;
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 }
	 
	 public static void bitOperate(){
	 long start = System.currentTimeMillis();
	 boolean a = false;
	 boolean b = true;
	 int c = 0;
	 //下面循环开始进行布尔运算，只计算表达式 a 即可满足条件
	 for(int i=0;i<1000000;i++){
	 if(a&&b&&"Test_123".contains("123")){
	 c = 1;
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	 }
	 
	 public static void main(String[] args){
	 OperationCompare.booleanOperate();
	 OperationCompare.bitOperate();
	 }
	}
运行输出如清单 25 所示。
*清单 25. 运行结果*

	63
	0
实例显示布尔计算大大优于位运算，但是，这个结果不能说明位运算比逻辑运算慢，因为在所有的逻辑与运算中，都省略了表达式“"Test_123".contains("123")”的计算，而所有的位运算都没能省略这部分系统开销。
## 使用 arrayCopy()
数据复制是一项使用频率很高的功能，JDK 中提供了一个高效的 API 来实现它。System.arraycopy() 函数是 native 函数，通常 native 函数的性能要优于普通的函数，所以，仅处于性能考虑，在软件开发中，应尽可能调用 native 函数。ArrayList 和 Vector 大量使用了 System.arraycopy 来操作数据，特别是同一数组内元素的移动及不同数组之间元素的复制。arraycopy 的本质是让处理器利用一条指令处理一个数组中的多条记录，有点像汇编语言里面的串操作指令 (LODSB、LODSW、LODSB、STOSB、STOSW、STOSB)，只需指定头指针，然后开始循环即可，即执行一次指令，指针就后移一个位置，操作多少数据就循环多少次。如果在应用程序中需要进行数组复制，应该使用这个函数，而不是自己实现。具体应用如清单 26 所示。
*清单 26. 复制数据例子*

	public class arrayCopyTest {
	public static void arrayCopy(){
	int size = 10000000;
	 int[] array = new int[size];
	 int[] arraydestination = new int[size];
	 for(int i=0;i<array.length;i++){
	 array[i] = i;
	 }
	 long start = System.currentTimeMillis();
	 for(int j=0;j>1000;j++){
	 System.arraycopy(array, 0, arraydestination, 0, size);//使用 System 级别的本地 arraycopy 方式
	 }
	 System.out.println(System.currentTimeMillis() - start);
	}
	
	public static void arrayCopySelf(){
	int size = 10000000;
	 int[] array = new int[size];
	 int[] arraydestination = new int[size];
	 for(int i=0;i<array.length;i++){
	 array[i] = i;
	 }
	 long start = System.currentTimeMillis();
	 for(int i=0;i<1000;i++){
	 for(int j=0;j<size;j++){
	 arraydestination[j] = array[j];//自己实现的方式，采用数组的数据互换方式
	 }
	 }
	 System.out.println(System.currentTimeMillis() - start);
	}
	
	 public static void main(String[] args){
	 arrayCopyTest.arrayCopy();
	 arrayCopyTest.arrayCopySelf();
	 }
	}
输出如清单 27 所示。
*清单 27. 运行结果*

	0
	23166
上面的例子显示采用 arraycopy 方法执行复制会非常的快。原因就在于 arraycopy 属于本地方法，源代码如清单 28 所示。
*清单 28.arraycopy 方法*

	public static native void arraycopy(Object src, int srcPos, 
	Object dest, int destPos, 
	int length);
src - 源数组；srcPos - 源数组中的起始位置； dest - 目标数组；destPos - 目标数据中的起始位置；length - 要复制的数组元素的数量。清单 28 所示方法使用了 native 关键字，调用的为 C++编写的底层函数，可见其为 JDK 中的底层函数。

Reference:[http://www.ibm.com/developerworks/cn/java/j-lo-basic-types/index.html](http://www.ibm.com/developerworks/cn/java/j-lo-basic-types/index.html)