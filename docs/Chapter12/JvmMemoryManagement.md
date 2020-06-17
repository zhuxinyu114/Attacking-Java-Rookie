# JVM内存管理

根据JVM规范，JVM把内存划分了如下几个区域：方法区、堆区、本地方法栈、虚拟机栈、程序计数器。
其中，方法区和堆是所有线程共享的。



#### 方法区

是java虚拟机规范中定义的名字 各个虚拟机实现上有所不同
HostSpot虚拟机中
1.在jdk1.7 以及前的版本实现的方法区称为- - -永久代

2.在java 虚拟机的堆内存中分配

3.里面主要存放的内容：已经被虚拟机加载的类信息，常量，静态变量，即时编译后的代码等

4.内存回收：主要是常量池的回收 和类型的卸载- -目前的回收效果不好

方法区存放了要加载的类的信息(如类名，修饰符)、类中的静态变量、final定义的常量、类中的field、方法信息，
当开发人员调用类对象中的getName、isInterface等方法来获取信息时，
这些数据都来源于方法区。方法区是全局共享的，在一定条件下它也会被GC。
当方法区使用的内存超过它允许的大小时，就会抛出OutOfMemory：PermGen Space异常。



在Hotspot虚拟机中，这块区域对应的是Permanent Generation(持久代)，
一般的，方法区上执行的垃圾收集是很少的，因此方法区又被称为持久代的原因之一，
但这也不代表着在方法区上完全没有垃圾收集，其上的垃圾收集主要是针对常量池的内存回收和对已加载类的卸载。
在方法区上进行垃圾收集，条件苛刻而且相当困难。

运行时常量池（Runtime Constant Pool）是方法区的一部分，
用于存储编译期就生成的字面常量、符号引用、
翻译出来的直接引用（符号引用就是编码是用字符串表示某个变量、接口的位置，
直接引用就是根据符号引用翻译出来的地址，将在类链接阶段完成翻译）；
运行时常量池除了存储编译期常量外，也可以存储在运行时间产生的常量，
比如String类的intern()方法，作用是String维护了一个常量池，如果调用的字符“abc”已经在常量池中，
则返回池中的字符串地址，否则，新建一个常量加入池中，并返回地址。

分配在方法区（永久代）中的，**但是1.7版本把 字符串常量池 单独拿到了堆空间中 **
存的内容：用于存放编译期生成的各种字面量以及符号引用，时机可能是静态编译期 也可以是动态编译时候



#### 堆

堆区是理解Java GC机制最重要的区域。
在JVM所管理的内存中，堆区是最大的一块，堆区也是JavaGC机制所管理的主要内存区域，
堆区由所有线程共享，在虚拟机启动时创建。堆区用来存储对象实例及数组值，
可以认为java中所有通过new创建的对象都在此分配。

对于堆区大小，可以通过参数-Xms和-Xmx来控制，-Xms为JVM启动时申请的最新heap内存，
默认为物理内存的1/64但小于1GB;-Xmx为JVM可申请的最大Heap内存，
默认为物理内存的1/4但小于1GB,默认当剩余堆空间小于40%时，JVM会增大Heap到-Xmx大小，
可通过-XX:MinHeapFreeRadio参数来控制这个比例；
当空余堆内存大于70%时，JVM会减小Heap大小到-Xms指定大小，
可通过-XX:MaxHeapFreeRatio来指定这个比例。对于系统而言，为了避免在运行期间频繁的调整Heap大小，
我们通常将-Xms和-Xmx设置成一样。
为了让内存回收更加高效，从Sun JDK 1.2开始对堆采用了分代管理方式，如下图所示：





###### 年轻代（Young Generation）

对象在被创建时，内存首先是在年轻代进行分配（注意，大对象可以直接在老年代分配）。
当年轻代需要回收时会触发Minor GC(也称作Young GC)。

年轻代由Eden Space和两块相同大小的Survivor Space（又称From Space和To Space）构成，
Eden区和Servior区的内存比为8:1，可通过-Xmn参数来调整新生代大小，
也可通过-XX:SurvivorRadio来调整Eden Space和Survivor Space大小。
不同的GC方式会按不同的方式来按此值划分Eden Space和Survivor Space，
有些GC方式还会根据运行状况来动态调整Eden、From Space、To Space的大小。

年轻代的Eden区内存是连续的，所以其分配会非常快；
同样Eden区的回收也非常快
（因为大部分情况下Eden区对象存活时间非常短，而Eden区采用的复制回收算法，
此算法在存活对象比例很少的情况下非常高效）。
如果在执行垃圾回收之后，仍没有足够的内存分配，也不能再扩展
，将会抛出OutOfMemoryError:Java Heap Space异常。

###### 老年代（Old Generation）

老年代用于存放在年轻代中经多次垃圾回收仍然存活的对象，
可以理解为比较老一点的对象，例如缓存对象；
新建的对象也有可能在老年代上直接分配内存，
这主要有两种情况：
一种为大对象，可以通过启动参数设置-XX:PretenureSizeThreshold=1024，
表示超过多大时就不在年轻代分配，而是直接在老年代分配。此参数在年轻代采用Parallel Scavenge GC时无效，
因为其会根据运行情况自己决定什么对象直接在老年代上分配内存；另一种为大的数组对象，
且数组对象中无引用外部对象。

