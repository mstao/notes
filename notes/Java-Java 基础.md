[TOC]

# 基础语法

**编程语言**

Q: 语言分类

编译时：   Java, GO

运行时：    Python, Perl, JS, Ruby

​       |-运行时知道每个变量的具体类型

 

运行/编译：      

​       编译为机器代码运行：   

编译为中间代码：  

解释执行：   Python, Perl, JS

​      |-解释器每看到一行代码便运行一行代码

​              |-解释器小

 

编程范式：

​       面向过程：

​       面向对象：  Scala

​       *函数式：*    Haskell，  Erlang

​              |-具体



⾯向对象和⾯向过程的区别

⾯向过程 ：⾯向过程性能⽐⾯向对象⾼。 因为类调⽤时需要实例化，开销⽐较⼤，⽐较消耗资 源，所以当性能是最重要的考量因素的时候，⽐如单⽚机、嵌⼊式开发、Linux/Unix 等⼀般采 ⽤⾯向过程开发。但是，⾯向过程没有⾯向对象易维护、易复⽤、易扩展。 ⾯向对象 ：⾯向对象易维护、易复⽤、易扩展。 因为⾯向对象有封装、继承、多态性的特性， 所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，⾯向对象性能⽐⾯向过 程低。



## 语法细节

@Q: final 和 finally 的区别

final 代表不可变，是 C++ 中 Const 的弱化版本；

finally 配合 try{} catch{} 实现的用于处理异常的关键字；



@Q: `&` 和 `&&` 的区别。
(1) 共同点： 都可以用作逻辑与的运算符，表示逻辑与（and），当运算符两边的表达式的结果都为true时，整个运算结果才为true，否则，只要有一方为false，则结果为false。

(2) 不同点：

- &&还具有短路的功能，即如果第一个表达式为false，则不再计算第二个表达式，例如，对于`if(str != null&& !str.equals(“”))`表达式，当str为null时，后面的表达式不会执行，所以不会出现 NullPointerException 如果将&&改为&，则会抛出 NullPointerException 异常。If(x==33 &++y>0) y会增长，If(x==33 && ++y>0)不会增长；

- &还可以用作位运算符，当 & 操作符两边的表达式不是 boolean 类型时，& 表示按位与操作我们通常使用 0x0f 来与一个整数进行&运算，来获取该整数的最低4个bit位，例如，0x31 & 0x0f的结果为0x01。
  备注：这道题先说两者的共同点，再说出&&和&的特殊之处，并列举一些经典的例子来表明自己理解透彻深入、实际经验丰富。





@Q: switch语句能否作用在byte上，能否作用在long上，能否作用在String上?
在switch（expr1）中，expr1只能是一个整数表达式或者枚举常量，整数表达式可以是int基本类型或Integer包装类型，由于，byte,short,char都可以隐含转换为int，所以，这些类型以及这些类型的包装类型也是可以的。显然，long和String类型都不符合switch的语法规定，并且不能被隐式转换成int类型，所以，它们不能作用于swtich语句中。

// TODO String, Long 。。。





@Q: short s1 = 1; s1 = s1 + 1;有什么错? short s1 = 1; s1 += 1;有什么错?
对于short s1 = 1; s1 = s1 + 1;由于s1+1运算时会自动提升表达式的类型，所以结果是int型，再赋值给short类型s1时，编译器将报告需要强制转换类型的错误。
对于short s1 = 1; s1 += 1;由于 +=是java语言规定的运算符，java编译器会对它进行特殊处理，因此可以正确编译。





@Q: 请设计一个一百亿的计算器
首先要明白这道题目的考查点是什么，一是大家首先要对计算机原理的底层细节要清楚、要知道加减法的位运算原理和知道计算机中的算术运算会发生越界的情况，二是要具备一定的面向对象的设计思想。

计算机中的算术运算是会发生越界情况的，两个数值的运算结果不能超过计算机中的该类型的数值范

自己设计一个类可以用于表示很大的整数，并且提供了与另外一个整数进行加减乘除的功能，大概功能如下：
（1）这个类内部有两个成员变量，一个表示符号，另一个用字节数组表示数值的二进制数
（2）有一个构造方法，把一个包含有多位数值的字符串转换到内部的符号和字节数组中
（3）提供加减乘除的功能

他最重要的还是考查你的能力，所以，你不要因为自己无法写出完整的最终结果就放弃答这道题，你要做的就是你比别人写得多，证明你比别人强，你有这方面的思想意识就可以了，毕竟别人可能连题目的意思都看不懂，什么都没写，你要敢于答这道题，即使只答了一部分，那也与那些什么都不懂的人区别出来，拉开了距离，算是矮子中的高个，机会当然就属于你了。另外，答案中的框架代码也很重要，体现了一些面向对象设计的功底，特别是其中的方法命名很专业，用的英文单词很精准，这也是能力、经验、专业性、英语水平等多个方面的体现，会给人留下很好的印象，在编程能力和其他方面条件差不多的情况下，英语好除了可以使你获得更多机会外，薪水可以高出一千元。

参考 java.math.BigInteger

```java
class BigInteger extends Number implements Comparable<BigInteger> {
    final int signum;
    final int[] mag;
}
```





@Q: 静态变量和实例变量的区别？

- 在语法定义上的区别：静态变量前要加static关键字，而实例变量前则不加。
- 在程序运行时的区别：实例变量属于某个对象的属性，必须创建了实例对象，其中的实例变量才会被分配空间，才能使用这个实例变量。静态变量不属于某个实例对象，而是属于类，所以也称为类变量，只要程序加载了类的字节码，不用创建任何实例对象，静态变量就会被分配空间，静态变量就可以被使用了。总之，实例变量必须创建对象后才可以通过这个对象来使用，静态变量则可以直接使用类名来引用。



三目运算符：

在与一般的算数表达式一起使用的时候需要注意其优先级

