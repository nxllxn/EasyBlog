---
title: What's QPS
tags:
     - Java
     - QPS
     - JMeter
     - JVisualVm
---

> 我听过，我知道;我看过，我记得;我做过，我懂了！

## 需求缘起
这一篇文章不是技术类的，可能并没有复杂的设计以及代码实现之类的，但是由于之前我自己老是搞不清楚QPS是什么怎么计算，并且在试图解决这个问题的时候遇到了一些新的问题，所以打算做个笔记，也把我兜圈子的过程给大家分享一下，如果刚好你也有这个问题并且我兜圈子的过程能给你一些提示，那再好不过。

**PS：多图，流量党慎入，wifi党、土豪请随意！**

<!--more-->

## 写在兜圈子之前
为什么说是兜圈子呢？首先呢是因为我知道QPS是什么，但是对具体有那些因素可能成为系统中QPS的瓶颈又有些模棱两可;其次，影响系统QPS的因素有很多，而且究竟是哪一个因素起着决定性的作用又和具体情景之间有着很微妙的关系，如果不能静下心来集中精力思考，思路很容易断掉。所以如果你和我一样有这个问题，那就烦请带着你的脑子我们去兜一圈。

## 兜圈子
### 什么是QPS
每秒查询率QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。

QPS，每秒查询速率，也可以说是系统的吞吐量。其实系统中很重要的一个性能指标，一般的业务系统还好，但是比如京东淘宝之内的电商系统，对于系统的吞吐量要求就比较高了。当然这些系统设计也更加复杂，今天我们只会举一些简单的例子来尽力搞明白QPS是什么，一个系统中影响QPS的因素有哪些，如果想要更深入的了解建议参考《亿级流量网站架构核心技术+跟开涛学搭建高可用高并发系统》。

### 举个栗子
在正式分析具体哪些因素会成为系统QPS的瓶颈之前，我们先来看一个例子。具体场景呢相信大家都很熟悉，那就是餐厅。

让我们来梳理一下一家餐厅大致有哪些需要注意的点。为了更直观，我们从客人的角度来分析。客人走到餐厅，这时一个负责接待的服务员上来为他带路，带他到对应的座位上坐下，于此同时这位负责接待的服务员召唤来另一名服务员，这位服务员负责点餐，然后通知厨师，厨师根据点餐订单，用对应的烹饪工具烹饪出对应的菜品，服务员上菜，客人食用，最终客人完成餐后支付离开餐厅。

此外我们做如下约定
* 一个服务员必须服务于客人进入餐厅到离开餐厅的整个时间段
* 这家餐厅的河豚烹饪做的非常好，所以一度成为这家餐厅的招牌菜，几乎所有食客都是慕名而来。
* 由于客户数目会是QPS的一个体现，所以我们假设有源源不断稳定的客户

那么在对整个系统做了足够的简化之后，我们需要关注的点有：
* 客人
* 负责接待的服务员
* 负责客人点餐、厨师调派、进餐、客人支付整个流程的服务员
* 一般厨师
* 主厨
* 座位
* 烹饪工具

#### 假设餐厅内的资源是无限制的
那么，有无限多的服务员来服务于无限多的客户，将无限多的客户带到无限多的座位上，无限多的客户产生的无限多的点餐订单由无限多的厨师利用无限多的烹饪工具进行烹饪。皆大欢喜，普天同庆。

#### 事实上

##### 不可能有无限多的服务员
首先老板不会招募无限多的服务员的，哈哈，开个玩笑！真实情况是，服务员也会占据一些资源，比如立体空间，过道等等，无限多的服务员只会让餐厅乱成一锅粥。所以一般服务员的数目是确定的，这里我们暂且前记作numberOfWaiter（单位：人），我们可以把服务员想象成一个组或者一个队列。有新的客人来的时候，从这个组或者队列中借一个服务员，当为这个客人服务完毕，这个服务员被归还到这个组中。
那么此时能够服务的客人的数目就是和客人用餐时间或者说服务员服务时间（暂且记为serviceTime，单位：小时）成反比了。也即服务客人数目=numberOfWaiter/serviceTime。