当老年代满了的时候就需要对老年代进行垃圾回收，老年代的垃圾回收称作Full GC。
老年代所占用的内存大小为-Xmx对应的值减去-Xmn对应的值。

#### 本地方法栈（Native Method Stack）

本地方法栈用于支持native方法的执行，存储了每个native方法调用的状态。
本地方法栈和虚拟机方法栈运行机制一致，它们唯一的区别就是，虚拟机栈是执行Java方法的，
而本地方法栈是用来执行native方法的，在很多虚拟机中（如Sun的JDK默认的HotSpot虚拟机），
会将本地方法栈与虚拟机栈放在一起使用。

#### 程序计数器（Program Counter Register）

程序计数器是一个比较小的内存区域，可能是CPU寄存器或者操作系统内存，
其主要用于指示当前线程所执行的字节码执行到了第几行，可以理解为是当前线程的行号指示器。
字节码解释器在工作时，会通过改变这个计数器的值来取下一条语句指令。
 每个程序计数器只用来记录一个线程的行号，所以它是线程私有（一个线程就有一个程序计数器）的。

如果程序执行的是一个Java方法，则计数器记录的是正在执行的虚拟机字节码指令地址；
如果正在执行的是一个本地（native，由C语言编写完成）方法，则计数器的值为Undefined，
由于程序计数器只是记录当前指令地址，所以不存在内存溢出的情况，
因此，程序计数器也是所有JVM内存区域中唯一一个没有定义OutOfMemoryError的区域。

#### 虚拟机栈（JVM Stack）

虚拟机栈占用的是操作系统内存，每个线程都对应着一个虚拟机栈，它是线程私有的，
而且分配非常高效。一个线程的每个方法在执行的同时，都会创建一个栈帧（Statck Frame），
栈帧中存储的有局部变量表、操作站、动态链接、方法出口等，当方法被调用时，
栈帧在JVM栈中入栈，当方法执行完成时，栈帧出栈。

局部变量表中存储着方法的相关局部变量，包括各种基本数据类型，
对象的引用，返回地址等。在局部变量表中，
只有long和double类型会占用2个局部变量空间（Slot，对于32位机器，一个Slot就是32个bit），
其它都是1个Slot。需要注意的是，局部变量表是在编译时就已经确定好的，
方法运行所需要分配的空间在栈帧中是完全确定的，在方法的生命周期内都不会改变。

虚拟机栈中定义了两种异常，如果线程调用的栈深度大于虚拟机允许的最大深度，
则抛出StatckOverFlowError（栈溢出）；
不过多数Java虚拟机都允许动态扩展虚拟机栈的大小(有少部分是固定长度的)，
所以线程可以一直申请栈，直到内存不足，此时，会抛出OutOfMemoryError（内存溢出）。

#### 元数据区

1.8版本，移除永久代，改为元数据区，元数据区分配在本地内存就是系统可用内存空间，
字符串常量池 还是在堆内的
优点：元空间的最大可分配空间就是系统可用内存空间。不容易发生内存溢出的问题

#### 永久代转元数据区的原因：
1.字符串存在永久代中，容易发生内存溢出的问题

2.类及犯法的信息比较难确定，因此对于永久代的大小指定比较困难，
太小容易出现永久代溢出，太大则容易导致老年代溢出。

3.永久代会为GC带来不必要的复杂度

#### 元数据区存在的问题：内存碎片

元空间虚拟机采用了组块分配的形式，同时区块的大小由类加载器类型决定。
类信息并不是固定大小，因此有可能分配的空闲区块和类需要的区块大小不同，这种情况下可能导致碎片存在。
元空间虚拟机目前并不支持压缩操作，所以碎片化是目前最大的问题。

#### 补录

在 Jdk6 以及以前的版本中，字符串的常量池是放在堆的Perm区的，Perm区是一个类静态的区域，
主要存储一些加载类的信息，常量池，方法片段等内容，默认大小只有4m，
一旦常量池中大量使用 intern 是会直接产生java.lang.OutOfMemoryError:PermGen space错误的。
在 jdk7 的版本中，字符串常量池已经从Perm区移到正常的Java Heap区域了。
为什么要移动，Perm 区域太小是一个主要原因，
Jdk8已经直接取消了Perm区域，而新建立了一个元区域。
应该是jdk开发者认为Perm区域已经不适合现在 JAVA 的发展了。













# Java方法区、永久代、元空间、常量池详解

#### 1.JVM内存模型简介

堆——堆是所有线程共享的，主要用来存储对象。其中，堆可分为：年轻代和老年代两块区域。
使用NewRatio参数来设定比例。对于年轻代，一个Eden区和两个Suvivor区，使用参数SuvivorRatio来设定大小；
Java虚拟机栈/本地方法栈——线程私有的，主要存放局部变量表，操作数栈，动态链接和方法出口等；
程序计数器——同样是线程私有的，记录当前线程的行号指示器，为线程的切换提供保障；
方法区——线程共享的，主要存储类信息、常量池、静态变量、JIT编译后的代码等数据。
方法区理论上来说是堆的逻辑组成部分；
运行时常量池——是方法区的一部分，用于存放编译期生成的各种字面量和符号引用；