```java
int val1 = root.val + (root.left != null ? rob(root.left.left) + rob(root.left.right) : 0)
        + (root.right != null ? rob(root.right.left) + rob(root.right.right) : 0);  
int val2 = rob(root.left) + rob(root.right);
```





## 数据类型

- 8 大基本类型： byte、int、short、int、float、double、char、boolean；

- 对于 String、Enum、Array 在 Java 中对其做特殊的支持；

- Collect、Map 等高级的数据结构；



**浮点数**

（1） 精度

使用二进制来表示小数部分，因而存在误差问题；

一般情况下需要精确运算时，将其转换成 int 进行处理，或者使用 BigDecimal；

(+/-)1.xxx * 2^y

符号位 | 指数部分 | 基数部分

64 位的 double 精度 10^15

（2） 浮点数的比较





**值类型与引用类型**

引用类型： a == b 判断是否为同一个 Object

用 a.equals(b), 或 Objects.equals(a, b) 判断是否相等；





## 装箱和拆箱

**缓冲池**

通过字面量创建的，底层调用 Integer.valueOf() 来实现，而其维护了一个 缓冲池；





**比较**

Integer 与 int 类型比较时自动拆箱比较；

```java
class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];
}
```



BigDecimal 数值类型：

用于数值的精确计算，配合数据库中所保存的位数进行控制





## 内部类

内部类分为静态内部类，成员内部类，局部内部类，匿名内部类四种。

1. 静态内部类可以访问外部类所有的静态变量和方法，即使是private的也一样。 
2. 静态内部类和一般类一致，可以定义静态变量、方法，构造方法等。 
3. 其它类使用静态内部类需要使用 `外部类.静态内部类` 方式，如下所示
```java
Out.Inner inner = new Out.Inner();
inner.print();
```
Java集合类HashMap内部就有一个静态内部类Entry。Entry是HashMap存放元素的抽象，HashMap内部维护Entry数组用了存放元素，但是Entry对使用者是透明的。像这种和外部类关系密切的，且不依赖外部类实例的，都可以使用静态内部类。

4. 成员内部类
定义在类内部的非静态类，就是成员内部类。成员内部类 <u>不能定义静态方法和变量（final修饰的除外）</u> 。这是因为成员内部类是非静态的，类初始化的时候先初始化静态成员，如果允许成员内部类定义静态变量，那么成员内部类的静态变量初始化顺序是有歧义的。

5、定义在方法中的类，就是局部内部类。如果一个类只在某个方法中使用，则可以考虑使用局部类。

6、匿名内部类（要继承一个父类或者实现一个接口、直接使用new来生成一个对象的引用） 匿名内部类我们必须要继承一个父类或者实现一个接口，当然也仅能只继承一个父类或者实现一个接口。同时它也没有class关键字，这是因为匿名内部类是直接使用new来生成一个对象的引用。在 JDK8 中常用。





@Q: 内部类可以引用它的包含类的成员吗？有没有什么限制？
完全可以。如果不是静态内部类，那没有什么限制！

如果你把静态嵌套类当作内部类的一种特例，那在这种情况下不可以访问外部类的普通成员变量，而只能访问外部类中的静态成员





@Q: Anonymous Inner Class (匿名内部类)是否可以extends(继承)其它类，是否可以implements(实现)interface(接口)?
可以继承其他类或实现其他接口。不仅是可以，而是必须!





@Q: super.getClass()方法调用

在test方法中，直接调用getClass().getName()方法，返回的是Test类名
由于getClass()在Object类中定义成了final，子类不能覆盖该方法，所以，在test方法中调用getClass().getName()方法，其实就是在调用从父类继承的getClass()方法，等效于调用 `super.getClass().getName()` 方法，所以，super.getClass().getName()方法返回的也应该是Test。
如果想得到父类的名称，应该用如下代码：
getClass().getSuperClass().getName();

```java
public class Test extends Date{
    public static void main(String[] args) {
    	new Test().test();
    }
    public void test(){
    	System.out.println(super.getClass().getName());
    }
}
```





## 克隆

原型模式的使用，直接从内存中复用原有的数据，构造的速度相较于通过 constructor 更快。



复制：

（1） 直接赋值

实际上复制的是引用，也就是说a1和a2指向的是同一个对象。因此，当a1变化的时候，a2里面的成员变量也会跟着变化。

```java
A a1 = a2;
```

（2） 浅复制

复制引用但不复制引用的对象。如果字段是值类型的，那么对该字段执行复制；<u>如果该字段是引用类型的话，则复制引用但不复制引用的对象</u>。因此，原始对象及其副本引用同一个对象。

（3） 深复制

深拷贝不仅复制对象本身，而且复制对象包含的引用指向的所有对象。

实现深拷贝： 先进行整体的浅拷贝，之后对特别的对象进行深拷贝即可。
```java
class Student implements Cloneable {
    String name;
    int age;
    Professor p;
    Student(String name, int age, Professor p) {
        this.name = name;
        this.age = age;
        this.p = p;
    }
    public Object clone() {
    	Student o = null;
      try {
          o = (Student) super.clone();
      } catch (CloneNotSupportedException e) {
          System.out.println(e.toString());
      }
      o.p = (Professor) p.clone();  // deep clone
      return o;
    }
}
```





## String

String 的几种比较：

- 不可变：  final 修饰，不可变

- 安全性：URL、文件路径中使用，减少安全隐患。

- 效率： 缓存 hash 值，无需计算得到。

- 常量池： 在 JVM 中对 String 单独存储。

- 字符拼接的底层实现： 





**字符格式化**

- 通过 C 语言风格的格式化：%d，%f，%s ...

- 通过 {0} 参数控制的方式进行格式化：

- 通过 {} 参数控制格式化方式：同 Java 的日志框架中定义使用的方式控制

- 命名的格式化：类似 EL、JSX 等模板表达式， 一般有默认值、格式化日期、循环等高级特性

