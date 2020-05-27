# Java中的自动装箱与拆箱
@(Java)[基础知识]
[Java中的自动装箱与拆箱 - 技术小黑屋](https://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/)
[自动拆箱&自动装箱以及String和基本数据类型封装类生成的对象是否相等](https://blog.csdn.net/u010126792/article/details/61616412)
[Integer缓存池（IntegerCache）及整型缓存池](https://blog.csdn.net/maihilton/article/details/80101497)

自动装箱和拆箱在 Java 中很常见，比如我们有一个方法，接受一个对象类型的参数，如果我们传递一个原始类型值，那么Java会自动讲这个原始类型值转换成与之对应的对象。最经典的一个场景就是当我们向 ArrayList 这样的容器中增加原始类型数据时或者是创建一个参数化的类。

赋值时：这是最常见的一种情况，在 Java 1.5 以前我们需要手动地进行转换才行，而现在所有的转换都是由编译器来完成。

对于基本类型（原始类型）：`boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, `double`来说，并不存在对象一说，所有生成的过程均为（以`int`为例）：
```plain
iconst_1 		// [-1， 5] 
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
System.out.println(System.identityHashCode(a)); // 739498517
System.out.println(System.identityHashCode(b)); // 739498517

int c = 128;
int d = 128;
System.out.println(System.identityHashCode(c)); // 125130493
System.out.println(System.identityHashCode(d)); // 914504136
```
对于引用类型（包装类型）：`Boolean`, `Character`, `Byte`,` Short`, `Integer`, `Long` 默认自动装箱池范围为小于等于 1 个`byte`（0 ~ 0xff, -128 ~ 127），当新建对象的值在该范围内时，程序会在自动装箱池（缓存池）`static final Object cache[]`中通过`intValue()`查找相同值的对象，如果有则直接返回该对象，否则初始化新的对象并存入自动装箱池。之后便不会再生成新的等于该值的对象，而是直接自动装箱池中的等于该值的对象。在其生成新的对象时，会有`private final int value`的成员变量一起生成，`intValue()`方法返回的就是这个变量。
 注意：浮点类型`Float`, `Double` 没有缓存。
```java
// 字面值大于0xff时
Integer a = new Integer(128);
Integer b = new Integer(128);
int c = a.intValue();
int d = b.intValue();
Integer e = 128;			// 将 int 通过 valueOf() 自动装箱成 Integer
int f = new Integer(128);	// 将 Integer 通过 intValue() 自动拆箱成 int

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
Integer b = new Integer(127);
int c = a.intValue();
int d = b.intValue();
Integer e = 127;			// 将 int 通过 valueOf() 自动装箱成 Integer
int f = new Integer(127);	// 将 Integer 通过 intValue() 自动拆箱成 int

System.out.println(System.identityHashCode(a)); // 739498517
System.out.println(System.identityHashCode(b)); // 125130493
System.out.println(System.identityHashCode(c)); // 914504136
System.out.println(System.identityHashCode(d)); // 914504136
System.out.println(System.identityHashCode(e)); // 914504136
System.out.println(System.identityHashCode(f));	// 914504136
```

对于`Integer`类型的变量，可以通过 JVM 参数`--XX:AutoBoxCacheMax=<size>`调整自动装箱池的范围为 [-size-1~size]，但是其他基本数据类型的自动装箱池范围不可变。

注意：自动装箱和自动拆箱不会影响函数的重载，使涉及重载的函数变为同一个函数。例如：
```java
public int add(Integer x) {
	return x;
}
public int add(int x) {
	return x;
}
```