#### 2.永久代和方法区的关系

      涉及到内存模型时，往往会提到永久代，那么它和方法区又是什么关系呢？
《Java虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。
那么，在不同的 JVM 上方法区的实现肯定是不同的了。 
同时大多数用的JVM都是Sun公司的HotSpot。在HotSpot上把GC分代收集扩展至方法区，
或者说使用永久代来实现方法区。
因此，我们得到了结论，永久代是HotSpot的概念，方法区是Java虚拟机规范中的定义，
是一种规范，而永久代是一种实现，一个是标准一个是实现。
其他的虚拟机实现并没有永久带这一说法。在1.7之前在(JDK1.2 ~ JDK6)的实现中，
HotSpot 使用永久代实现方法区，HotSpot 使用 GC分代来实现方法区内存回收，
可以使用如下参数来调节方法区的大小:

```
-XX:PermSize
方法区初始大小
-XX:MaxPermSize
方法区最大大小
超过这个值将会抛出OutOfMemoryError异常:java.lang.OutOfMemoryError: PermGen
```
#### 3.元空间

      对于Java8， HotSpots取消了永久代，那么是不是也就没有方法区了呢？
当然不是，方法区是一个规范，规范没变，它就一直在。那么取代永久代的就是元空间。
它可永久代有什么不同的？存储位置不同，永久代物理是是堆的一部分，和新生代，老年代地址是连续的，
而元空间属于本地内存；存储内容不同，元空间存储类的元信息，
静态变量和常量池等并入堆中。相当于永久代的数据被分到了堆和元空间中。

#### 4.Class文件常量池

       Class 文件常量池指的是编译生成的 class 字节码文件，
其结构中有一项是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，
这部分内容将在类加载后进入方法区的运行时常量池中存放。

这里的字面量是指字符串字面量和声明为 final 的（基本数据类型）常量值，
这些字符串字面量除了类中所有双引号括起来的字符串(包括方法体内的)，
还包括所有用到的类名、方法的名字和这些类与方法的字符串描述、字段(成员变量)的名称和描述符；
声明为final的常量值指的是成员变量，不包含本地变量，
本地变量是属于方法的。这些都在常量池的 UTF-8 表中(逻辑上的划分)；
符号引用，就是指指向 UTF-8 表中向这些字面量的引用，
包括类和接口的全限定名(包括包路径的完整名)、字段的名称和描述符、方法的名称和描述符。
只不过是以一组符号来描述所引用的目标，和内存并无关，所以称为符号引用，
直接指向内存中某一地址的引用称为直接引用；

#### 5.运行时常量池

       运行时常量池是方法区的一部分，是一块内存区域。
Class 文件常量池将在类加载后进入方法区的运行时常量池中存放。
一个类加载到 JVM 中后对应一个运行时常量池，运行时常量池相对于 Class 文件常量池来说具备动态性，
Class 文件常量只是一个静态存储结构，里面的引用都是符号引用。
而运行时常量池可以在运行期间将符号引用解析为直接引用。
可以说运行时常量池就是用来索引和查找字段和方法名称和描述符的。
给定任意一个方法或字段的索引，通过这个索引最终可得到该方法或字段所属的类型信息和名称及描述符信息，
这涉及到方法的调用和字段获取。

#### 6.字符串常量池
       字符串常量池是全局的，JVM 中独此一份，因此也称为全局字符串常量池。
运行时常量池中的字符串字面量若是成员的，则在类的加载初始化阶段就使用到了字符串常量池；
若是本地的，则在使用到的时候（执行此代码时）才会使用到字符串常量池。
其实，“使用常量池”对应的字节码是一个 ldc 指令，在给 String 类型的引用赋值的时候会先执行这个指令，
看常量池中是否存在这个字符串对象的引用，若有就直接返回这个引用，
若没有，就在堆里创建这个字符串对象并在字符串常量池中记录下这个引用（jdk1.7)。
String 类的 intern() 方法还可在运行期间把字符串放到字符串常量池中。
JVM 中除了字符串常量池，8种基本数据类型中除了两种浮点类型剩余的6种基本数据类型的包装类，
都使用了缓冲池技术，
但是 Byte、Short、Integer、Long、Character 
这5种整型的包装类也只是在对应值在 [-128,127] 时才会使用缓冲池，超出此范围仍然会去创建新的对象。
其中：


在 jdk1.6（含）之前也是方法区的一部分，并且其中存放的是字符串的实例；

在 jdk1.7（含）之后是在堆内存之中，存储的是字符串对象的引用，字符串实例是在堆中；

jdk1.8 已移除永久代，字符串常量池是在本地内存当中，存储的也只是引用。



https://docs.oracle.com/javase/specs/jvms/se13/jvms13.pdf
https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6