```java
System.out.printf("数字: %d, 浮点数: %f，字符串: %s", 2, 2.3, "string");
MessageFormat.format("xx{0}xx{1}", "xx", 2);
log.info("User name: [{}], access: [{}]", name, url);
${}  {{tt}}  {uuid}
```





@Q: String s = new String("xyz");创建了几个String Object?二者之间有什么区别？
两个或一个，”xyz”对应一个对象，这个对象放在字符串常量缓冲区，常量”xyz”不管出现多少遍，都是缓冲区中的那一个。New String每写一遍，就创建一个新的对象，它一句那个常量”xyz”对象的内容来创建出一个新String对象。如果以前就用过’xyz’，这句代表就不会创建”xyz”自己了，直接从缓冲区拿。





**底层结构**

注： JDK9 改为 byte[] 配合编码进行控制，减少不必要的空间损耗；

```java
final char value[];
int hash;
```





**特性-常量池**

JDK7 在 Method Area 的 PermGen，JDK8 转移到堆中 Metaspace；

（1） String.intern() 

native 方法, JDK8中常量池保存的可能是在堆中String对象的引用

```java
public native String intern();
```



JDK6 与 JDK6+ 不同:
 JDK6:
   ① 若 String Constant Pool 先前已创建出该字符串对象, 返回池中该字符串的引用.
   ② 池中无改String对象, 将此String 添加到池中, 并返回该String的引用

 JDK6+:
   ① 池中先前已创建该String, 返回池中该String 的引用
   ② 若该String存在于Java Heap 中, 将堆中对该 String 的引用添加到池中, 并返回该引用
     若Heap 中不存在, 在池中创建该String, 并返回其引用

（2） 字符串压缩

使得相同的字符串执行同一个对象,     G1 GC 中设置参数



**其他特性**

（1） 编译时优化

String 类型的字符串, 在编译时可以确定是一个字符串常量，编译完成后自动拼接成一个常量;   此时 String 的速度比 StringBuffer, StringBuilder 性能高.

（2）作为 switch 中 case 使用

JDK7 后可以作为 case 的条件判断；



**与 StringBuilder, StringBuffer 的区别**

(1) 可变性： String 不可变，StringBuilder 可变，StringBuffer 可变；

(2) 线程安全： String 线程安全，StringBuilder 线程不安全，StringBuffer 线程安全；

(3) 执行效率上： 在字符串拼接上，一般 String < StringBuffer < StringBuilder，在字面量时 String 可进行编译时优化从而效率最高；

(4) 作为 Map 的 Key： StringBuilder，StringBuffer 不可作为散列的键，未重写 hashCode 与 equals 方法；

(5) 使用场景上： String 适用于少量字符串操作，  StringBuilder 适用于单线程下在字符缓冲区下进行大量操作的情况， StringBuffer使用多线程下在字符缓冲区进行大量操作的情况；



StringBuilder

(1) 结构

```java
final class StringBuilder                       
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
```

(2) 内部数据保存：

通过维护一个足够大的数组实现

```java
// AbstractStringBuilder
char[] value;
int count;
```

(3) hashCode | equals

未重写





StringBuffer

(1) 底层结构

```java
// StringBuffer
transient char[] toStringCache;
```

(2) 安全性

```java
synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```





# 面向对象

**重载和重写**

Q: 重载和重写的区别?

重载是同一个类中，方法名称相同， 但是参数或个数不同。与返回值没有关系。是多态的静态体现。
重写是在多个类中， 产生继承关系。父类与子类的方法方法必须相同。



重载： 参数的个数、类型、顺序不一样即构成重载，返回值不能够作为判断标准；

重写： 子类重写父类的方法，执行的是子类的方法，对于 private 方法，子类不可见，若子类与父类一致时，将会被视为两个不同的方法；



重写的规则

- 两同： 方法名相同、参数列表相同

- 一小： 抛出的异常比父类少、或抛出父类异常的子异常；

- 一大： 子类的覆盖的方法的访问修饰符大于等于父类的访问修饰符，被覆盖的方法不能为private，否则在其子类中只是新定义了一个方法，并没有对其进行覆盖，相当于子类中增加了一个全新的方法。



@Q: 构造器Constructor是否可被override?
构造器 Constructor 不能被继承，因此不能重写 Override，但可以被重载 Overload。



@Q: 面向对象的特征有哪些方面

面向对象三大特性： 封装、继承、多态

封装： 保证软件部件具有优良的模块性的基础，封装的目标就是要实现软件部件的“高内聚、抽象





@Q: java中实现多态的机制是什么？

靠的是父类或接口定义的引用变量可以指向子类或具体实现类的实例对象，而程序调用的方法在运行期才动态绑定，就是引用变量所指向的具体实例对象的方法，也就是内存里正在运行的那个对象的方法，而不是引用变量的类型中定义的方法。





@Q: abstract的method是否可同时是static,是否可同时是native，是否可同时是synchronized?

1） abstract的method是否可同时是static,是否可同时是native，是否可同时是synchronized?

abstract 的 method 不可以是 static 的，因为抽象的方法是要被子类实现的，而static与子类不相关。

2） native方法表示该方法要用另外一种依赖平台的编程语言实现的，不存在着被子类实现的问题，所以，它也不能是抽象的，不能与abstract混用。例如，FileOutputSteam 类要硬件打交道，底层的实现用的是操作系统相关的api实现，例如，在windows用c语言实现的，所以，查看jdk的源代码，可以发现FileOutputStream的open方法的定义如下：

```java
private native void open(Stringname) throws FileNotFoundException;
```

如果我们要用java调用别人写的c语言函数，我们是无法直接调用的，我们需要按照java的要求写一个c语言的函数，又我们的这个c语言函数去调用别人的c语言函数。由于我们的c语言函数是按java的要求来写的，我们这个c语言函数就可以与java对接上，java那边的对接方式就是定义出与我们这个c函数相对应的方法，java中对应的方法不需要写具体的代码，但需要在前面声明native。

