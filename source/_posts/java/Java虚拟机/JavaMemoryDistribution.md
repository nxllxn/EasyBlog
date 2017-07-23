---
title: Java虚拟机--运行时数据区
tags:
     - Java
     - Java虚拟机
     - JVM
     - 运行时数据区
---

<!--## 写在开头-->

<!--LZ是一名软件工程专业的毕业生，毕业已有一年，过去一年里也写了一些代码，不过个人感觉写出来的东西还是有些不成体系。干任何事情都有无规矩不成方圆这么一说，软件开发是个精细的活，尤其得注重“规矩”。不过，一来没有经验老道之人给我带带路，而来本人性格懒惰、急躁，静不下心来，遇事也多是一副不求甚解的态度，所以终究不能窥其门径而入。日子一天快过一天，总不能天天这么混日子吧。间歇性雄心壮志，持续性混吃等死？这可要不得。既然入了坑，那还得想办法爬出去。-->


## Java内存区域

> Java与C++之间有一堵由内存分配和垃圾收集机制所围成的“高墙”，墙外面的人想进去，墙里面的人想出来

### 概述

如果你是一名c或者c++程序员，不可否认你对于每一个可用的内存单元都有完全的控制，但是你也不得不承认为了保证没有内存泄漏，你不得不时刻战战兢兢，调动每一个脑细胞时刻留意malloc，alloc，以及new等操作新分配的内存区域。
如果你是一名java程序员，恭喜你，不戴这王冠，也就无需承其重。不过即使是美好的东西，如果不能掌控，也可能后患无穷。你可能觉得我危言耸听，等你真正遇到java的百慕大三角你便会明白以为你明白了和你明白了绝大多数时候都是不能划等号的。
本节会介绍Java运行时数据区，本文参考自《深入理解Java虚拟机：JVM高级特性与最佳实践（最新第二版）》第二章，并会对关键知识点做梳理，同时也提供了详细的测试实例。[测试源码托管至github](https://github.com/nxllxn/JavaJvmTestCase)。

<!-- more -->

### 运行时数据区域
Java虚拟机在执行Java程序时会将它所管理的内存划分为多个不同的数据区域，每个数据区域有着不同的职责，以及特定的生命周期。根据《Java虚拟机规范（java se 7版）》，Java虚拟机所管理的内存将会包括如下几个数据区域，见下图。
![Java运行时数据区](/images/java_runtime_memory_area.jpg)

#### 程序计数器  
程序计数器（Program Counter Register）是一块较小的内存空间，**注意这里说的是一块，之前理解的程序计数器就是一个单独的内存单元，还是有点想当然了，具体为什么是一块内存区域，请继续往下看**。

程序计数器是作为当前线程所执行的字节码的行号指示器。这个很好理解了，程序计数器保存了当前执行指令的地址。当当前指令执行完成，根据具体的顺序，分支，循环，异常处理，线程恢复等控制程序计数器的值来决定下一条需要执行的指令地址。
Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的（不用多说，现在的计算机基本都是分时系统），但是需要注意的一点是，多个线程会交替执行，这中间便存在线程切换，为了使线程切换后能够恢复到正确的执行位置每条线程都需要一个独立的程序计数器，各个线程之间计数器互不影响，独立存储。我们称这类内存区域为“线程私有”的内存。
    
#### Java虚拟机栈  
虚拟机栈描述的是Java方法执行的内存模型，每个方法在执行的时候都会创建一个栈帧。如果不清楚这一点，可以网上查阅关于方法栈相关的资料，不过LZ坚持认为最好的方式是使用c或者c++这类语言，实现一个简单的函数调用，然后以debug的模式逐步跟踪方法栈从建立到销毁,栈上内存分配的全过程。当你知道方法参数如何传递，返回地址如何存放，局部变量内存单元如何分配，之后的知识就很容易理解了。

万变不离其宗，Java的方法栈帧用于存储局部变量表，操作数栈，动态链接，方法出口等信息。每一个方法从调用到执行再到返回，就对应着一个栈帧在虚拟机栈中入栈到出栈的全过程。

经常有程序员按照自身的关注点，以对象内存分配关系为区分，将Java内存区域分为堆内存和栈内存，不过这么分太过笼统，Java内存模型远比这复杂。

通常程序员所说的栈就是指虚拟机栈，或者说是虚拟机栈中的局部变量表部分。局部变量表存储了编译期可知的各种基本数据类型（boolean，int，float，long，double）、对象引用以及返回地址。long和double占据两个内存单元或者说slot。其他数据类型都只占据一个slot。局部变量表所需的空间在编译期间就已经确定，并且在方法运行期间都不会发生改变。

此外，这个内存区域定义了两种形式的异常：
* 如果线程请求的栈深度大于虚拟机所允许的深度（此处需要注意一下，允许的栈深度通常不固定，根据LZ理解，栈深度或者说栈的数目=可用栈空间/栈帧的大小，也就是说总内存一定的情况下，分配的局部变量较少通常能够保证更多的方法栈的建立），将抛出StackOverFlowError异常
* 如果虚拟机栈可以动态扩展（当前大部分虚拟机都可以扩展），扩展过程中无法申请到足够的内存，将抛出OutOfMemoryError异常
    
关于Java虚拟机栈更详细的知识请参考[Java虚拟机栈](http://www.cnblogs.com/niejunlei/p/5987611.html)
        
#### 本地方法栈（Native Method Stack）  
本地方法栈与虚拟机栈作用非常相似，他们之间的区别只不过是虚拟机栈为虚拟机执行Java方法（也就是用户程序或者说用户程序字节码）服务，而本地方法栈则为虚拟机使用到的native方法服务。
#### Java堆（Java Heap）  
对于大多数应用来说，Java堆是Java虚拟机所管理的内存中最大的一块。Java堆是被所有对象共享的一块内存区域，在虚拟机启动时创建。此内存的唯一目的就是存放对象的引用，几乎所有的对象实例都在这里分配内存。

Java堆也是垃圾收集器管理的主要区域，因此很多时候也被称为“GC堆”，从内存回收角度来看，由于现在收集器基本都采用分代收集算法，所以Java堆中还可以细分为新生代和老年代，再细致一点的有Eden空间，FromSurvivor空间，ToSurvivor空间。
    
#### 方法区（Method Area）  
方法区与java堆一样，由各个线程共享，用于存储被虚拟机加载的类信息，常量，静态变量，即时编译后的代码等数据。
#### 运行时常量池（Runtime Constant Pool） 
**运行时常量池是方法区的一部分**，Class文件中除了有类的版本，字段，方法，接口等描述信息，还有一项信息是常量池。用于存放编译期生成的各种字面量和符号引用。
既然运行时常量区是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时便会抛出OutOfMemoryError。常见的字符串的拼接会导致常量区存储一个新的字符串常量，如果在一个循环中不停的进行字符串的拼接将会导致OutOfMemoryError.
#### 直接内存（Direct Memory）  
直接内存不是Java虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存被频繁使用。比如Java nio（new IO）中引入了一种基于通道与缓冲区的IO方式，它使用Native函数库直接分配堆外内存，
然后通过一个存贮在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作，这样可以在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

### HotSpot虚拟机对象探秘
#### 对象创建  
虚拟机遇到一条New指令时，首先检查这个指令的参数是否在常量池中能够定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载，解析和初始化过，如果没有，那必须先进行类的加载，之后指定章节会详细讨论类的加载。
如果类加载完成。接下来虚拟机将为新生对象分配内存，对象所需的大小在编译期已经确定，为对象分配空间等同于将一块确定大小的内存从Java堆中划分出来。
假设Java堆是绝对规整的，所有用过的内存都放在一边，空闲的放在另外一边，中间放着一个指针作为分界点的指示器。那分配内存就好比将指针朝着空闲空间那边移动一段与对象大小相等的距离。这种方式称为指针碰撞。
如果Java堆不是规整的，已使用的内存和未使用的内存相互交错，那么虚拟机就只能维护一个列表，标识出内存中有哪些内存块是可用的。在分配的时候找到一块足够大的空间划分给对象（此处找一块空闲空间甚至可以单独拆出来写一个小节，为什么这么说呢，我们选一个最小的能满足需求的内存块进行分分配的话，剩余的空间会很小，这样容易产生内存碎片，如果我们选一个较大的空间来分配，虽然可以保证减少内存碎片，但是内存空间会越来越零散）。这种分配方式称之为“空闲列表”。
具体选择哪种方式视java堆是否规整而定，而Java堆是否规整通常取决于对应的垃圾收集算法是否带有压缩整理的功能。因此在使用Serial，ParNew等带Compact过程的收集器时，系统采用的是指针碰撞，而使用CMS这种基于Mark-sweep算法的收集器
通常采用空闲列表。

此外，由于堆内存相当于临界资源，多个线程并发访问进行内存分配，所以不可避免的需要线程同步机制，实际上虚拟机采用CAS配上失败重试的方式保证更新操作的原子性。另一种实现方式是，把内存分配的动作按照线程划分在不同的空间中进行，即每个线程在Java堆中预先分配一小块内存。称为本地线程分配缓冲，哪个线程需要分配内存就在自己的缓冲区上分配，只有缓冲区使用完需要重新分配新的缓冲区时才需要同步锁定。

内存分配完成后，虚拟机将分配到的内存空间除了对象头都初始化为零值（突然想到c语言中将内存空间初始化为0xcc0xcc0xcc0xcc0xcc0xcc，最终显示为烫烫，哈哈）。

接下来，虚拟机要对这个对象进行必要的设置，例如这个对象是哪个类的实例，如何才能找到类的元数据信息，对象的哈希码，GC分代年龄等。

这时在虚拟机看来一个对象已经生成，但从Java程序的角度来看，对象创建才刚刚开始。init方法还没有执行。所有的字段都还为零，所以一般来说，执行new指令之后，会接着执行init方法，对应的是一条invokeSpecial指令，把对象按照程序员的意愿进行初始化。
* 对象的内存布局  
在HotSpot虚拟机中，对象在内存中的布局可以分为3块区域：对象头（Header），实例数据（Instance Data）和对齐填充（Padding）。
    * 对象头第一部分用于存储对象自身的运行时数据，如哈希码，GC分代年龄，锁状态标志，线程持有的锁，偏向线程Id，偏向时间戳等。这部分数据通常与对象具体内容无关，所以为了节省额外成本，
        一般使用32位（32位机）或者64位（64位机）来存储，32位或者64位中不同区段分别代表着上述不同的子模块。
    * 对象头第二部分是类型指针，即指向它的类元数据的指针，虚拟机通过这个指针来确定当前对象是哪个类的实例。
    * 接下来的实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。包括从付父类继承下来的以及子类中新定义的。
    * 最后一部分对齐填充并不是必然存在的，由于HotSpotVm的自动内存管理系统要求对象起始地址必须是8字节的整数倍，所以如果实例数据部分末尾不足8字节需要进行填充来补全。
* 对象的访问定位  
我们的程序需要通过栈上的对象引用来操作堆上的具体对象。通常有两种实现方式：
    * 使用句柄访问，是指首先在Java堆中划分一块内存作为句柄池。对象引用中存储的就是句柄地址，而具体的句柄中包含了对象实例数据与类型数据各自具体的地址信息。
    * 使用直接指针访问，java栈中的引用存储的就是对象的地址，对象实例中保存指向当前类型数据的地址
    * 二者的优劣
        * 使用句柄访问好处就是，reference中存储的是稳定的句柄地址，在对象被移动时（比如垃圾回收的compact）只会改变句柄中的实例数据指针，而reference本身无需修改
        * 使用直接指针访问的好处就是，速度更快，因为它节省了一次指针定位的开销，由于对象在Java中的访问十分频繁。因此此类开销积少成多后也是一项可观的执行成本。

#### 实战：OutOfMemoryError  
本部分主要有两个目的：
* 通过代码验证Java虚拟机规范中描述的各个运行时区域的内容
* 希望读者在工作中遇到实际的内存溢出异常时能够根据异常信息快速判断哪个区域的内存溢出。知道为什么导致内存溢出，以及应该如何处理

> 注意：代码的开头都有设置虚拟机启动参数，读者请按照示例进行设置，因为这些参数将直接控制示例程序的行为


> Intellij IDEA JVM参数设置参考资料：[Intellij IDEA JVM参数设置](http://blog.csdn.net/u013063153/article/details/53762019)

##### Java堆溢出  
Java堆用于存储对象实例，如果不断的创建对象，并且保证GCRoots到对象之间有可达路径（利用List进行对象引用的存储，List类的栈对象引用指向List堆实例，List堆实例中的每个对象引用再指向具体的堆上的对象实例）来避免垃圾回收机制来回收这些对象，那么当对象实例占据的存储空间超过最大堆容量之后就会产生堆内存溢出。

设置参数限制Java堆大小为20Mb，不可扩展（将堆的最小值-Xms参数与最大参数-Xmx设置为一样即可避免堆自动扩展），通过参数-XX:+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前内存堆转储快照以便事后分析。
        
    
```
/**
 * -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError -verbose:gc -XX:+PrintGCDetails -XX:SurvivorRatio=8
 */
public class HeapOutOfMemory {
    public static class SomeClass{
        public SomeClass() {
        }
    }

    public static void main(String[] args) {
        List<SomeClass> someClasses = new ArrayList<SomeClass>();
        try {
            while (true){
                someClasses.add(new SomeClass());
            }
        }catch (OutOfMemoryError e){
            System.out.println("Total instance created:" + someClasses.size());
        }
    }
}
```
        
        
这段代码很简单，我们在一个while循环中不停地new出SomeClass的对象，并将这个实例的引用保存到一个列表中以避免它被垃圾回收机制回收。下面是这段代码在我的笔记本电脑上的运行结果。
           
```
[GC (Allocation Failure) [PSYoungGen: 5632K->488K(6144K)] 5632K->3021K(19968K), 0.0047814 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 6120K->504K(6144K)] 8653K->7804K(19968K), 0.0066141 secs] [Times: user=0.11 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 6136K->0K(6144K)] [ParOldGen: 10465K->12311K(13824K)] 16601K->12311K(19968K), [Metaspace: 3496K->3496K(1056768K)], 0.1781231 secs] [Times: user=0.31 sys=0.00, real=0.18 secs] 
[Full GC (Ergonomics) [PSYoungGen: 4393K->2754K(6144K)] [ParOldGen: 12311K->13733K(13824K)] 16704K->16487K(19968K), [Metaspace: 3497K->3497K(1056768K)], 0.1530778 secs] [Times: user=0.86 sys=0.03, real=0.15 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 2754K->2754K(6144K)] [ParOldGen: 13733K->13715K(13824K)] 16487K->16469K(19968K), [Metaspace: 3497K->3497K(1056768K)], 0.1069225 secs] [Times: user=0.58 sys=0.02, real=0.11 secs] 
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid1940.hprof ...
Heap dump file created [28250225 bytes in 0.123 secs]
Total instance created:810325
Heap
     PSYoungGen      total 6144K, used 2921K [0x00000000ff980000, 0x0000000100000000, 0x0000000100000000)
         eden space 5632K, 51% used [0x00000000ff980000,0x00000000ffc5a568,0x00000000fff00000)
         from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
         to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
     ParOldGen       total 13824K, used 13715K [0x00000000fec00000, 0x00000000ff980000, 0x00000000ff980000)
         object space 13824K, 99% used [0x00000000fec00000,0x00000000ff964d90,0x00000000ff980000)
     Metaspace       used 3504K, capacity 4502K, committed 4864K, reserved 1056768K
         class space    used 388K, capacity 390K, committed 512K, reserved 1048576K
```

* 首先可以看出设置了-verbose:gc参数之后，程序运行期间发生的GC操作的相关日志也被打印出来了。这部分内容我们会在之后的章节做具体分析，现在暂且略过。
* 然后是第一部分比较重要的打印java.lang.OutOfMemoryError: Java heap space。这行结果说明发生了内存溢出，具体溢出位置是Java虚拟机堆内存。
* 第二部分Dumping heap to java_pid1940.hprof ...是因为我们设置了-XX:+HeapDumpOnOutOfMemoryError参数，这个参数的意思是在发生内存溢出的时候dump出当前堆内存的快照，Java自带的内存分析工具jhat可以对这个文件进行解析，并展示出堆内存按照类引用为区分的内存分配情况。具体如何使用JHat我们也会放到之后Java命令行工具一节再做详细介绍。当然有兴趣的同学可以自行查阅相关资料。
* 第三部分Total instance created:810325是由我们的代码打印的，可以看出直到堆内存被分配完毕并抛出OutOfMemoryError，我们一共创建了810325个SomeClass的实例。
* 第四部分则由虚拟机参数-XX:+PrintGCDetails控制，就是在程序运行完毕，打印出当前内存各个模块的使用情况。

堆内存溢出相对来说应该是比较常见的内存溢出异常，如果真的是因为内存泄漏，我们可以利用内存分析工具查看泄漏对象（这里说泄漏对象是指，在逻辑上已经不会再访问这个对象了，但是这个对象仍然被直接或间接的引用着，比如某个类中的静态引用）到GCRoots的引用链，这样就可以知道泄漏对象是通过怎样的的路径与GCroots（通常是指栈内存中的引用）相关联而导致垃圾收集器无法自动回收它们。找到了泄漏对象的类型信息以及引用链信息，就可以比较准确地定位泄漏代码的位置了。
如果不存在内存泄漏，也就是说，内存中的每个对象实例确实都还必须存活，那么就应当检查虚拟机的堆参数（-Xms、-Xmx），根据具体的硬件条件看是否还可以调大。同时也检查代码看是否存在某些对象生命周期过长，持有状态时间过长的情况。尝试减少程序运行期的内存消耗。
上述便是处理Java堆内存问题最基本，最简单的思路了。

##### 虚拟机栈内存溢出
* 程序(每一个线程)运行期间，会有一个Java虚拟机栈，默认栈的大小为1M，用户也可以通过-Xss虚拟机参数设置栈的大小
* Java虚拟机规范：如果请求的栈深度超过了虚拟机所允许的最大深度（其实LZ认为这个深度是相对的，深度=栈总大小/栈帧平均大小），将抛出StackOverFlow异常
* 如果虚拟机在扩展时无法申请到足够的内存空间，那么抛出OutOfMemoryError
* 再次声明，上面两个描述虽然做了区分，但是本质上是一样的，内存总的空间一定，当栈空间无法分配时，可以说内存太小，也可以说用于栈帧分配的空间太大。
* 具体应该抛出什么异常其实不是那么严格。因为栈帧深度也受限于栈的大小，可以说栈帧数目太多，当然也可以说内存太少。
* 实际测试中，使用-Xss参数减少栈内存容量，结果抛出StackOverFlow异常，异常出现时输出的栈深度相应缩小
* 实际测试中，如果为待调用方法定义大量局部变量，也就是说在内存分配时需要为这个方法的调用分配更大的栈帧，结果抛出StackOverFlowError，异常出现时输出的栈深度相应缩小

```

/**
 * VM Args:-Xss128k 当指定为128k，报异常：The stack size specified is too small, Specify at least 228k，LZ最小使用256k
 */
public class StackOverFlow {
    private int stackDeep;

    private void recursiveCall() {
        this.stackDeep++;

        recursiveCall();
    }

    private static class RecursiveReference {
        private RecursiveReference another;

        private static int referCount = 0;

        private void refer() {
            referCount++;

            if (another != null) {
                another.refer();
            }
        }
    }

    private static void recursiveReference() {
        RecursiveReference currentRecursiveReference = new RecursiveReference();
        RecursiveReference anotherRecursiveReference = new RecursiveReference();

        currentRecursiveReference.another = anotherRecursiveReference;
        anotherRecursiveReference.another = currentRecursiveReference;

        currentRecursiveReference.refer();
    }

    private void recursiveCallWithBigStackFrame() {
        this.stackDeep++;

        double occupyPositionA;
        double occupyPositionB;
        double occupyPositionC;

        recursiveCallWithBigStackFrame();
    }

    private static class RecursiveReferenceWithBigStackFrame {
        private RecursiveReferenceWithBigStackFrame another;

        private static int referCount = 0;

        private void refer() {
            referCount++;

            double occupyPositionA;
            double occupyPositionB;
            double occupyPositionC;

            if (another != null) {
                another.refer();
            }
        }
    }

    private static void recursiveReferenceWithBigStackFrame() {
        RecursiveReferenceWithBigStackFrame currentRecursiveReference = new RecursiveReferenceWithBigStackFrame();
        RecursiveReferenceWithBigStackFrame anotherRecursiveReference = new RecursiveReferenceWithBigStackFrame();

        currentRecursiveReference.another = anotherRecursiveReference;
        anotherRecursiveReference.another = currentRecursiveReference;

        currentRecursiveReference.refer();
    }

    public static void main(String[] args) {
        StackOverFlow stackOverFlow = null;
        try {
            stackOverFlow = new StackOverFlow();
            stackOverFlow.recursiveCall();
        }catch (StackOverflowError e){
            //e.printStackTrace();

            System.out.println("使用方法循环调用栈深度：" + (stackOverFlow == null ? "null" : stackOverFlow.stackDeep));
        }

        try {
            recursiveReference();
        }catch (StackOverflowError e){
            //e.printStackTrace();

            System.out.println("使用对象循环引用栈深度：" + RecursiveReference.referCount);
        }


        try {
            stackOverFlow = new StackOverFlow();
            stackOverFlow.recursiveCallWithBigStackFrame();
        }catch (StackOverflowError e){
            //e.printStackTrace();

            System.out.println("使用方法循环调用(分配较大的方法栈帧)栈深度：" + (stackOverFlow == null ? "null" : stackOverFlow.stackDeep));
        }

        try {
            recursiveReferenceWithBigStackFrame();
        }catch (StackOverflowError e){
            //e.printStackTrace();

            System.out.println("使用对象循环引用(分配较大的方法栈帧)栈深度：" + RecursiveReferenceWithBigStackFrame.referCount);
        }
    }
}
```
            
我们来看下测试代码，为了进行测试，我们使用了两种方式来产生StackOverFlowError，第一种是纯粹的递归调用。第二种是利用对象之间的循环引用。然后对于两种情况我们分别定义了没有局部变量和有局部变量的两种情况（对应较小栈帧和较大栈帧）。实际测试过程中我们使用的虚拟机参数分别为-Xss256k,-Xss1M以及不带参数-Xss，根据结果可以看出，当增大栈空间相应的栈帧数目即栈帧深度会变大（这个没有问题的，除法运算嘛）。大的栈帧相对于小栈帧在同等栈内存大小的情况下，较大栈帧调用只能得到较小的栈帧深度（也没有问题，同样是除法运算），此外，不带-Xss参数和带-Xss1M参数栈帧数目大抵相近（不知道为什么会有波动，而且即使内存相同，不同运行也会有不同的结果），这也验证了栈内存大小默认为1M。
     
```
java8\bin\java -Xss256k ...
使用方法循环调用栈深度：2994
使用对象循环引用栈深度：2486
使用方法循环调用(分配较大的方法栈帧)栈深度：1610
使用对象循环引用(分配较大的方法栈帧)栈深度：1609

java8\bin\java -Xss1M ...
使用方法循环调用栈深度：19360
使用对象循环引用栈深度：16078
使用方法循环调用(分配较大的方法栈帧)栈深度：10409
使用对象循环引用(分配较大的方法栈帧)栈深度：7380

java8\bin\java ...
使用方法循环调用栈深度：18243
使用对象循环引用栈深度：11414
使用方法循环调用(分配较大的方法栈帧)栈深度：10530
使用对象循环引用(分配较大的方法栈帧)栈深度：7386
```

##### 虚拟机栈内存溢出（基于线程实现）  
之前讨论过，单个线程下，无论是栈帧太大还是虚拟机栈容量太小，当内存无法分配的时候，虚拟机抛出的都是StackOverFlowError。

当换到多线程的环境下，通过不断创建新的线程，可以产生内存溢出异常而不是StackOverFlowError，但是这样产生内存溢出异常与栈空间大小并无关系。或者准确地说这种情况下，为每个线程的栈分配的内存越大，反而越容易产生内存溢出。
为什么呢，因为这已经不再是一个Java虚拟机栈上的栈帧的分配问题了，而是整个虚拟机中剩余内存（总内存减去堆内存再减去永久代MaxPermSize内存）上的分配问题了。这部分剩余内存被虚拟机栈和本地方法栈瓜分，本地方法栈我们不会讨论。总结来说，我们不再是讨论如何通过分配栈帧来消耗虚拟机栈最终得到OutOfMemoryError或者StackOverFlowError，而是通过不断的分配一个一个的虚拟机栈来消耗这部分所谓的剩余内存最终得到OutOfMemoryError。
   
```         
/**
 * VM Args:-Xss2M(这时候不妨设置大些)
 */
public class StackOutOfMemory {
    private boolean flag;

    private StackOutOfMemory() {
        this.flag = true;
    }

    private void stackMemoryLeakByThread() throws Exception {
        final CountDownLatchWrapper countDownLatchWrapper = new CountDownLatchWrapper();

        int threadCount = 0;
        try {
            while (flag) {
                Thread thread = new Thread(new Runnable() {
                    public void run() {
                        try {
                            synchronized (StackOutOfMemory.class) {
                                StackOutOfMemory.class.wait();
                            }
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                        countDownLatchWrapper.countDownLatch.countDown();
                    }
                });
                thread.start();

                threadCount++;
            }
        } catch (OutOfMemoryError outOfMemoryError) {
            outOfMemoryError.printStackTrace();

            countDownLatchWrapper.countDownLatch = new CountDownLatch(threadCount);

            this.flag = false;

            synchronized (StackOutOfMemory.class) {
                StackOutOfMemory.class.notifyAll();
            }
        }

        System.out.println("created:" + threadCount + " thread has been created! " + new SimpleDateFormat("mm:ss.SSS").format(new Date()));

        countDownLatchWrapper.countDownLatch.await();

        System.out.println("all thread has finished! " + new SimpleDateFormat("mm:ss.SSS").format(new Date()));
    }

    private static class CountDownLatchWrapper {
        private CountDownLatch countDownLatch;
    }

    public static void main(String[] args) throws Throwable {
        StackOutOfMemory stackOutOfMemory = new StackOutOfMemory();
        stackOutOfMemory.stackMemoryLeakByThread();
    }
}
```
                
代码依然很简单。不过有几个地方需要注意：
* 第一点，设置虚拟机参数-Xss2M，什么意思呢，就是将分配的虚拟机栈设置为2M。这样使用更少的线程就可以消耗完“剩余内存”
* 第二点，原书所附的源代码执行时会导致系统假死。因为为了保证分配的内存不被收集，需要new出来的子线程持续运行。最终虽然成功得到OutOfMemoryError异常，但是所有分配的内存都无法被回收，线程没有停止，每个线程占有最少2M的栈空间。最终其它应用程序因为无法分配内存也无法运行。所以本例中，利用Java的等待\通知机制，我们在new线程的同时让线程阻塞在StackOutOfMemory.class类实例上，当发生utOfMemoryError异常时调用StackOutOfMemory.class.notifyAll()方法唤醒线程。让线程结束，并释放掉占有的内存。
* 第三点，利用CountDownLatch，当所有线程都结束的时候，打印线程全部结束的消息。
* **第四点，如果您使用的是伟大的视窗操作系统，请不要尝试运行此段代码（具体原因参见原文：特别提示一下，请如果读者尝试运行上面的代码，记得要先保存当前的工作， 由于在windows平台的虚拟机中，Java的线程是映射到操作系统的内核线程上的，因此上述代码执行时有较大风险，可能导致系统假死），LZ运行了两次，idea均闪退，之后不久可能是内存被完全占用， 系统假死，只能强制重启，真是无fuck说，以下结果来自LinuxMint17.3 Idea2017**
         
```         
/usr/local/java/jdk1.8.0_73/bin/java -verbose:gc -XX:+PrintGCDetails -Xss2M 
java.lang.OutOfMemoryError: unable to create new native thread
at java.lang.Thread.start0(Native Method)
at java.lang.Thread.start(Thread.java:714)
at com.jvm.memorymgn.StackOutOfMemory.stackMemoryLeakByThread(StackOutOfMemory.java:37)
at com.jvm.memorymgn.StackOutOfMemory.main(StackOutOfMemory.java:66)

created:31804 thread has been created! 38:29.804
all thread has finished! 38:33.671

Heap
PSYoungGen      total 150016K, used 38709K [0x0000000719100000, 0x0000000723800000, 0x00000007c0000000)
eden space 129024K, 30% used [0x0000000719100000,0x000000071b6cd620,0x0000000720f00000)
from space 20992K, 0% used [0x0000000722380000,0x0000000722380000,0x0000000723800000)
to   space 20992K, 0% used [0x0000000720f00000,0x0000000720f00000,0x0000000722380000)
ParOldGen       total 343040K, used 0K [0x00000005cb200000, 0x00000005e0100000, 0x0000000719100000)
object space 343040K, 0% used [0x00000005cb200000,0x00000005cb200000,0x00000005e0100000)
Metaspace       used 3937K, capacity 4680K, committed 4864K, reserved 1056768K
class space    used 427K, capacity 432K, committed 512K, reserved 1048576K
```
         
##### 方法区&运行时常量池溢出  
之前也提打到过关于字符串常量的问题。本实例中使用的是String.intern()Api.不过由于LZ使用的java版本为1.8,而自1.7包括1.7之后Java虚拟机实现就把字符串常量池从永久代中移除了，所以并没有得到预期的异常，感兴趣得同学可以自行在1.6上面进行测试。

```
/**
 * VM Args:-XX:PermSize=10M-XX:MaxPermSize=10M  LZ测试环境为java8,无法发生溢出
 */
public class RuntimeConstantPoolOutOfMemory {
    public static void main(String[] args) {
        //使用List保持着常量池引用,避免Full GC回收常量池行为
        List<String> strPool = new ArrayList<String>();

        long byteCount = 0;

        try {
            //10MB的PermSize在integer范围内足够产生OOM了
            int i = 0;

            String str;
            while (true) {
                str = String.valueOf(i++);

                byteCount += str.length();

                strPool.add(str.intern());
            }
        }catch (OutOfMemoryError e){
            e.printStackTrace();

            System.out.println(String.format("共创建了%s个字符串，共%s字节",strPool.size(),byteCount));
        }
    }
}

//一个小实例，测试String.intern()以及StringBuilder
public static void showJavaMagic(){
    String str1 = new StringBuilder("计算机").append("软件").toString();
    System.out.println(str1.intern() == str1);
    String str2 = new StringBuilder("ja").append("va").toString();
    System.out.println(str2.intern() == str2);
}

//1.8版本结果
true
false
```
 
##### 直接内存溢出  
 
```
public class DirectMemoryOutOfMemory {
    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception {
        int totalMemoryAllocated = 0;
        try {
            Field unsafeField = Unsafe.class.getDeclaredFields()[0];
            unsafeField.setAccessible(true);
            Unsafe unsafe = (Unsafe) unsafeField.get(null);

            while (true) {
                unsafe.allocateMemory(_1MB);

                totalMemoryAllocated ++;
            }
        }catch (OutOfMemoryError e){
            e.printStackTrace();

            System.out.println("目前一共分配了" + totalMemoryAllocated + "MB的内存！");
        }
    }
}
```

* 程序运行结果

```
java.lang.OutOfMemoryError
at sun.misc.Unsafe.allocateMemory(Native Method)
at com.jvm.memorymgn.DirectMemoryOutOfMemory.main(DirectMemoryOutOfMemory.java:21)
Java HotSpot(TM) 64-Bit Server VM warning: Attempt to deallocate stack guard pages failed.
Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00007f0fa3d98000, 12288, 0) failed; error='Cannot allocate memory' (errno=12)
目前一共分配了2124252MB的内存！
\#
\# There is insufficient memory for the Java Runtime Environment to continue.
\# Native memory allocation (mmap) failed to map 12288 bytes for committing reserved memory.
\# An error report file with more information is saved as:
\# /home/icekredit/Documents/workplace/JvmTestCase/hs_err_pid9932.log
```


