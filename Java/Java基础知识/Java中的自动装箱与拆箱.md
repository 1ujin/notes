#Java中的自动装箱与拆箱
@(Java)[基础知识]

---
- [引用](#引用)

- [基本数据类型](#基本数据类型)

- [字符串类型](#字符串类型)
	- [`String`, `StringBuilder`和`StringBuffer`](#string-stringbuilder和stringbuffer)

---
##[引用](#java中的自动装箱与拆箱)
[Java中的自动装箱与拆箱 - 技术小黑屋](https://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/)

[自动拆箱&自动装箱以及String和基本数据类型封装类生成的对象是否相等](https://blog.csdn.net/u010126792/article/details/61616412)

[Integer缓存池（IntegerCache）及整型缓存池](https://blog.csdn.net/maihilton/article/details/80101497)

[Java常量池理解与总结](https://www.jianshu.com/p/c7f47de2ee80)

[深入理解Java String类](https://blog.csdn.net/ifwinds/article/details/80849184)

[Java中几种常量池的区分](http://tangxman.github.io/2015/07/27/the-difference-of-java-string-pool/)

---
##[基本数据类型](#java中的自动装箱与拆箱)

自动装箱和拆箱在 Java 中很常见，比如我们有一个方法，接受一个对象类型的参数，如果我们传递一个原始类型值，那么 Java 会自动讲这个原始类型值转换成与之对应的对象。最经典的一个场景就是当我们向 ArrayList 这样的容器中增加原始类型数据时或者是创建一个参数化的类。

> 装箱（boxing）：int → Integer

> 拆箱（unboxing）：Integer → int

赋值时：这是最常见的一种情况，在 Java 1.5 以前我们需要手动地进行转换才行，而现在所有的转换都是由编译器来完成。

对于基本类型（原始类型）：`boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, `double`来说，不是对象类型所以不会生成对象，所有生成的过程均为（以`int`为例）：

```plain
iconst_1        // [-1, 5] 
istore_1 
bipush        6 // 小于-1或大于5时 
istore_2 
```

注意：`System.identityHashCode()`并不能反映出`int`类型的真实地址，因为此方法的声明为：

```java
public static native int identityHashCode(Object x);
```

其形参为包装类型。当基本类型的变量作为实参传入时，会自动装箱为包装类型，如`int`装箱为`Integer`，此时返回的是装箱后的对象地址。

```java
int a = 127;
int b = 127;
System.out.println(System.identityHashCode(a));   // 739498517
System.out.println(System.identityHashCode(b));   // 739498517
System.out.println(System.identityHashCode(127)); // 739498517

int c = 128;
int d = 128;
System.out.println(System.identityHashCode(c));   // 125130493
System.out.println(System.identityHashCode(d));   // 914504136
System.out.println(System.identityHashCode(128)); // 166239592
```

对于引用类型（包装类型）： `Byte`,` Short`, `Integer`, `Long` 默认自动装箱池存有0 ~ 0xff (-128 ~ 127)，即小于等于 1 个`byte`的值，而`Character`的默认自动装箱池则存有0 ~ 0x7f (0 ~ 127)的值。[当新建对象的值在该范围内时，程序会在自动装箱池（缓存池）`static final Object cache[]`中通过`intValue()`查找相同值的对象，如果有则直接返回该对象。之后便不会再生成新的等于该值的对象，而是直接自动装箱池中的等于该值的对象。](https://blog.csdn.net/u010126792/article/details/61616412)在生成新的对象时，会有`private final int value`的成员变量一起生成，`intValue()`方法返回的就是这个变量。
注意：浮点类型`Float`, `Double` 没有缓存。`Boolean`没有（不需要）缓存数组，存有`true`和`false`。

```java
// 字面值大于0xff时
Integer a = new Integer(128);
Integer b = new Integer(128); // 新建对象，无论如何地址都不同
int c = a.intValue();
int d = b.intValue();
Integer e = 128;              // 将 int 通过 valueOf() 自动装箱成 Integer
int f = new Integer(128);     // 将 Integer 通过 intValue() 自动拆箱成 int

System.out.println(System.identityHashCode(a)); // 739498517
System.out.println(System.identityHashCode(b)); // 125130493
System.out.println(System.identityHashCode(c)); // 914504136
System.out.println(System.identityHashCode(d)); // 166239592
System.out.println(System.identityHashCode(e)); // 991505714
System.out.println(System.identityHashCode(f)); // 385242642
```

```java
// 字面值小于等于0xff时
Integer a = new Integer(127);
Integer b = new Integer(127); // 新建对象，无论如何地址都不同
int c = a.intValue();
int d = b.intValue();
Integer e = 127;              // 将 int 通过 valueOf() 自动装箱成 Integer
int f = new Integer(127);     // 将 Integer 通过 intValue() 自动拆箱成 int

System.out.println(System.identityHashCode(a)); // 739498517
System.out.println(System.identityHashCode(b)); // 125130493
System.out.println(System.identityHashCode(c)); // 914504136
System.out.println(System.identityHashCode(d)); // 914504136
System.out.println(System.identityHashCode(e)); // 914504136
System.out.println(System.identityHashCode(f));	// 914504136
```

可以利用**反射**获取自动装箱池子（缓存数组），以`Integer`类型为例：

```java
Class<?> cls = Class.forName("java.lang.Integer$IntegerCache");
Field field = cls.getDeclaredField("cache");
field.setAccessible(true);
Integer[] cache = (Integer[]) field.get(null);
System.out.println(Arrays.toString(cache));	// [-128, -127, -126, ..., 127]
```

从 **Java 9** 开始通过构造器生成对象的方式`Integer i = new Integer(127)`已经过时，此方法**必定**会在堆中生成新的对象，相同值生成的两个对象地址不同。现在采用静态方法`Integer i = Integer.valueOf(127)`，先在自动装箱池中**查找**有无已经生成并且值相同的对象，有则直接返回，否则才通过构造器生成。

如果我们更改缓存数组中的某个对象，就会影响`Integer`对象的生成和自动装箱：

```java
System.out.println(1 + ": " + System.identityHashCode(1));       // 1: 1013423070
System.out.println(-128 + ": " + System.identityHashCode(-128)); // -128: 1552787810
cache[0] = Integer.valueOf(1);                                   // 将 -128 改为 1
System.out.println(Arrays.toString(cache));                      // [1, -127, -126, ..., 127]
Integer a = Integer.valueOf(-128);
int b = Integer.valueOf(-128);
int c = -128;
System.out.println(a + ": " + System.identityHashCode(a));       // 1: 1013423070
System.out.println(b + ": " + System.identityHashCode(b));       // 1: 1013423070
System.out.println(c + ": " + System.identityHashCode(c));       // -128: 1013423070
```

将 -128 改为 1 之后，通过`Integer.valueOf(-128)`方法生成对象时，本应在自动装箱池中找到 -128 并返回，现在在相同位置找到并返回的却是 1。而获取基本类型的 c 的地址时，由于`System.identityHashCode()`会自动装箱传入的实参，其地址也变为了 1 的地址。由此可以验证上文中的论述。

对于`Integer`类型的变量，可以通过 JVM 参数`--XX:AutoBoxCacheMax=<size>`调整自动装箱池的范围为 [-size-1, size]，但是其他基本数据类型的自动装箱池范围不可变。

注意：自动装箱和自动拆箱不会影响函数的**重载**，使涉及重载的函数变为同一个函数。例如：

```java
public int add(Integer x) {
    return x;
}
public int add(int x) {
    return x;
}
```

---
##[字符串类型](#java中的自动装箱与拆箱)
`String`是`final`修饰的类型，不可继承（区别于不可变类）：

> 当你将final用于类身上时，一个final类是无法被任何人继承的，那也就意味着此类在一个继承树中是一个叶子类，并且此类的设计已被认为很“完美”而不需要进行修改或扩展。对于final类中的成员，你可以定义其为final，也可以不是final。而对于方法，由于所属类为final的关系，自然也就成了final型的。

`String`与`Integer`等基本类型的包装类一样是[不可变类（Immutable Class）](https://www.jianshu.com/p/48b011688edc)，成员变量被`final`修饰过不可更改。作为参数传入一个方法后，里面的修改（会创建一个新对象）不影响外面。

> 不可变类只是其实例不能被修改的类，每个实例中包含的所有信息都必须在创建该实例的时候就提供，并在对象的整个生命周期内固定不变。
>
> 要创建不可变类，只要遵循下面几条规则：
> 1. 不要提供任何会修改对象状态的方法。
> 2. 保证类不会被拓展（一般声明为final即可）。
> 3. 使所有的域都是 private final的。
> 4. 确保对于任何可变组件的互斥访问（可以理解如果中存在可变对象的域，得确保客户端无法获得其引用，并且不要使用客户端提供的对象来初始化这样的域）。

`String`的底层就是一个被`final`修饰过`char`类型的数组：
```java
private final byte[] value;
```
该数组是**不可变**的，所以适合作为`HashMap`的`key`。`HashMap`内部实现是通过`key`的**哈希值**来确定`value`的存储位置，因为字符串是不可变的，所以当创建字符串时，它的哈希值被缓存下来，不需要再次计算，所以相比于其他对象更快。

但是我们可以通过**反射**的方式获取并修改其内容：

```java
String s = "abc";
System.out.println(s + " @ " + System.identityHashCode(s));     // abc @ 380936215
System.out.println(Arrays.toString(s.getBytes()) + " @ "
        + System.identityHashCode(s.getBytes()));               // [97, 98, 99] @ 142638629
Field valueField = String.class.getDeclaredField("value");
valueField.setAccessible(true);
byte[] value = (byte[]) valueField.get(s);                      // 无法直接转为 char[]
System.out.println(Arrays.toString(value) + " @ "
        + System.identityHashCode((byte[]) valueField.get(s))); // [97, 98, 99] @ 428746855
value[0] = (byte) 'b';                                          // 将第一个字符改为 'b'
System.out.println(s + " @ " + System.identityHashCode(s));     // bbc @ 380936215
```

可以看出此处`s.getBytes()`返回的对象是`Arrays.copyOf()`方法**浅复制**的，地址不同，所以不是`value`。

JDK 源码中的字符串构造方法：
>Initializes a newly created `String` object so that it represents  the same sequence of characters as the argument; in other words, the newly created string is a copy of the argument string. Unless an  explicit copy of `original` is needed, use of this constructor is unnecessary since Strings are immutable.

```java
@HotSpotIntrinsicCandidate // 被注解的方法在 HotSpot 中都有一套高效的实现，该高效实现基于 CPU 指令，运行时，HotSpot 维护的高效实现会替代 JDK 的源码实现，从而获得更高的效率
public String(String original) {
    this.value = original.value;
    this.coder = original.coder;
    this.hash = original.hash;
}
```

通过形如 s1, s2, s3, s4 采用引号包含文本方式创建的`String`对象，在**第一次**创建时会被加入到**字符串常量池**中，之后若再次创建，就在字符串常量池中拿对象，其地址都相同。而采用`new String()`的方式会产生新的对象，地址各不相同。如果构造器传入的参数文本为第一次创建，则会在**类加载**时先执行前一种创建方法，将字符串加入到字符串常量池中。然后从字符串常量池中取出字符串对象的地址赋给引用，从而生成新的对象。[字符串常量池在 JDK 7 之前存在运行时常量池（Prem）中，在 JDK 7 已经将其转移到堆（Java Heap）中。](https://www.javatt.com/p/47643)

> 在 HotSpot VM 中字符串常量池是通过一个`StringTable`类实现的，它是一个 Hash 表，默认值大小长度是1009；这个`StringTable`在每个 HotSpot VM 的实例中**只有一份**，被所有的类共享；字符串常量由一个一个字符组成，放在了`StringTable`上。要注意的是，如果放进字符串常量池的字符串非常多，就会造成 Hash 冲突严重，从而导致链表很长，而链表长了后直接会造成的影响就是当调用`intern()`时性能会大幅下降（因为要一个一个找）。

> 在 JDK 6 及之前版本，字符串常量池中**所引用的**字符串实例是放在 Perm Gen 区（方法区）中的，`StringTable`的长度是固定的1009；在 JDK 7 版本中，这些实例被移到了堆中。而`StringTable`自身则一直在 Native Memory 中。 `StringTable`的长度可以通过`--XX:StringTableSize=<size>`参数指定。至于 JDK 7 为什么把常量池中的引用移动到堆中，原因可能是由于方法区的内存空间太小且不方便扩展，而堆的内存空间比较大且扩展方便。

>字符串常量池，全局字符串池（String Pool, String Literal Pool）：全局字符串池里的内容是在类加载完成，经过验证，**准备阶段之后在堆中生成字符串对象实例，然后将该字符串对象实例的引用值存到 String Pool 中（记住：String Pool 中存的是引用值而不是具体的实例对象，具体的实例对象是在堆中开辟的一块空间存放的）。** 在 HotSpot VM 里实现的 String Pool 功能的是一个`StringTable`类，它是一个哈希表，里面存的是驻留字符串(也就是我们常说的用双引号括起来的)的引用（而不是驻留字符串实例本身），也就是说在堆中的某些字符串实例被这个`StringTable`引用之后就等同被赋予了”驻留字符串”的身份。这个`StringTable`在每个 HotSpot VM 的实例只有一份，被所有的类共享。 

字符串的拘留：`String`对象的`intern()`方法会查找在字符串常量池中是否存在一份内容相等的字符串，如果有则返回该字符串的引用，如果没有则添加自己的字符串进入字符串常量池。直接使用双引号声明出来的`String`对象会直接存储在字符串常量池中，如果不是用双引号声明的`String`对象，可以使用`intern()`方法。使用`intern()`方法会增加少量时间，但会节省空间，从而也节省了占用空间引起的垃圾回收的时间。

```java
String s1 = "ab", s2 = "cd";                              // 创建"ab"和"cd"对象分别放入字符串常量池
String s3 = "abcd";                                       // 创建"abcd"对象并放入字符串常量池
String s4 = "ab" + "cd";                                  // 直接从字符串常量池中取出
String s5 = s1 + s2;                                      // new 的方式，创建了1个对象，不放入字符串常量池
String s6 = s1 + s2;                                      // new 的方式，创建了1个对象，不放入字符串常量池
String s7 = "ab" + s2;                                    // new 的方式，创建了1个对象，不放入字符串常量池
String s8 = s1 + "cd";                                    // new 的方式，创建了1个对象，不放入字符串常量池
String s9 = new String("abcd");                           // new 的方式，创建了1个对象，字符串常量池中已有"abcd"
String s10 = new String("qwer");                          // new 的方式，创建了2个对象，并且"qwer"放入字符串常量池
System.out.println(System.identityHashCode(s3));          // 317983781
System.out.println(System.identityHashCode(s4));          // 317983781
System.out.println(System.identityHashCode(s5));          // 987405879
System.out.println(System.identityHashCode(s4.intern())); // 317983781
System.out.println(System.identityHashCode(s5.intern())); // 317983781
System.out.println(System.identityHashCode(s6));          // 1555845260
System.out.println(System.identityHashCode(s7));          // 874088044
System.out.println(System.identityHashCode(s8));          // 104739310
System.out.println(System.identityHashCode(s9));          // 1761291320
System.out.println(System.identityHashCode(s9.intern())); // 317983781
```

> 在Java中，**唯一被重载**的运算符就是用于String的“+”与“+=”。除此之外，Java不允许程序员重载其他的运算符。

如果创建字符串时，“+”两边全部是字面量文本，则会被编译器优化为一个字面量文本，生成的结果会被放入字符串常量池。如果含有字符串引用则会被编译器自动引入`StringBuilder`类，创建一个`StringBuilder`对象，并调用`append()`方法，最后调用`toString()`生成结果，从而避免中间对象的性能损耗，字符串并**不会**放入字符串常量池。后者虽然没有显式的用 new 的方式创建对象，但依然会被编译器编译成 new 的方式。所以遇到多次循环时，循环体中需要尽量避免隐式或者显式创建`StringBuilder`对象。参考[字符串的拼接](https://www.jianshu.com/p/88aa19fc21c6)。

形如 s5, s6, s7, s8 的不放入字符串常量池中的情况：

```java
String s1 = new String("ABC") + new String("DEF");        // 生成的"ABCDEF"不会被放入字符串常量池
System.out.println(System.identityHashCode(s1));          // 987405879
System.out.println(System.identityHashCode(s1.intern())); // 987405879 添加进字符串常量池中
String s2 = "ABCDEF";                                     // 直接从字符串常量池中取出
System.out.println(System.identityHashCode(s2));          // 987405879
```

```java
String s3 = new String("abc") + new String("def");        // 生成的"abcdef"不会被放入字符串常量池
System.out.println(System.identityHashCode(s3));          // 1555845260
String s4 = "abcdef";                                     // 生成的"abcdef"并放入字符串常量池
System.out.println(System.identityHashCode(s4));          // 874088044
System.out.println(System.identityHashCode(s3.intern())); // 874088044 尝试添加，但字符串常量池中已有
```

上述例子的字符串对象都是在方法内也就是运行时生成的动态对象，下面是关于`static final`关键字（静态常量）的特例：

```java
public static final String s1 = "aa";
public static final String s2 = "bb";
public static void main(String[] args) {
    System.out.println(System.identityHashCode(s1 + s2)); // 104739310
    System.out.println(System.identityHashCode("aabb"));  // 104739310
}
```

s1和s2都是已经初始化的常量，值是**固定**的，因此`s1 + s2`的值也是固定的，它在类被**编译时**就已经确定了，也就是说`s1 + s2`等同于`"aa" + "bb"`。

```java
public static final String s1;
public static final String s2;
static {
    s1 = "aa";
    s2 = "bb";
}
public static void main(String[] args) {
    System.out.println(System.identityHashCode(s1 + s2)); // 104739310
    System.out.println(System.identityHashCode("aabb"));  // 1761291320
}
```

s1和s2虽然被定义为常量，但是它们都没有马上被赋值。在运算出`s1 + s2`的值之前，他们何时被赋值，以及被赋予什么样的值，都是个变数。因此s1和s2在被赋值之前，性质类似于一个变量。那么`s1 + s2`就不能在编译期被确定，而只能在运行时被创建。

###[`String`, `StringBuilder`和`StringBuffer`](#字符串类型)
![@继承结构|center](https://img-blog.csdn.net/20180703182143144?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lmd2luZHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
- `String`：不可变类，用“+”连接时较慢（详见拘留池）
- `StringBuilder`：可变类，非线程安全，较快，比`StringBuffer`快10%~15%
- `StringBuffer`：可变类，线程安全（同步锁）