3） 关于synchronized与abstract合用的问题，我觉得也不行，因为在我几年的学习和开发中，从来没见到过这种情况，并且我觉得synchronized应该是作用在一个具体的方法上才有意义。而且，方法上的synchronized同步所使用的同步锁对象是this，而抽象方法上无法确定this是什么。







## 类与对象

**构成**

成员变量 -- 对象的状态

成员函数 -- 对象的行为

静态变量 -- 

静态函数



**当前对象的引用**

this、self(python)



**静态变量 | 静态函数**

没有  this 引用，静态变量全局唯一一份

普通函数引用变量、函数                            OK

对象上引用静态变量、函数                        编译器警告

静态函数引用普通成员变量、函数             编译错误





## 类的特殊函数

**静态块**

类的加载顺序

静态类被初始化的时机

// todo 补充 JVM 中初始化类的几个时机

main 方法对应的类

静态方法、静态属性第一次被调用的时候





**构造函数**

① 可以在内部相互调用 this，对构造函数的重载减少代码量

② 通常提供 无参、全参 构造器，作为阿里开发的规范?，可以通过 Lombok 进行快速的实现

③ 对传入的参数进行校验，维护传入的数据进行校验，进行数据的维护

pageBean，数据规整

Node(val)，顺带初始化

④ 适配器方式实现创造

⑤ 拷贝构造函数





Q: 创建对象的方式
(1) new 关键字创建；
(2) 反射创建；
(3) 序列化创建；
(4) 克隆创建；



### Object 中的函数

**equals**

 重写equals 不重写hashCode:
 存储散列集合时, origin.equals(newObj) , 未重写 hashCode ，
 在集合中将会存储两个值相同的对象，从而导致混摇.



实现了等价关系 ( equivalence relation )：

① 自反性 ( reflexive )，    x.equals(x) = true，x not null

② 对称性 (symmetric) ， x.equals(y)=true             ⇒ y.equals(x)， x not null

③ 传递性 (transitive )，   x.equals(y), y.equals(z)  ⇒ x.equals(z)， x,y,z not null

④ 一致性 (consistent )，equals 比较操作在对象中所用的信息没有被修改，多次调用 x.equals(y) 就会一致得返回 true OR false

⑤ x.equals(null) = false, x not null



**hashCode**

通过乘法使其不强依赖于原来的数据，31 作为 prime 的数学特性，奇素数，扩散性更高；

同时对其进行编译期优化，将乘法转换成移位与减法；

（1） 对于 hashCode 与 equals 方法的常规约定

- o1.equals(o2) T                           ⇒ o1.hashCode == o2.hashCode T
- o1.hashCode != o2.hashCode  ⇒ o1.equals(o2) F
- o1.hashCode == o2.hashCode ⇒ o1.equals(o2) F OR T

（2） hash 函数

① Hash 碰撞攻击:  故意构造相同的 hash 的字符

② hash 函数需要的特性
  扩散性: 仅仅改变一位就能够改变结果中的多位
  一致性: 相同的输入对应指定的输出

 （3） 简单的解决方法

1. 把某个非零的常数值，比如说17，保存在一个名为  result 的  int 类型的变量中。
2. 对于对象中每个关键域  f （equals 方法中涉及的每个域），完成以下步骤：
   a.   int 类型的散列码  c :
   i.  boolean 类型，则计算  (f ? 1 : 0) .
   ii.  byte 、  char 、  short 或者  int 类型，则计算  (int)f 。
   iii.  long 类型，则计算  (int)(f ^ (f >>> 32)) 。
   iv.  float 类型，则计算  Float.floatToIntBits(f) 。
   v.   double 类型，则计算  Double.doubleToLongBits(f) ，然后按照步骤2.a.iii，为得到的  long 类型值计算散列值。
   vi. 如果该域是一个对象引用，并且该域的  equals 方法通过递归地调用  equals 的方式来比较这个域，则同样为这个域递归地调用  hashCode
   。如果需要更复杂的比较，则为这个域计算一个“范式（canonical representation）”，然后针对这个范式调用  hashCode 。如果这个域的值为  null
   ，则返回0（或者其他某个常数，但通常是0）。
   vii.
   如果该域是一个数组，则要把每一个元素当做单独的域来处理。也就是说，递归地应用上述规则，对每个重要的元素计算一个散列码，然后根据步骤2.b中的做法把这些散列值组合起来。如果数组域中的每个元素都很重要，可
   以利用发行版本1.5中增加的其中一
3. 返回result。
4. 写完了 hashCode 方法之后，问问自己“相等的实例是否都具有相等的散列码”。要编写单
   元测试来验证你的推断。如果相等实例有着不相等的散列码，则要找出原因，并修正错
    误。

```JAVA
int hashCode() {
    int result = hashCode;
    if (hashCode == 0) {
        result = 17;
        result = 31 * result + areaCode;
        result = 31 * result + prefix;
        result = 31 * result + lineNumerber;
        result = 31 * result + obj.hashCode();
        result = 31 * result + Arrays.hashCode(products);
        hashCode = result;
    }
    return result;
}
```





**toString**

POJO 类，便于日志打印，便于找出错误

阿里开发规范中说明最好实现

```java
Public boolean equals(Object obj){
    If(this == obj) return true;
    If(obj == null) return false;
    If(this.getClass() != obj.getClass()){
  Return false;
  }
  Employee that = (Employee)obj;
  ...
}
```

JDK7 提供 Objects.equals 简化处理 NULL

```java
public boolean equals(Object obj){
    ...
    Employee that = (Employee)obj;
    return Objects.equals(this.name, that.name)
      && Objects.equals(this.salary, that.salary);
}
```



**getClass()**

获取 Class 文件的一种方式



**clone()** 

原型模式的体现

分为深拷贝、浅拷贝



实现步骤：

(1) implememnts Clonable 

(2) @Override clone()



