# Java中的自动装箱与拆箱
@(Java)[基础知识]
[Java中的自动装箱与拆箱 - 技术小黑屋](https://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/)
自动装箱和拆箱在 Java 中很常见，比如我们有一个方法，接受一个对象类型的参数，如果我们传递一个原始类型值，那么Java会自动讲这个原始类型值转换成与之对应的对象。最经典的一个场景就是当我们向 ArrayList 这样的容器中增加原始类型数据时或者是创建一个参数化的类。

赋值时：这是最常见的一种情况，在 Java 1.5 以前我们需要手动地进行转换才行，而现在所有的转换都是由编译器来完成。

对于基本类型（原始类型）：`boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, `double`来说，字面值小于等于 1 个`byte`时（0 ~ 0xff, -128 ~ 127），JVM 会将此基本类型对象缓存入栈，之后便不会再生成新的等于该值的对象，而是直接引用栈中的等于该值的对象。
```java
int a = 127;
int b = 127;
System.out.println(System.identityHashCode(a)); // 739498517
System.out.println(System.identityHashCode(b)); // 739498517

int c = 128;
int d = 128;
System.out.println(System.identityHashCode(c)); // 739498517
System.out.println(System.identityHashCode(d)); // 125130493
```
对于引用类型（包装类型）：`Boolean`, `Character`, `Byte`,` Short`, `Integer`, `Long`, `Float`, `Double`。在其生成新的对象时，会有`private final int value;`的成员变量一起生成，该变量为基本类型，所以当字面值小于等于 1 个`byte`时（0 ~ 0xff, -128 ~ 127），JVM 的栈中也会缓存这个成员变量。
```java
// 字面值大于0xff时
Integer a = new Integer(128);
Integer b = new Integer(128);
int c = a.intValue();
int d = b.intValue();
Integer e = 128; // 将int类型自动装箱成Integer类型
int f = new Integer(128); // 将Integer类型自动拆箱成int类型

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
Integer e = 127; // 将127自动装箱成Integer类型
int f = new Integer(127); // 将Integer类型自动拆箱成int类型

System.out.println(System.identityHashCode(a)); // 739498517
System.out.println(System.identityHashCode(b)); // 125130493
System.out.println(System.identityHashCode(c)); // 914504136
System.out.println(System.identityHashCode(d)); // 914504136
System.out.println(System.identityHashCode(e)); // 914504136
System.out.println(System.identityHashCode(f)); // 914504136
```
