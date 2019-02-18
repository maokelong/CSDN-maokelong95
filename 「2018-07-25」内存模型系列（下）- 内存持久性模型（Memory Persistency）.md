
# Memory Models Series - Memory Persistency (Slides)

> **日志**：
>
> 1. [2019-02-13] 引入了前言；
> 1. [2019-02-06] PPT 版式升级，阅读面积提升约三分之一，提供沉浸式阅读体验；
> 1. [2018-07-25] 完成了本文的 PPT 框架。

---

> **作者按**：内存模型系列 - 内存一致性模型。本文为内存模型系列下篇，主要深入浅出地介绍了用于约束内存持久化指令完成顺序的内存持久性模型（Memory Persistency）。该模型面向未来的 NVMM（Non-volatile Main Memory，非易失主存）系统，其核心主张为：通过将 Memory Consistency 与 Memory Persistency 解耦，而 1) 帮助架构师挖掘指令级并行度，2) 帮助编程者推理崩溃一致性。
>
> - 本系列的上篇为：[内存模型系列（上）- 内存一致性模型（Memory Consistency）](https://blog.csdn.net/maokelong95/article/details/80727952)。
> - 本博客所有文本及胶片均公开在了 [Github](https://github.com/maokelong/CSDN-maokelong95) 上，如需转载，注明出处即可。如若建议、勘误等，发起 Issue 或在下方评论均可。

---

[toc]

## 前言

在开始介绍内存持久性模型之前，先为大家讲一下提出这个模型的背景。

持久内存（Persistent Memory，PM），也即能够通过常规访存指令（而非系统调用）访问的，具有低延迟（而非 I/O 总线）及字节可寻址（而非块）特性的非易失存储器[^whisper]。基于 3D XPoin 的 Intel® Optane™ DC Persistent Memory，基于 DRAM 和 NAND Flash 的 NVDIMM-N 等，都是持久内存。下表给出了常见存储器的部分属性，其中字节可寻址（byte-addressable）表示以字节而非块为基本单位进行寻址，而非易失性（non-volatile）表示在掉电后其中的数据不会挥发。

> **注意**：早期文献中也常使用 SCM（Stroage Class Memory），NVMM（Non-volatile Main Memory），NVDIMM（Non-volatile DIMM）等词指代持久内存，但近来学术界和工业界已统一使用持久内存一词。

| 存储器                   | 读取延迟 | 写入延迟 | 字节可寻址 | 非易失性 |
| ------------------------ | -------- | -------- | ---------- | -------- |
| DRAM[^survey]            | 50ns     | 50ns     | 是         | 否       |
| NAND FLASH[^flashsummit] | 10μs     | 10μs     | 否         | 是       |
| 3D XPoint[^flashsummit]  | 100ns    | 500ns    | 是         | 是       |


[^flashsummit]: Raghu Kulkarni. Persisent Memory and NVDIMMs [C]. Flash Memory Summit 2018. SNIA. 2018.
[^survey]: Mittal S, Vetter J S. A survey of software techniques for using non-volatile memories for storage and main memory systems[J]. IEEE Transactions on Parallel and Distributed Systems (TODS’16), 2016, 27(5): 1537-1550.
[^whisper]: Nalli S, Haria S, Hill M D, et al. An analysis of persistent memory use with WHISPER[C]//Architectural Support for Programming Languages and Operating System (ASPLOS'17). ACM, 2017: 135-148.

持久内存并不是什么虚无缥缈，离落地很远的事情。实际上持久内存有望于 2019 进入市场[^intelnews]。有资料表明，很多大型 IT 企业已经拿到了其工程样片，并据此进行了早期研发：[非易失性内存在阿里生产环境的首次应用: Tair NVM最佳实践总结](https://102.alibaba.com/detail?id=165) 。

[^intelnews]: https://newsroom.intel.com/editorials/re-architecting-data-center-memory-storage-hierarchy/, May 30, 2018.

在使用持久内存的时候，一个问题需要被严肃对待：内存一致性问题。出于各种此处不便展开的因素，未来持久内存问世的时候，处理器中的寄存器及缓存仍将是易失性，而大电容仅能确保掉电后内存控制器中的数据写入持久内存[^derpecate_pcommit]，使得持久内存中的数据可能并非数据的最新副本。因缓存-内存中的数据不一致而引发的问题，便称为内存一致性问题。内存一致性问题的影响无疑是巨大的，轻则导致数据丢失，重则导致系统无法恢复。

[^derpecate_pcommit]: https://software.intel.com/en-us/blogs/2016/09/12/deprecate-pcommit-instruction

那么如何避免内存一致性问题呢？不同工作从不同角度进行了解读，但最广为认可的方案还是由应用显示调用**持久化指令**，将缓存的数据刷入内存控制器中。有关持久化指令的介绍见[博主另外的博文](https://blog.csdn.net/maokelong95/article/details/81362837)。

> **注意**：有论文总结了解决一致性问题的若干种方法，可见《Programming for Non-Volatile Main Memory Is Hard》。

毫无疑问，持久化指令的执行效率将是十分关键的。在最坏的情况下，如 CLFLUSH 的设计，所有持久化指令顺序地、毫无重叠（overlapped）执行，而这显然是大家无法接受的。在某些不强调这些指令完成顺序的情况下，如内存拷贝，大家还是期望能够牺牲对持久化指令的顺序约束，而获得效率上的提升。如果大家了解过[内存一致性模型](https://blog.csdn.net/maokelong95/article/details/80727952)，肯定会想为什么不扩展已有内存一致性模型，去专门描述对持久化指令的约束呢？实际上已经有人提出了这样扩展的内存模型，也就是今天要介绍的内存持久性模型。

## 1. Terminology

![21](https://img-blog.csdnimg.cn/20190206230824904.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![22](https://img-blog.csdnimg.cn/20190206230832806.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

## 2. Strict Persistency

![23](https://img-blog.csdnimg.cn/2019020623083979.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![24](https://img-blog.csdnimg.cn/20190206230844347.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

## 3. Relaxed Persistency
### 3.1 Epoch Persistency

![25](https://img-blog.csdnimg.cn/201902062308555.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![26](https://img-blog.csdnimg.cn/20190206230900336.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![27](https://img-blog.csdnimg.cn/20190206230906482.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![28](https://img-blog.csdnimg.cn/20190206230912468.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![29](https://img-blog.csdnimg.cn/2019020623091821.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

### 3.2 Strand Persistency

![30](https://img-blog.csdnimg.cn/20190206230923269.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

## Reference
![31](https://img-blog.csdnimg.cn/2019020623092922.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