**wait() | notify() | notifyAll**

线程中用于通信的等待通知模式

（1） wait() 调用释放锁
必须配合 synchronized(obj) 使用, 即必须要有锁, 之后调用 obj.wait()
当前线程进入 WAIT 状态
之后等待其他线程进行唤醒

（2） notify | notifyAll

随机从等待的集合中选择一个进行通知；

将所有放在等待池中的线程全部唤醒；



**finalize**

主要用于回收 JNI 调用 non-java 程序(C OR C++)

可给对象一次逃脱 GC 的机会





### 其他函数

**校验函数**

对前端传递过来的数据进行简单的校验： 此部分一般不涉及到具体的业务校验，可通过jsr 中定义的注解简化校验。

- 校验传递数据的长度，不能够大于数据库中保存的长度
- 校验多个列表项中的重复情况
- 基本的空、长度、值得范围简单校验
- 正则校验
- 业务校验 ..



**转换函数**

常见的用途：

- 配合注解进行接收参数的转换，如将字符的数据转换成特定的 Money 提高金额计算的精度以及应对高并发
- 转换成一个参数较少的类，适配特定的实体，作为 Stream 的 map 参数，快速的进行流式编程
- 在抽象的层之间转换：在 Controller -> Service -> Dao 的过程中 BO、VO、DTO ...

注： 对于前端，需要 fromJson、toJson 方法控制后端数据的接收和返回。





## 接口与实现

接口： **编译器强制的**，  一个模块间协作的合约 ( Contract )，系统对外提供的服务。



**与抽象类的区别**

层次上： 接口可多重继承，抽象类单继承；

组成特性上： 接口无成员变量，无实现的方法；

访问修饰： 接口不支持 private、protected 修饰，仅有 public 修饰；

接口不能有普通方法，抽象类可以有普通方法(模板方法模式)；



@Q： 抽象类和接口的区别？

(1) 构造方法：抽象类可以有构造方法，接口中不能有构造方法。
(2) 普通成员变量： 抽象类中可以有普通成员变量，接口中没有普通成员变量
(3) 非抽象的普通方法： 抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。
(4) 方法的访问修饰符： 抽象类中的抽象方法的访问类型可以是public，protected，但接口中的抽象方法只能是public类型的，并且默认即为public abstract类型。
(5) 静态方法： 抽象类中可以包含静态方法，接口中不能包含静态方法。JDK8 + 可以有静态方法。
(6) 静态成员变量： 抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是public static final类型，并且默认即为public static final类型。
(7) 继承和接口实现： 一个类可以实现多个接口，但只能继承一个抽象类。



## 继承与封装

**继承**

IS-A 关系

子类 **增加或修改** 基类 ( 增加成员函数，函数 )



**封装的可见性**

| 访问级别   | 类内部 | 包内部 | 派生类 | 外部 |
| ---------- | ------ | ------ | ------ | ---- |
| private    | Y      | N      | N      | N    |
| (默认)     | Y      | Y      | N      | N    |
| protectedY | Y      | Y      | Y      | N    |
| public     | Y      | Y      | Y      | Y    |

默认也称为 package private, 包级私有。

尽量 **只使用** private 和 public



**可见性的选择**

默认：

​       |-若为public， 从db读取, 加载时间长, 不可随意使用,

|-若为private， 则同时与该对象之间的关系较弱

Employee类中的static void loadAllEmployee(){}

```java
//Package private for logic in the package to control
//when employees are loaded
static void loadAllEmployee(){
    //loads all Employees from database.
}
```



## 不可变性

Immutable Objects 

**作用：**  防止打破逻辑、本身的规范，用于保证对象（数据结构）的合法性

 

**好处**

性能：  可以引用传递，可以缓存,  编译期自动优化

安全性： 线程安全

 

**特性**

类申明

函数申明

变量申明

类不可以被继承

函数不可以在派生类中重写

**变量不可以指向其它对象**

​       **|-**但可以通过引用改变其中的值,**

**必须初始化**

​       |-构造函数中

​       |-static域中

​       |-定义时

**带来的不便性**

​       **|-**无法更改引用==”****复用”**  **à**  **对于集合需要clear()****才能重用**

与C++比较：  C++中constant用法多,  Java简化了final



**实现方式**

（1） Collections

```java
class UnmodifiableCollection<E> implements Collection<E>,
 Serializable{
    final Collection<? Extends E> c;
    Iterator<E> iterator(){
	final Iterator<? Extends E> I = c.iterator();
	...
	}
}
```

（2） Guava 提供的不可变工厂模式放入



**总结**

final 关键字无法保证不可变性；

从接口定义，类的实现上保证不可变性；

Collection.unmodifiableXXX



## 泛型

**声明和使用位置**

（1） 类

在一个类中声明的子类型, 沿用大类型的泛型,  无需在子类型中添加泛型

（2） 使用在静态方法上

`new Node<Integer>(value)  Integer `    为对象的一部分

**而对于static** 函数是属于对象的,  **因而必须在静态函数上声明**

```java
static <T> LinkedList<T> newEmptyList(){}
```

（3） 使用在一般方法上， 类上不添加：
类上不添加泛型， 只在需要的方法上添加泛型
应用： 不仅仅能只能删除Integer | String， 扩大范围,  -->

```java
<T> Node<T> deleteIfEquals(Node<T> head, T value){
  ...
}
```



**使用语法**

**特殊情况的泛型比较**

 **|-**使用静态 | **一般方法,** **而类中无泛型**

```java
<V> void printList(List<V> list);
Objects.equals(list, LinkedList.<Integer>newEmptyList());
```



**Java Type Erasure**

为了兼容性  --> 运行时无泛型

​       |-仅在编译期进行

*如何获得运行时泛型的类型?*  *如何解决因为兼容性而带来的损失？*  

​       |-传递Class类型

void <T> printList(List<T> list, class<T> elementType);



**Convariance**

（1） `List<Integer> `不为 `List<Object>`