##### 不可能有无限多的座位
这个很好理解了，餐厅的空间是有限的嘛。假定餐厅中的座位数目为numberOfSeat（单位：个）。
那么即使服务员能够服务的客人数目numberOfWaiter/serviceTime大于numberOfSeat，实际能够服务的客人数目也不会超过numberOfSeat。

##### 不可能有无限多的厨师，或者说厨师不可能烹饪出无限多的菜肴
道理也很简单，厨房空间有限。能够容纳的厨师很有限，即使一个厨师同时掌勺，控制多个烹饪工具，但是总归是有一个上限的。这里我们假设单位时间厨师可以服务的菜肴数目是numberOfCustomerCanFeed（为了方便，我们以人为计量单位，单位：人）。
那么，即使服务员能够服务足够多的客人，且有足够的座位，实际能够服务的客人数目也不会超过numberOfCustomerCanFeed。

##### 不可能有无限多的烹饪工具
道理也很简单，厨房空间有限。这里我们假设单位时间烹饪工具最多能够产出的菜肴数目是numberOfCustomerCanApprove（同样，为了方便我们以人为计量单位，单位：人）;
那么即使有足够多的客人需要服务，有足够多的位置可以坐，有足够多的厨师来掌勺，实际能够服务的客人数目也不会超过numberOfCustomerCanApprove。

##### 只有一小部分或者一个主厨
只有厨艺精湛的主厨才能恰当地烹饪河豚，做这个假设应该不过分。也就是说，即使大多数客人慕名而来，但是，客人能够点到的河豚是很有限的。这里我们假定能够提供的河豚烹饪量为numberOfCustomerStupid(为了方便，同样以人为计量单位，单位：人)。
那么即使有足够多的客人需要服务，有足够多的位置可以坐，有足够多的厨师来掌勺，足够多的工具用于烹饪，实际能够服务的客人数目也不会超过numberOfCustomerStupid。

##### SO？
天哪！我就吃个饭咋这么麻烦呢！其实上面的描述还算是简单的，实际上可能根据客户需要，不同情景下，更加复杂，究竟在某一种情景下，具体是哪一种因素是整个系统的瓶颈，还是需要消耗一些脑细胞进行分析的。
比如某个菜肴制作过程十分复杂，那么numberOfCustomerCanFeed可能会急剧降低。如果用于客户用餐服务的服务员有几个请了假，或者每个客户的平均进餐时间变长了，numberOfWaiter/serviceTime也可能会受到很大影响。又比如某一台烹饪设施出了问题，某个主厨的妻子今天生小孩，都会对能够服务的客人数目造成影响。当然了这里说的是可能，毕竟最终的吞吐量是由最严重的瓶颈控制的。就像车道，吞吐量是由最窄的部分决定的，第二窄部分的波动可能并不会对整体吞吐量造成任何影响。

终于把这个栗子举完了，手真酸！不知道有没有讲明白！再举一个？算了还是不举了，不举了！

#### 从餐厅到业务系统
扯了这么多，究竟跟具体的业务系统有什么关系呢？来来来，让我们先看张美图。
![What's QPS Mind Graph](/images/qps/What's QPS.png)

看完图片大家应该明明白了，其实上面这个例子中客人数目就是图中的请求线程数目;服务员数目numberOfWaiter就是我们图中的服务线程数目;服务员服务时间serviceTime就是图中的后台任务执行时间，包括IO和CPU总时间;厨师呢则可以看成是CPU，座位则可以看作是系统内存，当然了也可以把负责接待的服务员看作是带宽。此外烹饪用的工具可以看作是软件资源限制，比如数据库连接数目，Redis连接数目，RServer连接数目等。即使厨师操作很快，也不能跳过烹饪工具的限制，即使CPU计算能力很强，也不能无视第三方连接的限制。此外，能够恰当地烹饪河豚的主厨就是图中的第三方依赖，整个餐厅的运转速度都会受限于这个主厨烹饪河豚的能力，同理一个系统的QPS不可能高于其依赖的第三方系统的QPS，我们可以倒过来思考，如果当前系统的QPS更高，那么当前系统就会有不断的请求累积起来，系统最终就挂掉了。当然这只是一个思考的方式，实际上多余的请求会被放置到队列中，如果队列满了，根据具体的线程池策略，可能在得到处理机会之前已经被丢掉了。

