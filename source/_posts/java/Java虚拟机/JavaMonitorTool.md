---
title: Java虚拟机监控与故障处理工具
tags:
     - Java
     - Java虚拟机
     - JVM
     - 虚拟机监控
     - 故障处理工具
---

> Java与C++之前有一堵内存动态分配和垃圾收集技术所围成的高墙，墙外面的人想进去，墙里面的人却想出来

## 概述

前面也提到过，Java有自己的垃圾收集技术，开发人员不需要花费太多的精力来考虑内存的分配与回收，对于开发人员的门槛也降低了很多很多。但是当得到这些“好处”的同时，也必须付出一定的代价，那就是一旦系统出了问题，如果你不清楚底层的实现细节，不知道该用什么有效的工具来定位问题，那么你将无法从错误中快速回复过来。

> 她那时还太年轻，不知道所有命运赠送的礼物早已在暗中标上价格。--茨威格《断头皇后》

理论总是作为指导实践的工具，能够把这些只是应用到实际工作中才是我们的最终目的。

给一个系统定位问题的时候，知识、经验是关键基础，数据是依据，工具则是运用知识处理数据的手段。什么是知识经验呢，那就是对于CPU，内存，带宽，硬盘等硬件资源限制以及诸如线程，数据库连接，Redis连接，其他系统临界资源等软件资源限制具体有哪些，在某一个具体场景中究竟是哪一个因素导致了系统运行缓慢或者崩溃。什么是数据呢，那就是系统某一个时刻的运行情况的快照，狭义上可以是一个简单的堆存储快照，而广义上就是某个时间点与程序运行相关的任意资源的分配情况的快照，有了这些关联的事实依据，我们才能去尝试定位问题。而工具是运用知识经验处理数据的手段又是指什么意思呢，那就是利用一些有效的工具我们在对资源分配情况做分析是将更加高效，更容易定位问题。

## JDK的命令行工具
命令行工具大家应该不陌生了，一般做JAVA开发的人员最开始在没有使用任何IDE的情况下，一般都至少使用过javac、javap以及java这几个工具，javac用于将Java源代码编译为字节码.class文件,javap是jdk自带的反编译工具，java呢则负责执行编译好的字节码文件或者打包好的jar包。本文不会讨论如何使用这些工具，这些工具和性能监控或者故障处理都没有太大关系。
![JDK全部命令行工具](images/monitor/all_command_line_tools.png)
上图是JDK1.8自带的全部命令行工具，下面会就其中几个工具进行简要讨论

### JPS--虚拟机进程状况工具
jps（JVM process status tool）虚拟机进程状态工具，JDK很多小工具的名称都参考了UNIX命令的命名方式，jps命令除了名字长的像UNIX的ps命令之外，他的功能也和ps命令十分相似，可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（MainClass，main方法所在类）的名称，以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier，LVMID）。虽然功能比较简单，但是这是一个使用频率最高的命令，毕竟想要一探究竟，必须有一个入口，也就是说其他的JDK工具大多需要一个LVMID来确定需要监控的是哪一个进程。此外，对于本地虚拟机进程来说，LVMID与操作系统的进程ID是一致的，那么也就是说直接使用ps命令也可以找到对应的进程Id，不过这肯定是没有主类名称直观的，当然了你也可以使用kill命令杀掉指定LVMID对应的进程。

jps命令格式：jps \[options] \[hostid]
jps常用选项：
* -q quiet 只输出LVMID，不输出主类名称
* -m 输出虚拟机启动时传递给主类的参数
* -l 输出类的全类名如果执行的是jar包，输出执行jar包的路径
* -v verbose 输出虚拟机进程启动时jvm参数

Jps也可以使用RMI协议查询开启了RMI协议的远程虚拟机的进程状态

### JSTAT--虚拟机统计信息监视工具
jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。包括类装载、内存分配、垃圾收集等。对于没有GUI，只提供了纯文本控制台环境的服务器，它将是排查系统问题的首选。

jstat命令格式：jstat [option vmid[interval[s|ms][count]]]
jstat常用选项：
* -class 监视类装载、卸载数量，总空间以及装载所消耗的时间
* -gc 监视java堆情况，包括Eden区，两个Survivor区，老年代，永久带等的容量，已用空间，GC时间等信息
* -gccapacity 同-gc，只是输出主要关注各个区域使用的最小、最大空间 
* -gcutil 同-gc，只是输出主要关注各个区域使用的比例
* -gccause 同-gc，只是会额外输出导致上一次gc的原因
* -gcnew 监视新生代gc情况
* -gcold 监视老年代gc情况
* -compiler 输出JIT编译器编译过的方法耗时等信息

