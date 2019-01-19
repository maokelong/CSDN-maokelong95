# Memory Models Series - Memory Consistency (Slides)

> **日志**：
>
> 1. [2018-06-19] 完成了本文的 PPT 框架及除 TSO 外的全部文字说明。等闲了再把说明补全。

---

> **作者按**：内存模型系列（上）- 内存一致性模型。本文为内存模型系列上篇，主要深入浅出地介绍了用于描述访存顺序及访存原子性的一致性模型。本文主要面对对象为刚接触并发编程的编程者，为了方便读者理解，本文将尽量采用偏口语的文风，并尽量避免有关硬件实现的部分。其实后半部分没法谈也不需要谈吧，这个话题太大了，而且作为一名编程者，你只需要用把内存模型当作一个工具来用就可以了。
>
> 本系列的下篇为：[内存模型系列（下）- 内存持久性模型（Memory Persistency）](https://blog.csdn.net/maokelong95/article/details/81199226)。

---

[toc]

## 1. 简介

首先介绍一下本文的大纲。本文将尝试性地采用讲 PPT 的形式展开。PPT 的主要内容共计 20 页，其中 5 页讲了我们为什么需要了解（内存）一致性模型，5 页讲了最严格的一致性模型——顺序一致性模型，10 页以 x86 的 TSO 为例讲了较为宽松的一致性模型，最后 1 页列出了我所参考的文献。本文中所有图片相应的介绍都在图片上方。

![幻灯片1](https://img-blog.csdn.net/201806190124292?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

在谈到并发编程的时候，我们主要有两种编程范式：消息传递及共享内存。前者主要用于分布式系统，通过网络，采用消息的发送及接收的方式完成对共享数据的访问；而后者主要用于单机，通过内部互联总线，采用读写共享内存的方式完成对共享数据的访问。一致性模型为后者服务。

![幻灯片2](https://img-blog.csdn.net/20180619012441677?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

下面考虑一段很简单的程序。下图中，三个处理器（P1、P2、P3）依次执行三段程序片段。在执行虚线下部分代码之前，三个处理器**共享**三个初始化为零的变量，ABC。按照你的理解，下例中可能输出的结果是什么呢？

你很可能会说，A == 1 && C == 1。是的，这是最符合直觉的结果。但别的结果，包括 A == 0 && C == 1、A == 1 && C == 0 及 A == 1 && C == 0 都是有可能的。这是怎么回事呢？

以 A == 0 && C == 1 为例，当 P1 上的赋值语句 C = 1 及 A = 1 完成之后，P2 **看见** P1 对 A 的修改，从而跳出循环并执行 B = 1，接下来 P3 **看见** P2 对 B 的修改，从而跳出循环并打印 AC。在打印 AC 时，存在 P3 **看见** P1 对 C 的修改，而**未看见** P1 对 A 的修改，从而打印出 A == 0 && C == 1。

简而言之，其中的坑在于，你以为 C = 1 执行完之后，所有处理器都应该看到 C 的最新值，但实际可能不是这样。

> **注意**： 本文会使用「看见」这个词。当一个处理器（PA）对共享变量（V）的本地副本进行了修改，而另一个处理器（PB）用于保存相应变量的缓存行因此而失效或被更新，则本文称 PB 看见了 PA 对变量 V 的修改。

![幻灯片3](https://img-blog.csdn.net/20180619012508564?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

所以到底是谁规定着，处理器能否在一个变量为给定处理器可见之前继续执行呢？就是今天我们要介绍的主角，**一致性模型**。

![幻灯片4](https://img-blog.csdn.net/20180619012517416?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

首先，让我们来看看一致性模型的定义。所谓**一致性模型**，也就是一个用来规定新值啥时候传播到某一给定处理器的策略。最早 \ 最晚是啥时候嘛，给句亮堂话。这个定义看起来有点像**缓存一致性协议**，这个协议就是一个用来将修改过的缓存副本传播给别的缓存的算法。

俩兄弟看着挺像的，但前者确定的是何时，而后者确定的是如何。另外，还有些系统，虽然它们没有狭义上的缓存，但它们也存在一致性问题。

![幻灯片5](https://img-blog.csdn.net/20180619012534232?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

比如，给定一种系统，它在每个处理器内部都设置一块写缓冲区，用以吸收写指令。后续本处理器在读取内存的时候，如果检测到写缓冲区里已经存在针对同一地址的写了，那么它就直接解析这个写指令要写入的值并返回。显然，这样的系统的优点在于，它能够很好地增加访存指令吞吐量。

如果这样的设计放在单处理器系统上，那么一点问题都没有，这很像处理器设计里常见的重定向设计。但是，在多核系统上就麻烦了。处理器可能会从内存里读到过期的数据，因为相关的写操作可能仍然缓冲在缓冲区里而未执行。也就是说别的处理器可能在一个处理器的写操作可见之前读取了过期的数据，这就是一致性问题嘛。

换作采用内部互联总线的系统可能也存在类似的问题，详情请参考文献[1]，无非是之前数据缓冲在了缓冲区，而现在缓冲在了寄存器上罢了，总之都造成了不同的处理器看见不一致数据的现实情况。

> **注意**：这一段纯属考古，感兴趣就看看，没兴趣完全可以跳过。另外本文中的访存（Memory Access）指令包括读存指令（Load）及写存指令(Store)。

![幻灯片6](https://img-blog.csdn.net/20180619012547278?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2. 顺序一致性

一致性模型里最严苛的就是顺序一致性了，它的定义很绕口：我们称一个多处理器系统是「顺序一致的」，当该多处理器系统的运行结果看起来好像所有处理器的操作按照某种线性顺序执行，且在该线性顺序中，各处理器的操作还遵循程序指定顺序。

当然，你们勤劳的博主是不会把这么一大段难消化的东西直接丢给你们的，这样也太失败了。其实大家抓住两个关键词就可以了：**程序顺序**和**原子性**。

![幻灯片7](https://img-blog.csdn.net/20180619012556535?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

所谓程序顺序，也就是说各处理器的操作保持程序指定的顺序。

为什么要约束到程序顺序？以下图程序为例，开发者原本期望通过 Flag1 和 Flag2 来控制两段程序中的临界代码最多只有一个可以执行，但是，如果处理器 / 编译器对执行的执行顺序进行了调度，使得 Flag1 和 Flag2 的执行被挪到了临界代码之后，那这样一来两段临界代码都会执行，从而违背了开发者的期望。

所以这和一致性有什么关系吗？如果一条访存指令都被挪到别的地方去了，那么你就可以理解为，甚至连本处理器都将延迟看到本条指令的结果。翻到上面去看看一致性模型的定义，是不是觉得就明白了？

![幻灯片8](https://img-blog.csdn.net/20180619012606784?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

所谓原子性，也就是说各处理器能够同时看到访存指令。

为什么要约束原子性？这是很显然的事情。拿这个曾经讲过的例子来说，如果 A = 1 同时被 P2 和 P3 看见，那么 register1 的值就不可能是 0 了，因为 register1 能被赋值说明所有处理器都看见 B = 1，说明所有处理器都看见 A = 1。P3 总不能装瞎不是？

![幻灯片9](https://img-blog.csdn.net/20180619012615235?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

虽然开发者可能很喜欢顺序模型，因为真的很容易理解，但编译器就不开心了。以前我能用的那些基于重排序的优化都不能用了啊！我的代码移动啊！我的寄存器染色啊！我的子表达式消除啊！我的别的骚操作啊！统统都凉了啊！

![幻灯片10](https://img-blog.csdn.net/20180619012624183?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 3. 宽松一致性

所以说把顺序一致性的要求放低一点，让编程者有更多知识学，让指令优化有更多空间折腾嘛。比顺序一致性更宽松的内存模型就称之为宽松一致性。

下图是一张考古图，里面放了各大著名内存模型的特征。比较新的可以移步维基百科「[Memory Ordering](https://en.wikipedia.org/wiki/Memory_ordering)」。实用着想，这里主要以 Intel 所采用的 TSO 模型为例展开。

图表中标题的含义将在后面介绍。

> **注意**：1. 最早是 SPARC 采用的 TSO，只是现在 Intel x86 更出名一些；2. 在维基百科上你能看到「x86 oostore」，不用管它，这鬼东西已经灭亡了。

![幻灯片11](https://img-blog.csdn.net/20180619012633544?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

什么叫做 TSO 呢。当各处理器看见所有写存指令的执行顺序是一样的时候，我们称这些写存指令是 **TSO** 的。我不喜欢直接丢定义，所以接下来我会很结合大量例子详细地介绍 TSO 的特性。恩，特性而非定义，才是大家最喜闻乐见的。

在具体讲解之前，先给大家打一针预防针。前面我们也谈到了，顺序模型主要从**程序顺序**及**原子性**两个维度上约束了新值为任一处理器可见的时机。自然而然地，我们放松一致性模型时也会从这两方面入手。对应到上张图中的表，2~4 列就是用来描述是如何放松程序顺序的，5~6 列就是用来描述如何放松原子性的。那么放松的一致性模型怎么再变得严格起来呢？所预留的手段就称之为安全网（Safety Net）。

另外你应该会很困惑「→」是个什么鬼，其实这就是 Happened-before 关系的标识，通俗地理解就是，当 A→B 时，B 必须发生在 A 之后。

![幻灯片12](https://img-blog.csdn.net/20180619012642311?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片13](https://img-blog.csdn.net/20180619012651880?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片14](https://img-blog.csdn.net/20180619012659842?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片15](https://img-blog.csdn.net/20180619012706892?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片16](https://img-blog.csdn.net/20180619012714796?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片17](https://img-blog.csdn.net/20180619012721403?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片18](https://img-blog.csdn.net/20180619012728274?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片19](https://img-blog.csdn.net/20180619012738496?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![幻灯片20](https://img-blog.csdn.net/20180619012746179?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 参考文献

下图给出了本文主要参考的文献，其中本文的内容框架启发自第一篇文献，本文中有关 TSO 的介绍主要抠自第二篇文献，本文中有关锁指令的描述主要抠自第三篇文献。

![21](https://img-blog.csdn.net/20180619012757245?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