#### 实际的业务系统
* 首先是请求用户数目，之后的实战里面我们会用到Apache JMeter，其有一个线程组的概念，即在某一时间开启多少个线程，每一个线程循环调用指定接口，达到模拟的效果。这里再提一下，毕竟是模拟，而且是单机的，所以线程数目开太多的话最终结果和实际情况可能会有较大偏差，读者需要注意。
* 其次是服务线程数目，以SpringBoot内嵌的tomcat容器来说，其使用Java nio，实现高并发。其服务线程由线程池维护，默认核心线程数目是10个，线程池最多200个线程，注意，此数据不是文档给出，而是笔者在实际测试过程中根据JVisualVm工具观测得到的。什么意思呢，比如后台指定接口调用时间为1000ms，就是说一个线程一秒钟只能完成一次调用，那么此时系统的QPS最高只能到达200。之后会也有对应的测试案例供大家参考、验证。
* 后台任务执行时间，这里再次强调一下，这里执行时间时间包括CPU以及IO执行时间。也就是接口总执行时间
* 带宽，在每次调用传输的数据量较大时，此处很容易成为性能瓶颈，比如上传的测试数据是1M的Json字符串，服务器贷款是100mbps,那么实际的QPS可能最多只能达到100/8 = 12.5。
* 内存，这个和具体的业务代码有关了，如果业务代码执行期间需要大量的内存分配，此处可能成为QPS瓶颈，当然也可能是OutOfMemoryError，之后看能不能实际测试一下。
* CPU，当然还是和具体业务代码有关，比如之前听过关于跑机器学习的机器QPS只能达到2。如果有比较复杂的算法，此处可能成为瓶颈，之后我会做个简单的测试，在一次接口调用中进行1000000次的浮点数乘法运算来展示具体的效果。
* 软件资源限制或者说第三方依赖QPS，比如数据库连接、数据库一次调用时间，Redis连接、Redis一次调用时间，RServer连接、RServer一次调用时间等。比如数据库连接100个，如果一次查询时间为200ms，那么系统吞吐量最大最大可达到500;如果一次查询时间为20ms，那么系统吞吐量最大可达到5000;这也可以解释为什么一般生产环境Mysql（Mysql一般不是），Redis，RServer等都部署到同一个节点上面，这样可以得到非常低的调用时间。之后我会设计一个模拟测试。展示第三方依赖QPS是如何成为当前业务系统QPS瓶颈的。

#### 实战演练
在实战之前，我们先做一下简单的介绍，简单的补一下课。那就是关于Java性能测试相关的工具。怎么说呢，如果某个业务系统运行十分缓慢，我们就可以利用这些工具快速的定位问题点，甚至熟练了之后我们在进行系统设计实现的时候就能够有效的避免这些问题。