参数interval和count代表查询间隔和次数，如果省略这两个参数说明只查询一次。假设需要每1000ms查询一次进程9527的垃圾收集情况，一共查询60次，那么具体命令为：`jatat -gc 9527 1000 60`。

此外需要说明一下上面的LVMID和VMID，如果是本地虚拟机进程，VMID与LVMID一致，如果是远程虚拟机进程，那么VMID的格式应该为`[protocol:][//]lvmid[@hostname[:port]/servername]`

### JINFO--Java配置信息工具
jinfo（Configuration Info For Java）的作用是实时的查看和调整虚拟机各项参数。之前提到过，使用jpd -v可以查看JVM启动时显式指定的参数列表，但是如果想要查看未被显式指定的参数，则需要通过jinfo的-flag选项来查询了。

jinfo命令格式：jinfo [options] LVMID
jstat常用选项:
* -flag 指定需要查看的指定flag

### JMAP--Java内存映像工具
jmap（Memory Map for Java）命令用于生成堆存储快照，一般称之为heap dump或者dump heap。之前也提到过通过设置JVM启动参数-XX:+HeapDumpOnOutOfMemoryError来控制当程序出现内存溢出时dump出当前的堆内存快照。
jmap的作用并不仅仅是为了获取dump文件，它还可以查询finalize执行队列，Java堆和永久代的详细信息，比如空间使用率以及目前使用什么类型的垃圾收集器等。

jmap命令格式：jmap [option] vmid
jmap常用选项：
* -dump 生成堆转储快照
* -finalizerinfo 显示需要执行finalize方法的对象
* -heap 显示Java堆的详细信息
* -histo 显示堆中对象统计信息，包括类、实例数量，合计容量等。
* -permstat 显示永久带内存情况
* -F 当虚拟机进程对-dump选项没有响应时，使用此选项强制生成堆转储快照

### JHAT--虚拟机转储快照分析工具
jhat（JVM Heap Analysis Tool）一般用来配合jmap工具使用。用来分析Jmap生生成的dump文件。jhat内部内置了一个微型的Http/Html服务器。生成dump文件分析结果之后，可以直接在浏览器中查看。不过进行dump文件分析一般比较耗时且比较依赖硬件资源，一般不会在生产环境做这件事情。此外，jhat能够提供的分析结果是很简陋的，使用起来也并不是很方便。

### JSTACK--Java堆栈跟踪工具
jstack（stack trace for Java）命令用于生成当前时刻的虚拟机线程快照（一般称之为threaddump或者javacore）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，主要被用来定位线程出现长时间停顿的原因，比如线程死锁，阻塞等。通过分析线程堆栈就可以知道线程具体在做什么事情或者在等待什么资源。

jstack命令格式：jstat [option] vmid
jstack常用选项：
* -F 当请求不被响应时强制输出线程堆栈
* -l 除堆栈外，显示关于锁的附加信息

### JCONSOLE--Java监视与管理控制台
JConsole（Java Monitoring and Management Console）是一种给予JMX的可视化监视管理工具。

#### 启动
！[启动JConsole](/images/monitor/start_jconsole.png)
启动JConsole之后，从途中可以看出，存在两个idea相关的Java进程以及一个JConsole自己的进程。

![idea JConsole 视图](/images/monitor/idea_jconsole.png)
上图为选中idea主类所在进程后，JConsole控制面板的布局情况，可以看出展示了包括内存，CPU，线程，类，MBean在内的诸多很详细的信息。

#### 实战演练
##### 内存监控
内存面板所展示的信息相当于可视化的jstat命令，用于展示虚拟机内存分配以及回收情况。我们将使用一段测试代码来体验一下它的功能。

程序大体设计如下：
* 以64KB/50ms的速度向Java堆中填充数据，循环填充1000次

程序运行时我们会设置相关的虚拟机参数，主要包括：
* -Xms100m -Xmx100m 避免堆扩展（将堆的最小值-Xms参数与最大参数-Xmx设置为一样即可避免堆自动扩展）
* -XX:+UseSerialGC 设置使用的垃圾收集器的类型
* -verbose:gc -XX:+PrintGCDetails 由于JConsole试图中没有提供详细的时间戳信息，所以我们加上这两个参数打印出详细的gc信息

###### 具体代码
```java

```
