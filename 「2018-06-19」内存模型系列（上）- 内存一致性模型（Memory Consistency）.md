
# Memory Models Series - Memory Consistency (Slides & Talk)

> **日志**：
>
> 1. [2019-02-01] PPT 版式升级，阅读面积提升约三分之一，提供沉浸式阅读体验；文字描述从图片上方替换到下方，使更符合阅读 PPT 的习惯；翻新已有内容，使更容易理解；补全文字描述；
> 1. [2018-06-19] 完成了本文的 PPT 框架及除 TSO 外的全部文字说明。

---

> **作者按**：内存模型系列（上）- 内存一致性模型。本文为内存模型系列上篇，主要深入浅出地介绍了用于描述访存顺序及访存原子性的一致性模型。本文主要面对对象为刚接触并发编程的编程者，为了方便读者理解，本文将尽量采用偏口语的文风，并尽量避免有关硬件实现的部分。其实后半部分没法谈也不需要谈吧，这个话题太大了。作为一名编程者，你只需要用理解内存模型给出的承诺就可以了。
>
> - 本系列的下篇为：[内存模型系列（下）- 内存持久性模型（Memory Persistency）](https://blog.csdn.net/maokelong95/article/details/81199226)。
> - 本博客所有文本及胶片均公开在了 [Github](https://github.com/maokelong/CSDN-maokelong95) 上，如需转载，注明出处即可。如若建议、勘误等，发起 Issue 或在下方评论均可。

---

[toc]

## 1. 简介

![1](https://img-blog.csdnimg.cn/20190203001605843.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

首先介绍一下本文的大纲。本文将尝试性地采用讲 PPT 的形式展开。PPT 的主要内容共计 20 页，其中 5 页讲了我们为什么需要了解（内存）一致性模型，5 页讲了最严格的一致性模型，也即顺序一致性模型，10 页以 x86 的 TSO 为例讲了较为宽松的一致性模型，最后 1 页列出了我所参考的文献。本文中所有图片的介绍都在图片下方。第二章见本系列下篇。

![2](https://img-blog.csdnimg.cn/20190203001722871.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在谈到并发编程的时候，我们主要有两种编程范式：消息传递及共享内存。前者主要用于分布式系统，通过网络，采用消息的发送及接收的方式完成对共享数据的访问；而后者主要用于单机，通过内部互联总线，采用读写共享内存的方式完成对共享数据的访问。一致性模型为后者服务。

> **简单的常识**：典型的处理器（Processor）往往包含多个核心（Core），各个核心私有 L1/L2 缓存，而共享 L3 缓存和内存控制器。 -- [Intel® Xeon® Scalable Processors](https://blog.csdn.net/maokelong95/article/details/78604037)。

![3](https://img-blog.csdnimg.cn/20190203001734538.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

下面考虑一段很简单的程序。下图中，三个处理器（P1、P2、P3）依次执行三段程序片段。在执行虚线下部分代码之前，三个处理器**共享**三个初始化为零的变量，ABC。按照你的理解，下例中可能输出的结果是什么呢？

你很可能会说，A == 1 && C == 1。是的，这是最符合直觉的结果。但别的结果，包括 A == 0 && C == 1、A == 1 && C == 0 及 A == 1 && C == 0 都是有可能的。这是怎么回事呢？

以 A == 0 && C == 1 为例，当 P1 上的赋值语句 C = 1 及 A = 1 完成之后，P2 **看见** P1 对 A 的修改，从而跳出循环并执行 B = 1，接下来 P3 **看见** P2 对 B 的修改，从而跳出循环并打印 AC。在打印 AC 时，存在这样的可能，也即 P3 **看见了** P1 对 C 的修改，而**没看见** P1 对 A 的修改，从而打印出 A == 0 && C == 1。

一言以蔽之，其中的坑在于，你以为 C = 1 执行完之后，所有处理器都应该看到 C 的最新值，但实际可能并不是这样。

> **注意**： 本文会使用「看见」这个词。当一个处理器（PA）对共享变量（V）的本地副本进行了修改，而另一个处理器（PB）用于保存相应变量的缓存行因此而失效或被更新，则本文称 PB 看见了 PA 对变量 V 的修改。


![4](https://img-blog.csdnimg.cn/2019020300174844.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

所以到底是谁规定着，处理器能否在一个变量为指定处理器可见之前继续执行下去呢？就是今天我们要介绍的主角，**内存一致性模型**。方便起见，以下均简称为一致性模型。

![5](https://img-blog.csdnimg.cn/20190203001804198.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

首先，让我们来看看一致性模型的定义。所谓**一致性模型**，也就是一个用来规定新值什么时候传播到某一指定处理器的策略。大概啥时候，给句亮堂话嘛。这个定义看起来有点像**缓存一致性协议**，所谓缓存一致性协议也即一个用来将修改过的缓存副本传播给别的缓存的算法。

俩兄弟看着挺像的，但前者确定的是何时，而后者确定的是如何。另外，还有些系统，虽然它们没有狭义上的缓存，但它们也存在一致性问题。

![6](https://img-blog.csdnimg.cn/20190203001812302.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

比如，给定一种系统，它在每个处理器内部都设置了一块写缓冲区，用以吸收写指令，并异步地将这些写指令分别提交到内存控制器中。后续，本处理器在读取内存时，如果检测到写缓冲区里已经存在针对同一地址的写了，那么它就直接解析这个写指令要写入的值并返回。显然，这样的设计，其优点在于：它能够很好地增加访存指令吞吐量。

这样的设计放在单处理器系统上，是没有问题的，这很类似于处理器设计里常见的重定向设计。但是，在多处理器系统上就麻烦了，处理器很可能会从内存中读到过期的数据，因为相关的写操作可能仍缓冲在另一个处理器的缓冲区里，从而遭遇一致性问题。

> **注意**：显然，这篇文章是以典型的 SMP 结构而非时下盛行的 NUMA 结构展开的。在 SMP 结构中所有处理器共享内存控制器，而 NUMA 结构为避免内存控制器的带宽成为瓶颈，在每个处理器内都设有内存控制器。

换作采用内部互联总线的系统可能也存在类似的问题，详情请参考文献[1]，无非是之前数据缓冲在了缓冲区，而现在缓冲在了寄存器上罢了，总之都造成了不同的处理器看见不一致数据的现实情况。

> **注意**：这一段纯属考古，感兴趣就看看，没兴趣完全可以跳过。另外本文中的访存（Memory Access）指令包括读存指令（Load）及写存指令(Store)。

## 2. 顺序一致性

![7](https://img-blog.csdnimg.cn/20190203001826206.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

一致性模型里最严苛的就是顺序一致性了，它的定义很绕口：我们称一个多处理器系统是「顺序一致的」，当该多处理器系统的运行结果看起来好像所有处理器的操作按照某种线性顺序执行，且在该线性顺序中，各处理器的操作还遵循程序指定顺序。

当然，你们勤劳的博主是不会把这么一大段难消化的东西直接丢给你们的，这样也太失败了。其实大家抓住两个关键词就可以了：**程序顺序**和**原子性**。

![8](https://img-blog.csdnimg.cn/20190203001840915.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

所谓程序顺序，也就是说各处理器的操作保持程序指定的顺序。

为什么要约束到程序顺序？以下图程序为例，开发者原本期望通过 Flag1 和 Flag2 来保护临界区，使得应两个线程之中，只有一个能访问临界资源。但是，如果如果处理器 / 编译器调度了指令的执行顺序，使得写入 Flag1 和 Flag2 的操作被挪到了读取 Flag1 和 Flag2 之前，假设在某个时间点，两个处理器恰好都执行了读取 Flag1 和 Flag2 的操作并据此完成判断，而正准备写入 Flag1 和 Flag2，就会出现两个线程都能访问临界资源的情况，从而违背了开发者的期望。

所以这和一致性有什么关系吗？如果访存指令的顺序被调整了，那么就可以理解为，甚至连本处理器都将延迟看到本条指令的结果。翻到上面去看看一致性模型的定义，是不是觉得就明白了？

![9](https://img-blog.csdnimg.cn/20190203001933411.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

所谓原子性，也就是说各处理器能够同时看到访存指令的结果。

为什么要约束原子性？这是很喜闻乐见的事情。拿这个曾经讲过的例子来说，如果原子性得以保证，那么 register1 的值就不可能是 0 了。因为 register1 能被赋值说明各处理器均看见了 B 的最新值，而 B 能被赋值则说明各处理器均看见了 A 的最新值。既然 P3 在赋值 register1 之前能看见 A 的最新值，register1 就会被赋为 A 的最新值，也即 1。

![10](https://img-blog.csdnimg.cn/20190203001945253.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

虽然开发者可能很喜欢顺序模型，因为真的很容易理解，但编译器就不开心了。以前能用的那些基于重排序的优化都不能用了，比如代码移动、寄存器染色啊、子表达式消除。这些统统都凉了。

## 3. 宽松一致性

![11](https://img-blog.csdnimg.cn/2019020300195615.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

所以说得把顺序一致性的要求放低一点。虽然这样会让大家脑壳子疼一些，但其实是一种利好，有利于系统性能的提升。比顺序一致性更宽松的内存模型就称之为宽松一致性。

下图是一张考古图，里面放了各大著名内存模型的特征。比较新的可以移步维基百科「[Memory Ordering](https://en.wikipedia.org/wiki/Memory_ordering)」。实用着想，这里主要以 Intel 所采用的 TSO 模型为例展开。

图表中标题的含义将在后面介绍。

> **注意**：1. 最早是 SPARC 采用的 TSO，只是现在 Intel x86 更出名一些；2. 在维基百科上你能看到「x86 oostore」，不用管它，这鬼东西已经灭亡了。

![12](https://img-blog.csdnimg.cn/20190203002006346.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

什么叫做 TSO 呢。当各处理器看见所有写存指令的执行顺序是一样的时候，我们称这些写存指令是 **TSO** 的。我不喜欢直接丢定义，所以接下来我会很结合大量例子详细地介绍 TSO 的特性。恩，特性而非定义，才是大家最喜闻乐见的。

在具体讲解之前，先给大家打一针预防针。前面我们也谈到了，顺序模型主要从**程序顺序**及**原子性**两个维度上约束了新值为任一处理器可见的时机。自然而然地，我们放松一致性模型时也会从这两方面入手。对应到上张图中的表，2~4 列就是用来描述是如何放松程序顺序的，5~6 列就是用来描述如何放松原子性的。那么放松的一致性模型怎么再变得严格起来呢？所预留的手段就称之为安全网（Safety Net）。

另外你应该会很困惑「→」是个什么鬼，其实这就是 Happened-before 关系的标识，通俗地理解就是，当 A→B 时，B 必须发生在 A 之后。

![13](https://img-blog.csdnimg.cn/20190203002020558.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在继续介绍这些例子之前，不妨先约束一些助记符。以下各例子中，1）使用 M1、M2 等标记指令；2）使用 `mov [_x], val` 表示 store 指令，其中 val 将被存储到内存的 x 位置中；3）使用 `mov r, [_x]` 表示 load 指令，其中内存 x 位置中的数据将被读取到寄存器 r 中。

本例介绍了在程序顺序一项上，写后读的顺序可以改变。其直接影响为：M2->M1 和 M4->M3 均是有可能的。

于是可能存在这么一瞬间，恰好仅执行了 M2 和 M4，而未执行 M1 和 M3，使得 r1 和 r2 均仅读取到 x 和 y 的初始值，也即 0。

> **注意**：二者访问相同内存位置时不会排序。

![14](https://img-blog.csdnimg.cn/20190203002038229.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

本例介绍了在程序顺序一项上，写后写和读后读写的顺序不可以改变。其直接影响为：M1->M2，M3->M4。

`r2 == 0`，说明 M1 未执行而 M3 已执行。根据前述「M1->M2」，M2 必然也未执行。于是 M3 中只能读出 M2 生效前的 y 值，也即 0。所以 `r2 == 0 && r1 == 1` 是不可能的。

![15](https://img-blog.csdnimg.cn/20190203002049181.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

本例介绍了在原子性一项上，不能提前读取到别的核心/处理器的写。这句话看起来可能会让你一头雾水，实际上这样理解就可以了：某个核心/处理器修改了变量 x 之后，别的处理器要么都看到这个修改，要么都没看到这个修改，不能说其中某个瞒着别人偷偷看到了。其直接影响为：M3 和 M5 须同时看见 M1 或 M2。

[x] R->R 约束着 M3->M4，于是若 `r1 == 1 && r2 == 2`，则说明 P2 先后看见 M1 和 M2。P3 亦然。[x] R->R 也约束着 M5->M6，则 r1 和 r2 必然先后输出 0、1、2 中的任一或任意先后两个。而 `r1 == 2 && r1 == 1` 不在其列，是不可能的。

> **建议你了解的内容**：Intel 在《手册》[^1]中介绍本例时，采用的是这样的描述：「**Total order on stores** to the same location」，也即：所有处理器眼中，针对同一位置的写操作其顺序应当是一样的。比如各处理器都会对 x 写，其中某个处理器观测到这样的写顺序：1->2->3->4->5，各处理器一定也会看到这样的顺序，虽然是在 1 还是 2 还是别的时候看到可能是随缘的事情。

![16](https://img-blog.csdnimg.cn/20190203002102307.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

本例介绍了在原子性一项上，存在因果律关系。这听着很玄乎的，其实不过是对上条规则的简单演绎。看示例就明白了。

若 `r1 == 1`，说明 M1->M2，且 x 新值已经传播到了 P1 和 P2；若 `r2 == 1`，说明 M3->M4，且 y 新值已经传播到了 P0 和 P2。根据 [x] R->RW，有 M3->M4->M5，M2->M3。显然，根据上述推导过程，可得 M1->M2->M3->M4->M5，M5 应能看见 M1 结果，得出 `r == 1`。

![17](https://img-blog.csdnimg.cn/20190203002114162.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

本例介绍了在原子性一项上，可以提前读取本核心 / 处理器的写。也就是说，本核心 / 处理器不必等待别的核心 / 处理器看见本核心 / 处理器的写操作，就可以继续运行下去。其直接影响为：这种情况，也即 P0 在看见 M4 之前看见 M1，且 P1 在看见 M1 之前看见 M4，是可能的。

简化起见，这里不考虑 W->R 的排序，如果排序了可以直接读到初始值，自然是可能的，如果不排序则有 M1->M3，M4->M6。假设 M3 和 M6 执行完之时，P0 仍未看见 M4，而 P1 仍未看见 M1，就会出现 `r2 == 0 && r4 == 0` 的情况。
 
> **无需要了解的内容**：Intel 在《手册》[^1]里备注道，这种结果可能因「store-buffer forwarding」而起。一个小常识，核心流水线中的访存器在发射 store 之后，并非直接将数据提交到 Cache，而是先提交到 StoreQueue 中。若后续的 load 指令直接命中了 StoreQueue 中缓存的 store，则称发生了 forwarding。当 M1 和 M2 都写到了 StoreQueue 中，自然而然地，别的核心无法看到 M1 或 M2，而本核心却可以 forwarding。

[^1]: Intel Corporation. Intel 64 Architecture Memory Ordering White Paper, Aug 2007.

![18](https://img-blog.csdnimg.cn/20190203002125753.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

前述放松的一致性模型可以再次变得严格起来，能够达成这样效果的手段就称之为安全网（Safety Net）。介绍 TSO 自然不能遗漏其安全网。

TSO 的安全网有两种：1）read-modify-write（以下简称为 RMW）；2）membar。

先谈 RMW。

用论文[^3]的话来说，当读 / 写指令已经是 RMW 或被 RMW 替换后，即能保持程序顺序；用 intel[^2] 的话来说，「Locked instructions have a total order」，「Loads and stores are not reordered with locks」。显然 Intel 的说法更苛刻一些，其能同时从原子性和程序顺序两个维度上，而非仅从程序顺序这一个维度上，加强约束，使达到了顺序一致性的地步。这就是 RMW 的作用。

[^3]: S. Adve and K. Gharachorloo. Shared memory consistency models: a tutorial. Computer, 29(12):66–76, 1996.

那么 RMW 是什么？intel 在《手册》[^2] 中介绍到，所谓 RMW，也即 intel 64 架构中的 locked 指令，其包括隐式 locked 指令如 `xchg` 等，也包括别的（助记符中）以 lock 为前缀的 RMW 指令。这一类指令的特点正如其名，能够使用一个指令完成读取-修改-写入这样复合的行为。比如 `xchg`，其作用为将两个操作数中的内容置换，显然需要读-写。

```asm
XCHG reg, reg // 置换寄存器
XCHG reg, mem // 置换寄存器和内存
XCHG mem, reg // 置换内存和寄存器
```

> **扩展阅读**：[为什么 read-modify-write 不干脆叫 read-write](https://stackoverflow.com/questions/49452022/why-its-termed-read-modify-write-but-not-read-write)。

[^2]: Guide P. Intel® 64 and IA-32 Architectures Software Developer’s Manual[B]. Volume 3B: System programming Guide, 2011.

![19](https://img-blog.csdnimg.cn/20190203002139577.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

上页谈到可以将普通读写指令替换为 RMW 指令，从而加强约束。那么在实践中该如何操作呢？

- 对于读，将其「包装」为一个「假写」的 RMW 指令，把读到的值写回原处；
- 对于读，将其「包装」为一个「假读」的 RMW 指令，不管读到了什么都把要写的值写进去；

<!-- TODO: 加个例子 -->

![20](https://img-blog.csdnimg.cn/20190203002151797.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

再谈谈 membar。虽然 membar 的资料实在是太多了，但为了读者的一站式阅读体验，这里还是稍微谈一下。

其实 membar 有很多种称呼，至少 wikipedia 是这样说的。什么 Memory barrier（内存屏障，或简称为 membar），memory fence（内存栅栏，或简称为 fence），都是同一个东西。

membar 是什么？一种特别的屏障指令，其导致 CPU 或编译器强制约束内存指令，使之在屏障之前或之后发射。以 mfence 这个指令为例，其保证在程序顺序中，mfence 之前的所有访存指令，其结果都能为 mfence 之后的访存指令所见。

> **注意**：发射，或 issue，在乱序处理器中指指令进入指令调度器；而派发，或 dispatch，指指令进入执行单元。但一般将发射理解为已经开始执行了。

## 参考文献

![21](https://img-blog.csdnimg.cn/20190203002223288.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

下图给出了本文主要参考的文献，其中本文的内容框架启发自第一篇文献，本文中有关 TSO 的介绍主要抠自第二篇文献，本文中有关锁指令的描述主要抠自第三篇文献。