* [深入理解JVM----JDK的命令行工具](http://blog.csdn.net/ochangwen/article/details/52971913)
* [JMeter使用入门](http://www.cnblogs.com/ceshisanren/p/5639895.html) 

主要是两个工具，第一个是Apache JMeter用于性能测试的，非常非常强大的一个工具，也提供了很多插件供大家选择。第二个工具是JVisualVm，用于实时监测当前计算机上面运行的进程，可以提供CPU，内存，线程，堆等的具体状态监控。也十分强大，对于应用程序性能分析十分方便。

##### 最简单的接口调用
###### 代码
```java
    @RestController
    @RequestMapping("/qps")
    public class QPSController {
        private static final Logger logger = LoggerFactory.getLogger(QPSController.class);
    
        @PostMapping("/helloWithoutHesitation")
        public String helloWithoutHesitation() throws Exception{
            long startTimeInNano = System.nanoTime();
    
            logger.info("线程名称：{}，线程ID：{},请求时间：{}",
                    Thread.currentThread().getName(),
                    Thread.currentThread().getId(),
                    System.nanoTime() - startTimeInNano);
    
            return "I am fine!Thank you!and you?";
        }
    }
```
可以看出，在这个接口中我们没有添加任何的业务逻辑。只是简单的打印了一条日志，并返回了一个结果。
###### 测试JMeter && JVisualVM
![JMeter 简单接口调用](/images/qps/helloWithoutHesitation.png)</br>
上图为接口调用相关配置，非常简单，这里不做额外解释。

* 系统刚启动,应用程序线程，cpu，内存分配情况
![JVisualVM 系统启动CPU、内存](/images/qps/start_cpu_memory.png)
![JVisualVM 系统启动线程](/images/qps/start_thread.png)

可以看出，当JVM进程启动的时候，SpringBoot自带的tomcat容器初始化了一个大小为10的服务线程线程池。这个可以理解为为线程池核心线程数目。当有更快速的请求被提交时，线程池会进一步扩充，根据测试结果，线程池默认的最大大小为200,从之后的测试用例中也可以看出来。当客户端请求速度进一步提升时，线程池已经无法进一步增长了，这时“多余的”请求会被放置到队列中，如果请求速度进一步增加，那么当队列满了之后会根据具体的线程池策略进行处理,比如直接丢弃当前请求等。
更多关于tomcat线程池相关的读者可以自行百度。下面是两个相关的资料。
[Tomcat线程池策略](https://segmentfault.com/a/1190000008052008)
[Tomcat的maxThreads、acceptCount（最大线程数、最大排队数）](http://blog.csdn.net/qq_26217487/article/details/51735803)

* 线程组只有一个线程
![JMeter 线程组设置一个线程](/images/qps/JMeterWithOneThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/001.png)</br>
![测试结果JVisualVM Thread](/images/qps/002.png)</br>
![测试结果JMeter](/images/qps/simple_1_thread.png)

做一下简单的分析：
图1展示了JMeter线程组设置，可以看出我们只启动了一个线程，这个线程会持续运行60秒钟。
图2展示了程序运行期间CPU消耗情况以及内存分配情况。
图3展示了程序运行期间，当前应用程序内线程运行情况，可以看出，来自客户端的请求分布在了这10个核心线程上。
图4展示了JMeter测试结果，可以看出整个测试期间一共发送了16070个请求，系统当前的吞吐量为267.8/s

* 线程组有10个线程
![JMeter 线程组设置10个线程](/images/qps/JMeterWithTenThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/101.png)</br>
![测试结果JVisualVM Thread](/images/qps/102.png)</br>
![测试结果JMeter](/images/qps/simple_10_thread.png)
做一下简单的分析：
图1展示了JMeter线程组设置，可以看出我们计划启动10个线程，刚开始立即启动两个，之后每一秒启动两个线程，启动完成后整个线程组持续运行60秒，最后每一秒关闭两个线程。整个测试计划会运行68秒。
图2展示了程序运行期间CPU消耗情况以及内存分配情况。
图3展示了程序运行期间，当前应用程序内线程运行情况，可以看出，来自客户端的请求分布在了这10个核心线程上。（其实图截的有点问题，其实服务线程会比请求线程数目稍微大一些，实际测试时图片下方还有一个新建的线程11）
图4展示了JMeter测试结果，可以看到当前测得系统吞吐量有了很大的提升。那么为什么要更高呢，那就是因为单个线程虽说持续运行，但是他的请求提交能力是有限的，不足以产生足够的请求速度。

* 线程组有100个线程
![JMeter 线程组设置100个线程](/images/qps/JMeterWithOneHundredThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/201.png)</br>
![测试结果JVisualVM Thread](/images/qps/202.png)</br>
![测试结果JMeter](/images/qps/simple_100_thread.png)
做一下简单的分析：
图1展示了JMeter线程组设置，可以看出我们计划启动100个线程。具体设置参考第二步的设置，此处不再赘述。
图2展示了程序运行期间CPU消耗情况以及内存分配情况。从右下角也可以大致看出，tomcat容器内创建了更多的服务线程，100多一点的样子。
图3展示了程序运行期间，当前应用程序内线程运行情况。可以看出请求分布在了这100个线程上（其实图截的有点问题，其实服务线程会比请求线程数目稍微大一些，实际测试时应该有101个服务线程）
图4展示了JMeter测试结果，可以看出相较于第二步，系统吞吐量又有了一些提升。

* 线程组有200个线程
![JMeter 线程组设置200个线程](/images/qps/JMeterWithTwoHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/301.png)</br>
![测试结果JVisualVM Thread](/images/qps/302.png)</br>
![测试结果JMeter](/images/qps/simple_200_thread.png)
做一下简单分析：
图1展示了JMeter线程组设置，可以看出我们计划启动200个线程。具体设置参考第二步的设置，此处不再赘述。
图2展示了程序运行期间CPU消耗情况以及内存分配情况。
图3展示了程序运行期间，当前应用程序内线程运行情况。可以看出系统一共创建了200个线程（高亮的那一行线程名http-nio-8080-200）。
图4展示了JMeter测试结果，可以看出系统吞吐量有了很大的提升。

* 线程组有500个线程
![JMeter 线程组设置500个线程](/images/qps/JMeterWithFiveHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/401.png)</br>
![测试结果JVisualVM Thread](/images/qps/402.png)</br>
![测试结果JMeter](/images/qps/simple_500_thread.png)
做一下简单分析：
图1展示了JMeter线程组设置，可以看出我们计划启动500个线程。具体设置参考第二步的设置，此处不再赘述。
图2展示了程序运行期间CPU消耗情况以及内存分配情况。
图3展示了程序运行期间，当前应用程序内线程运行情况。可以看出系统还是一共创建了200个线程（高亮的那一行线程名http-nio-8080-200）。这也说明tomcat默认的maxThreads为200
图4展示了程序运行结果，可以看出系统的吞吐量反而降低了。此处需要注意一下，我觉得此处吞吐量降低可能是因为测试工具里面开的线程数目太多，所以由于测试工具所在计算机忙着调度去了，没有全心全意的专注于发请求。其实后台并没有达到QPS极限，毕竟第四步就已经证明系统最大QPS至少比第五步的高，就是说系统处理能力是没有消耗完的。

* 总结
上面几个测试中我们大致过了一下：1）应用容器请求处理的模块的池化策略。2)JMeter,JVisualVM工具的使用

##### 测试较高的请求处理时间
此测试中我们会控制请求处理时间，通过Thread.sleep()来模拟。我们会让线程休眠一秒钟，来看测试结果。

###### 具体代码

```java
    @RestController
    @RequestMapping("/qps")
    public class QPSController {
        private static final Logger logger = LoggerFactory.getLogger(QPSController.class);
        
        @PostMapping("/helloWithHesitation1000Millis")
        public String helloWithHesitation1000Millis() throws Exception{
            long startTimeInNano = System.nanoTime();
    
            //模拟比较长的操作，比如io，复杂的数据计算等
            Thread.sleep(1000);
    
            logger.info("线程名称：{}，线程ID：{},请求时间：{}",
                    Thread.currentThread().getName(),
                    Thread.currentThread().getId(),
                    System.nanoTime() - startTimeInNano);
    
            return "I am fine!Thank you!and you?";
        }
    }
```

###### 测试JMeter && JVisualVM
![JMeter 高服务调用时间](/images/qps/helloWithHesitation1000Millis.png)</br>
上图为接口调用相关配置，非常简单，这里不做额外解释。

* 线程组有100个线程
![JMeter 线程组设置100个线程](/images/qps/JMeterWithOneHundredThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/501.png)</br>
![测试结果JVisualVM Thread](/images/qps/502.png)</br>
![测试结果JMeter](/images/qps/start_100_hesitation_1000_result.png)
做一下简单分析：
图1展示了JMeter线程组设置，可以看出我们计划启动100个线程。
图2展示了程序运行期间CPU消耗情况以及内存分配情况。
图3展示了程序运行期间，当前应用程序内线程运行情况。看出为了服务这100个客户端提交的请求，tomcat容器一共创建了了101个线程。
图4展示了JMeter测试结果，可以看出系统整体的吞吐量只有83。就是说101个线程在每个接口处理时间大概为1秒或者说单位时间的情况下能够产出的QPS是小于服务线程总数目的。这也验证了之前提到的，当加上了池化策略时，系统的QPS和每个请求处理时间是成反比的。

* 线程组有200个线程
![JMeter 线程组设置200个线程](/images/qps/JMeterWithTwoHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/601.png)</br>
![测试结果JVisualVM Thread](/images/qps/602.png)</br>
![测试结果JMeter](/images/qps/start_200_hesitation_1000_result.png)
做一下简单分析：
此测试同上一个测试用例大致相同。结果也是一致的。再一次验证，当加上了池化策略时，系统的QPS和每个请求处理时间是成反比的。

##### 测试复杂的CPU计算
此测试中我们在请求接口中执行1000000次浮点数计算，来模拟复杂的CPU计算。

###### 具体代码

```java
    @RestController
    @RequestMapping("/qps")
    public class QPSController {
        private static final Logger logger = LoggerFactory.getLogger(QPSController.class);
    
        @PostMapping("/helloWithComplicateCalculation")
        public String helloWithComplicateCalculation() throws Exception{
            long startTimeInNano = System.nanoTime();
    
            //模拟比较复杂的cpu计算
            double neverUsedValue = 0D;
            for (int index = 0;index < 1000000;index ++){
                neverUsedValue *= index;
            }
            logger.info("result:" + neverUsedValue);
    
            logger.info("线程名称：{}，线程ID：{},请求时间：{}",
                    Thread.currentThread().getName(),
                    Thread.currentThread().getId(),
                    System.nanoTime() - startTimeInNano);
    
            return "I am fine!Thank you!and you?";
        }
    }
```

###### 测试JMeter && JVisualVM
![JMeter 复杂的CPU计算](/images/qps/helloWithComplicateCalculation.png)</br>
上图为接口调用相关配置，非常简单，这里不做额外解释。

* 线程组有1个线程
![JMeter 线程组设置1个线程](/images/qps/JMeterWithOneThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/701.png)</br>
![测试结果JVisualVM Thread](/images/qps/702.png)</br>
![计算机CPU使用情况](/images/qps/703.png)</br>
![测试结果JMeter](/images/qps/start_1_for_complicate_result.png)

* 线程组有10个线程
![JMeter 线程组设置10个线程](/images/qps/JMeterWithTenThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/801.png)</br>
![测试结果JVisualVM Thread](/images/qps/802.png)</br>
![计算机CPU使用情况](/images/qps/803.png)</br>
![测试结果JMeter](/images/qps/start_10_for_complicate_result.png)

* 线程组有20个线程
![JMeter 线程组设置20个线程](/images/qps/JMeterWithTwentyThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/901.png)</br>
![测试结果JVisualVM Thread](/images/qps/902.png)</br>
![计算机CPU使用情况](/images/qps/903.png)</br>
![测试结果JMeter](/images/qps/start_20_for_complicate_result.png)

* 线程组有50个线程
![JMeter 线程组设置50个线程](/images/qps/JMeterWithFiftyThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/1001.png)</br>
![测试结果JVisualVM Thread](/images/qps/1002.png)</br>
![计算机CPU使用情况](/images/qps/1003.png)</br>
![测试结果JMeter](/images/qps/start_50_for_complicate_result.png)

* 线程组有100个线程
![JMeter 线程组设置100个线程](/images/qps/JMeterWithOneHundredThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/1101.png)</br>
![测试结果JVisualVM Thread](/images/qps/1202.png)</br>
![计算机CPU使用情况](/images/qps/1203.png)</br>
![测试结果JMeter](/images/qps/start_100_for_complicate_result.png)

* 线程组有200个线程
![JMeter 线程组设置200个线程](/images/qps/JMeterWithTwoHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/1301.png)</br>
![测试结果JVisualVM Thread](/images/qps/1302.png)</br>
![计算机CPU使用情况](/images/qps/1303.png)</br>
![测试结果JMeter](/images/qps/start_200_for_complicate_result.png)

* 线程组有500个线程
![JMeter 线程组设置500个线程](/images/qps/JMeterWithFiveHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/1401.png)</br>
![测试结果JVisualVM Thread](/images/qps/1402.png)</br>
![计算机CPU使用情况](/images/qps/1403.png)</br>
![测试结果JMeter](/images/qps/start_500_for_complicate_result.png)

上面几组测试用例分别展示了客户端请求线程数目为1,10,20,50,100,200,500时调用包含复杂CPU计算的接口时系统可能达到的吞吐量结果。可以看出当请求增加，很快就能够消耗掉CPU全部的计算能力，此时CPU的计算能力成为了系统QPS的一个瓶颈。

##### 测试第三方依赖QPS
此测试中我们在请求接口中会模拟一个第三方依赖调用。同样也是利用线程池。现今许多地方都会有池化技术，比如数据库连接池，DBCP，Druid等，Redis连接池，Jedis等，RServer连接池，如RUtils内部实现的RConnectionPool。
我们通过控制连接池中线程执行时间来控制第三方依赖QPS。比如大小为10的固定线程池，当一次调用时间为200ms，这个连接池的QPS为10 / （200D / 1000） = 50。当一次调用时间为100ms，这个连接池的QPS为10 / （100D / 1000） = 100。

###### 具体代码

```java
@RestController
@RequestMapping("/qps")
public class QPSController {
    private static final Logger logger = LoggerFactory.getLogger(QPSController.class);

    private static final ExecutorService thirdDependenciesPool = Executors.newFixedThreadPool(10);
    private Callable<String> thirdDependencyCallable = new Callable<String>() {
        @Override
        public String call() throws Exception {
            try {
                //Thread.sleep(100);
                Thread.sleep(200);
            }catch (InterruptedException e){
                logger.error("线程中断！" );
            }

            return "I am fine!Thank you!and you?";
        }
    };
    @PostMapping("/helloWithThirdDependenciesLimit")
    public String helloWithThirdDependenciesLimit() throws Exception{
        long startTimeInNano = System.nanoTime();

        String responseStr = thirdDependenciesPool.submit(thirdDependencyCallable).get();

        logger.info("线程名称：{}，线程ID：{},请求时间：{}",
                Thread.currentThread().getName(),
                Thread.currentThread().getId(),
                System.nanoTime() - startTimeInNano);

        return responseStr;
    }
}
```

上面已经对思路做了具体阐述，代码也相对比较简单，此处不再赘述。

###### 测试JMeter && JVisualVM
此处我们就不提供复杂的测试用例了，为了做区分，这里统一使用JMeter开启200个线程进行测试
* 一次调用时间为200ms
![JMeter 线程组设置200个线程](/images/qps/JMeterWithTwoHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/1501.png)</br>
![测试结果JVisualVM Thread](/images/qps/1502.png)</br>
![测试结果JMeter](/images/qps/third_qps_50.png)

* 一次调用时间为100ms
![JMeter 线程组设置200个线程](/images/qps/JMeterWithTwoHundredsThread.png)</br>
![测试结果JVisualVM Cpu Memory](/images/qps/1601.png)</br>
![测试结果JVisualVM Thread](/images/qps/1602.png)</br>
![测试结果JMeter](/images/qps/third_qps_100.png)

可以看出，第一个测试用例QPS为49,稍微低于模拟的第三方依赖QPS 50,第二个测试用例QPS为91,稍微低于模拟的第三方依赖QPS 100。

#### 总结
从上面各个测试用例可以大致看出之前提到的这些因素具体是如何影响系统最终的吞吐量的。到了具体分析系统QPS时，只需要对这些关键因素进行分析，再结合具体的系统部署场景便可以快速的定位到系统QPS瓶颈所在。