（2） 将 List<integer> 转换成 List<Object>

```java
new Array<Object>(intList);            // ✔
(List<Object>)(List)intList;           // RISK
```





**泛型数组的创建**

```java
T[] arr = (T[]) new Object[a.length];
Comparable[] arr = Array.newInstance(a.getClass().getContentType(), a.length);
```



**继承关系**

不能把`ArrayList<lnteger>`转型为`ArrayList<Number>`或`List<Number>`

`ArrayList<Number>`和`ArrayList<lnteger>`两者没有继承关系

泛型就是编写模板代码来适应任意类型

- 不必对类型进行强制转换
- 编译器将对类型进行检查
- 注意泛型的继承关系·
- 可以把ArrayList向上转型为List（T不能变！）
- 不能把ArrayList向上转型为ArrayList

泛型类型不能用于静态方法：

- 编译错误
- 编译器无法在静态字段或静态方法中使用泛型类型
- static 上的 T 与定义在 类上的 T 不是同一个类型

擦拭法 
 Type Erase : JVM 对泛型透明，只是交给编译期使用

擦拭法的局限：

- 不能是基本类型，例如int
- Object字段无法持有基本类型

擦拭法的局限：

- 无法取得带泛型的Class

擦拭法的局限：

- 不能实例化T类型
- 因为擦拭后实际上是newObject()
- 实例话T类型必须借助Class

 

```JAVA
class Pair<T> {
    private T first;
    private T last;
    public Pair(Class<T> clazz) {
        first = clazz.newInstance();
        last = clazz.newInstance();
    }
}
Pair<String> pair = new Pair<>(String.class);
Pair<String> pair = new Pair<>(Integer.class);   
```

**继承** 
 可以继承自泛型类： 
 父类的类型是pair

- 子类的类型是IntPair
- 子类可以获取父类的泛型类型Integer

Class :

- getGenericSuperClass()

ParameterizedType :

- getActualTypeArguments()

 

```JAVA
public class IntPair extends Pair<Integer> {  }
    Class<IntPair> clazz = IntPair.class;
    Type t = clazz.getGenericSuperclass();
    if (t instanceof ParameterizedType) {
    Parameterized pt = (ParameterizedType) t;
    Type[] types = pt.getActualTypeArguments();
    Type firstType = types[0];
    Class<?> typeClass = (Class<T>) firstType;
```

·Java的泛型采用擦拭法实现 
 ·擦拭法决定了泛型 
 ·不能是基本类型，例如：Int 
 ·不能获取带泛型类型的Class，例如：Pair、class 
 不能判断带泛型类型的类型，例如．xinstanceofPair 
 ·不能实例化T类型，例如．newT() 
 ·泛型方法要防止重复定义方法，例如：publicbooleanequals(Tobj) 
 ·子类可以获取父类的泛型类型



**通配符** 
 **extends** : 具有继承导致多态的特性, 只能调用 获取引用的方法

**super** 

 只能调用 传递 引用的方法

方法参数为<？extendsT>和方法参数为<？superT>的区别：

- <？extendsT> 允许调用方法获取T的引用
- <？superT> 允许调用方法传入T的引用

```java
public class Collection {
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i = 0; i < src.size(); i++) {
            T t = src.get(i);
            dest.add(t);
        }
    }
}
```



**? 通配符(少)** 
 <>的通配符·

- 不允许调用set方法（n創除外）
- 只能调用get方法获取Object引用
- Pair<>和Pair不同

使用类似<？super《nteger>通配符作为方法参数时表示。 
 。方法内部可以调用传入|nteger引用的方法 
 ObjsetXxx(lntegern) 
 。方法内部无法调用获取|nteger引用的方法(Object*外） 
 Integern=ObjgetXxx() 
 使用类似定义泛型类时表示。 
 泛型类型限定为Integer或Integer的超类 
 无限定通配符<？>很少使用，可以用替换



**与反射**

Class 是泛型 ：

- T newInstance() :
- CLass getSupperclass() :
- Constructor getConstructor():

**泛型数组** 
 **1. 可以声明带泛型的数组，但不能用new创建带泛型的数组：** 
 Pair[] ps = null;

必须通过强制转型实现带泛型的数组．

 

```
@SuppressWarnings("unchecked"）
Pair<String>[] ps = (Pair<String>[]）new Pair[2]；
```

带泛型的数组实际上是编译器的类型擦除， 

**2. 不能直接创建 T[] 数组：**

- 擦拭后代码变成 new Object
- 必须借助 Class 

 

```
public class Abc<T> {
    T[] createArray(Class<T> cls) {
        return (T[]) Array.newInstance(cls, 5);
    }
}
```

**3. 利用可变参数创建T[]数组**

- @SafeVarargs消除编译器警告



# 异常

![1553513726138](http://img.janhen.com/202101301722401553513726138.png)

列举出常见的异常:

NullPointerException:

IlleagleArgumentException, IlleagleStateException:

ArrayIndexOutOfException:

ClassNotFoundException, ClassCastException

IOException, FileNotFoundException:





**异常分类 | 区别**

（1） Error

OutOfMemoryError

（2） RuntimeException

NullPointerException

ClassCastException

（3） CheckedException

IOException

SQLException





异常处理方式
① throw

② throws

③ 系统自动抛出



throw VS throws：

throw 在程序中明确的抛出异常；

throws 语句用来标明方法不能处理的异常；



位置不同： throws用在函数上，后面跟的是异常类，可以跟多个；而throw用在函数内，后面跟的是异常对象。  


功能不同：throws用来声明异常，让调用者只知道该功能可能出现的问题，可以给出预先的处理方式；throw抛出具体的问题对象，执行到throw，功能就已经结束了，跳转到调用者，并将具体的问题对象抛给调用者。也就是说throw语句独立存在时，下面不要定义其他语句，因为执行不到。

throws表示出现异常的一种可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某种异常对象。

两者都是消极处理异常的方式，只是抛出或者可能抛出异常，但是不会由函数去处理异常，真正的处理异常由函数的上层调用处理。



受检异常和不受检异常： 

不受检查的异常不需要在方法或者是构造函数上声明，就算方法或者是构造函数的执行可能会抛出这样的异常，并且不受检查的异常可以传播到方法或者是构造函数的外面。相反，受检查的异常必须要用throws语句在方法或者是构造函数上声明。



Exception VS Error：

都是  Throwable 的子类；

Exception 用于用户程序可以捕获的异常情况；

Error 定义了不期望被用户程序捕获的异常





**其他**

try...catch 的性能

JVM 为其做了特殊处理，效率较低





**异常的 message 信息**

配合国际化进行输出提示，主要是 Spring 中集成的 i18n 国际化

配合参数化的 String 字符进行必要的参数填充

两者相互配合实现信息的展示





**异常的堆栈信息**

必要时对整个堆栈的结果进行保存，通过框架或第三方包的工具获取堆栈信息存储到数据库中





**自定义异常**

一般是运行时异常，根据不同模块需要分成不同的异常，存在一定的继承关系

支持格式化的信息展示





**异常的捕获**

- 捕获预检异常

- 捕获运行时异常

- 捕获其他模块的异常，出现在微服务中，通过 Feign 调用其他的微服务得到其他服务的异常堆栈信息，进行结果的返回

- 捕获多种类型的异常，对特定的异常做不同的处理，注意异常被捕获的顺序

```java
try {
} catch (XXException e1) {
} catch (YYException e2) { 
}
```





**异常的处理**

- 通过  try 对可能出现的每种异常进行特定类型的处理

- 通过 Spring MVC 对抛出的异常做统一的处理，借助切面控制好异常的捕获，对特定类型的异常进行自动类型匹配，统一封装好状态码返回处理

- 通过 finally 进行统一的处理，做流、资源的关闭





**Spring 中的异常处理机制**

对异常进行统一的转换，Spring 中的异常处理机制

// todo 阅读 Spring 源码

实现对 数据库中的异常进行统一的处理，。。。







异常的转换和继承：





异常与继承：

子异常和父异常的捕获顺序处理

对于普通方法，父类中定义的方法抛出的异常与对应的子类中覆盖的异常

对于构造方法，父类中抛出的异常与子类中的异常???   ==>> 派生类构造器不能捕获基类构造器抛出的异常





# JDBC

JDBC中的PreparedStatement相比Statement的好处
答：一个sql命令发给服务器去执行的步骤为：语法检查，语义分析，编译成内部指令，缓存指令，执行指令等过程。
select * from student where id =3----缓存--àxxxxx二进制命令
select * from student where id =3----直接取-àxxxxx二进制命令
select * from student where id =4--- -à会怎么干？
如果当初是select * from student where id =?--- -à又会怎么干？



上面说的是性能提高
可以防止sql注入。



Class.forName的作用?为什么要用?
答：按参数中指定的字符串形式的类名去搜索并加载相应的类，如果该类字节码已经被加载过，则返回代表该字节码的Class实例对象，否则，按类加载器的委托机制去搜索和加载该类，如果所有的类加载器都无法加载到该类，则抛出ClassNotFoundException。加载完这个Class字节码后，接着就可以使用Class字节码的newInstance方法去创建该类的实例对象了。
有时候，我们程序中所有使用的具体类名在设计时（即开发时）无法确定，只有程序运行时才能确定，这时候就需要使用Class.forName去动态加载该类，这个类名通常是在配置文件中配置的，例如，spring的ioc中每次依赖注入的具体类就是这样配置的，jdbc的驱动类名通常也是通过配置文件来配置的，以便在产品交付使用后不用修改源程序就可以更换驱动类名。





说出数据连接池的工作机制是什么?
J2EE服务器启动时会建立一定数量的池连接，并一直维持不少于此数目的池连接。客户端程序需要连接时，池驱动程序会返回一个未使用的池连接并将其表记为忙。如果当前没有空闲连接，池驱动程序就新建一定数量的连接，新建连接的数量有配置参数决定。当使用的池连接调用完成后，池驱动程序将此连接表记为空闲，其他调用就可以使用这个连接。
实现方式，返回的Connection是原始Connection的代理，代理Connection的close方法不是真正关连接，而是把它代理的Connection对象还回到连接池中。





为什么要用 ORM? 和 JDBC有何不一样?
orm是一种思想，就是把object转变成数据库中的记录，或者把数据库中的记录转变成objecdt，我们可以用jdbc来实现这种思想，其实，如果我们的项目是严格按照oop方式编写的话，我们的jdbc程序不管是有意还是无意，就已经在实现orm的工作了。
现在有许多orm工具，它们底层调用jdbc来实现了orm工作，我们直接使用这些工具，就省去了直接使用jdbc的繁琐细节，提高了开发效率，现在用的较多的orm工具是hibernate。也听说一些其他orm工具，如toplink,ojb等。









# 其他机制

## 序列化

序列化： 将对象的状态和信息保存下来，存储到文件中

反序列化： 根据文件解析得到 Java 对象



一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间。序列化是为了解决在对对象流进行读写操作时所引发的问题。





序列化的作用：

- 将对象持久化存储的一种机制，保存对象及其状态到内存或磁盘上；
- 用户远程对象传输，便于网络中二进制流的传输，实现进程之间的数据交换从而实现进程的通信；

序列化对象以字节数组保持-静态成员不保存

在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。
必须注意地是，对象序列化保存的是对象的”状态”，即它的成员变量。





**原理**

通过 ObjectInputStream 的 readObject 实现反序列化， ObjectOutputStream 的 writeObject 实现序列化





**实现**

① 序列化接口 Serializable

② ObjectOutputStream 和 ObjectInputStream 

③ writeObject 和 readObject 自定义序列化规则

④ 序列化 ID

虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致





**序列化注意事项**

① 被 `transient` 和 `static` 修饰的变量无法通过序列化；

② 要想将父类对象也序列化，就需要让父类也实现Serializable 接口。





**应用**

Tomcat 中 Session 的持久化： 

在web开发中，如果对象被保存在了Session中，tomcat在重启时要把Session对象序列化到硬盘，这个对象就必须实现Serializable接口。如果对象要经过分布式系统进行网络传输或通过rmi等远程调用，这就需要在网络上传输对象，被传输的对象就必须实现Serializable接口。





## 反射

(1) 介绍

在运行状态中，  <u>对于任意一个类都能够知道这个类所有的属性和方法；并且对于任意一个对象，都能够调用它的任意一个方法</u>；这种动态获取信息以及动态调用对象方法的功能成为Java语言的反射机制。

(2) 动态语言：

指程序在运行时可以改变其结构：新的函数可以引进，已有的函数可以被删除等结构上的变化。

从反射角度说JAVA属于半动态语言。

![1553514079461](http://img.janhen.com/202101301722471553514079461.png)



反射 API：

Class：

Field：

Method：

Constructor：





使用流程：

获取 Class，通过 Class 任意调用类的方法；

调用 Class 方法；

使用反射 API 来操作这些信息；





通过反射创建对象的两种方式：

① Class 的 newInstance()： 要求有默认构造器

② Constructor 的 newInstance()

```java
// 方式一：
Class clazz = Class.forName("xx.xx.Person");
Person p = (Person) clazz.newInstance();
// 方式二：
Constracutor c = clazz.getDeclaredConstrucotr(String.class, String.class, int.class);
Person p2 = (Person) c.newInstance("AA", "male", 23);
```





**优缺点**

（1） 优点

增加了程序的灵活性

（2） 缺点：

① 性能问题： 使用反射基本是一种解释操作， 用于字段和方法接入时要远慢于直接代码。

② 模糊程序内内部逻辑：  程序员希望在源代码中看到程序的逻辑，反射等绕过了源代码的技术，因而会带来维护问题。反射代码比相应的直接代码更复杂。





**应用场景**

（1） 应用举例
JDBC、Java 常用框架

JDK 的动态代理

android 的加载布局文件

序列化中自定义序列化规则通过反射调用 readObject、writeObject 实现；



**结构**

（1） 整体设计

```java
final class Class<T> implements java.io.Serializable,
                              GenericDeclaration, Type,  AnnotatedElement {
```

（2） 参数

name： 缓存下来，由底层提供

cachedConstructor： 用于反射实现构建对象

```
transient String name;
native String getName0();
volatile transient Constructor<T> cachedConstructor;
volatile transient Class<?>       newInstanceCallerCache;
```



**包含的类型**

类
接口
数组
8大基本类型
枚举  ∈class
注解  ∈interface
关键字(void) class



**获取方式**

- obj.getClass()
- Object.class
- Class.forName("")



 forName

获取到的 Class 为已经经过初始化阶段后的 Class



Q:`Class.forName()` 与 `ClassLoader.loadClass()` 的比较

forName 会对类进行初始化， loadClass 仅进入类的加载阶段；

forName 执行 static， loadClass 在 newInstance 才会执行 static；



**newInstance**

底层通过反射调用无参构造器实现





## 注解

三个元注解： 
@Target : 作用在 Method, Field, 

@Resource 
@Inject 
@PostConstruct

如何使用注解由工具决定

编译器可以使用的注解．

- @OverrIde：让编译器检查该方法 
  是否正确地实现了覆写
- @Deprecated：告诉编译器该方法 
  已经被标记为“作废"，在其他地方 
  引用将会出现编译警告
- @SuppressWarnings

支持的类型 ： 
．配置参数由注解类型定义 
．配置参数可以包括， 
·所有基本类型 
·String 
·枚举类型 
．数组 
·配置参数必须是常量





**定义注解**

**元注解** 
使用@Target定义Annotation可以被应用于源码的哪 
些位置． 
类或接囗：ElementType.TYPE 
字段：ElementType.FlELD 
·方法：ElementType.METHOD 
·构造方法：ElementType.CONSTRUCTOR 
·方法参数：ElementType.PARAMETER

使用@Retention定义Annotation的生命周期： 
·仅编译期：etentionPolicy.SOURCE 
·仅class文件：RetentionPoIicy.CLASS 
·运行期：RetentionPolicy.RUNTlME 
如果@Retention不存在，则该Annotation默认为 
CLASS 
通常自定义的Annotation都是RUNTIME



**Annotation的生命周期**

·RetentionPoIicy.SOURCE编译器在编译时直接丢弃 
@Override 
·RetentionPolicy.CLASS：该Annotation仅存储在cass文件中 
·RetentionPoIicy.RUNTlME在运行期可以读取该Annotation

使用@Repeatable定义Annotation是否 
可重复 
。JDK>=18

使用@]nher《ted定义子类是否可继承父 
类定义的Annotation 
·仅针对@Target为TYPE类型的 
Annotation 
。仅针对class的继承 
。对interface的继承无效



**处理注解**

如何读取RUNT|ME类型的注解？ 
·Annotation也是class 
。所有Annotation继承自 
java.langannotation.AnnotatIon 
·使用反射API

对应 API 读取 Annotation ：

- Class.isAnnotationPresent(Class):
- Field.isAnnotationPresent(Class)
- Method.isAnnotationPresent()
- Constructor.isAnnotationPresent()
- Class.getAnnotation(Class)
- Field.getAnnotation(Class)
- Method.getAnnotation(Class)
- Constructor.getAnnotation(Class)

·可以在运行期通过反射读取RUNT牖E类型的注解 
不要漏写@Retention(RetentionPoIicy.RUNTlME) 
·可以通过工具处理注解来实现相应的功能： 
·对JavaBean的属性值按规则进行检查 
·JUnit会自动运行@Test注解的测试方法


