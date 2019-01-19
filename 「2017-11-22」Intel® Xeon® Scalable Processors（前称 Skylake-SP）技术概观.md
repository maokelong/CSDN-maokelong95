
# Intel® Xeon® Processor Scalable Family Technical Overview

>**Note**：
>
> 1. Intel® Xeon® Scalable Processors with Intel® C620 Series Chipsets，其前称包括 Purley，Skylake-SP 和 Lewisburg。-- [Intel Products & Solutions](https://www.intel.com/content/www/us/en/design/products-and-solutions/processors-and-chipsets/purley/intel-xeon-scalable-processors.html)
> 1. 本文将 processer family 译作「处理器族」；
> 1. 本文将 socket 译作「槽」；
> 1. 原文刊于 2017-09-14，详见 [Intel® Xeon® Processor Scalable Family Technical Overview](https://software.intel.com/en-us/articles/intel-xeon-processor-scalable-family-technical-overview)，本文对原文进行了一定的提炼和注释；

---

[toc]

## Executive Summary

Intel 使用 tick-tock 模型迭代处理器，而本代「Intel® Xeon® Processor Scalable Family」就是基于 14nm 技术的 tock。

![tick-tock model](https://software.intel.com/sites/default/files/managed/91/45/xeon-processor-scalable-family-tech-overview-fig01.png)

跟上一代 「Intel® Xeon® processor E5-2600 v4 product family」（Broadwell 微架构） 相比，这一代的新特性包括：

- 增加了核数
- 增加了内存带宽
- [Non-inclusive cache](https://en.wikipedia.org/wiki/Cache_inclusion_policy)
- Intel® Advanced Vector Extensions 512 (Intel® AVX-512)
- Intel® Memory Protection Extensions (Intel® MPX)
- Intel® Ultra Path Interconnect (Intel® UPI)
- Sub-NUMA clusters

前代将 2 / 4 槽处理器族分为两个不同的产品线，而本代只有一个处理器族，其包含所有的处理器模型！

前后三个处理器族系列的对等关系如下图所示：

![New branding for processor models](https://software.intel.com/sites/default/files/managed/d0/c6/xeon-processor-scalable-family-tech-overview-fig02.png)

不难发现，本代命名采用的是金属系，其中：

- 铂金（Platinum [ˈplætɪnəm]）支持拓展至 8+ 槽；
- 金牌（Gold [gəʊld]）支持拓展至 4 槽；
- 银牌（Silver [ˈsɪlvə(r)]）支持拓展至 2 槽；
- 铜牌（Bronze [brɒnz]）同银牌。

> 以上羞耻度爆表的译名是 intel 官网亲自翻译的... 

其中铂金级支持本代所有特性。

> 亚马逊卖 6K$ 一颗，向土豪低头 orz...


## 微架构概观

本代的提升在于：

- 核数多达 28（前代 22）；
- Non-inclusive last-level cache；
- 1MB L2 cache；
- 2666 MHz DDR4 memory；
- 6 Memory Channels / CPU；
- New memory protection features；
- Intel® Speed Shift Technology；
- on-die PMAX detection；
- integrated Fabric via Intel® Omni-Path Architecture (Intel® OPA)；
    - 集成了 [Omni-Path Fabric](https://www.intel.cn/content/www/cn/zh/high-performance-computing-fabrics/omni-path-architecture-fabric-overview.html)。
- Internet Wide Area RDMA Protocol (iWARP)*；
- Intel® Virtual RAID on CPU ([Intel® VROC](https://www.intel.com/content/www/us/en/support/articles/000024498/memory-and-storage/ssd-software.html))；
    - 允许 NVMe SSDs 通过 PCIe 连接，并直接被 CPU 管理，从而组成 NVMe RAID。
- ...

具体信息请看原文。

本代微架构概观图：

![Block Diagram of the Intel® Xeon® processor Scalable family microarchitecture](https://software.intel.com/sites/default/files/managed/19/d1/xeon-processor-scalable-family-tech-overview-fig03.png)

前后三族处理器对比表：

![Generational comparison of the Intel Xeon processor Scalable family to the Intel® Xeon® processor E5-2600 and E7-4600 product families](https://software.intel.com/sites/default/files/managed/e7/ad/xeon-processor-scalable-family-tech-overview-table-01.png)

## 特性概观

新特性 / 技术见下表：

![New features and technologies of the Intel Xeon processor Scalable family](https://software.intel.com/sites/default/files/managed/6b/1a/Untitled.png)

限于时间精力有限，这里仅摘选部分博主比较感兴趣的特性，其余特性请自行查阅原文。

在摘选的部分中：

- 第一部分谈了新型的片上核心连接架构；
- 第二部分谈了新平台的 NUMA 拓扑结构；
- 第三部分谈了新型处理器缓存架构；
- 第四部分谈了新型页保护算法；

### Skylake Mesh Architecture

过去 Grantley 平台上的处理器族（Haswell 和 Broadwell），其处理器、核心、LLC、内存控制器、I/O 控制器及槽间的 Intel® QPI 端口均使用环形架构连接。

然而随着 CPU 中核心数的迭代，访问延迟不断增加，而核心可用带宽不断减小。

彼时 intel 为了缓解这个问题，将芯片分为两半，然后引入了第二个环，用于减少路径长及增加带宽。

Broadwell-EP 中的双环架构见下图：

![Intel® Xeon® processor E5-2600 product family (formerly Broadwell-EP) on Grantley platform ring architecture](https://software.intel.com/sites/default/files/managed/e3/a4/xeon-processor-scalable-family-tech-overview-fig04.png)

然而本代单处理器核心数、内存带宽及 I/O 带宽继续增加，片上通信的需求随之增加。倘若此时仍采用落后的环形架构，则可能会导致片上通信成为处理器性能的瓶颈。

因此本代采用了新的网孔架构，其包含一系列横竖交叉的通信路径，使得两个核心之间能够以最短的路径进行通信。

本代还以模块化和分布式的方式将 cache agent、home agent 和 I/O subsystem 集成到了网孔上，以消除访问这些功能的瓶颈。

现在每个核心及其 LLC 切片都有一个 combined Cacheing and Home Agent（CHA），该组件为 intel® Ultra Path Interconnect (Intel® UPI) 缓存一致性功提供了资源跨越的扩展性。

网孔状架构如下图所示：

![Intel Xeon processor Scalable family mesh architecture](https://software.intel.com/sites/default/files/managed/5a/03/xeon-processor-scalable-family-tech-overview-fig05.png)

除了降低 core-to-cache 和 core-to-memory 的延迟，该架构还降低了 I/O 启动访问的延迟。以前访问 LLC / memory / IO 的资源时，若 source 和 targets 不在同一个环中，则 core 或 I/O 可能会绕环然后经过环间交换器的仲裁。而本代则可直接在网孔中，以最短路径访问 LLC / memory / IO 的资源。

### Intel® Ultra Path Interconnect (Intel® UPI)

本代使用 UPI 替代了前代的 QPI。支持 UPI 的处理器会提供两到三个 Intel UPI 链接，以提供高速、低延迟的处理器间数据传输。

UPI 是一个一致互联组件，用于可拓展系统，该系统包含共享单个地址空间的多个处理器。 

具体而言，其采用了 directory-based home snoop coherency protocol（详见原文余下章节），能够提供高达 10.4 GT/s 的运行速度。

典型的 2 槽 / 4 槽(ring & crossbar) / 8 槽 配置：

![2](https://software.intel.com/sites/default/files/managed/7c/6e/xeon-processor-scalable-family-tech-overview-fig06.png)

![2](https://software.intel.com/sites/default/files/managed/77/f2/xeon-processor-scalable-family-tech-overview-fig07.png)

![2](https://software.intel.com/sites/default/files/managed/77/72/xeon-processor-scalable-family-tech-overview-fig08.png)

![2](https://software.intel.com/sites/default/files/managed/77/72/xeon-processor-scalable-family-tech-overview-fig08.png)

### Cache Hierarchy Changes

前代的 mid-level cache(MLC) 为 256KB/Core，而 shared last level cache(LLC) 为 2.5MB/Core，且是 includesive 的。

而本代 MLC 升级为 1MB/Core，而 shared LLC 为 1.375MB/Core，且是 non-includesive 的。

本代所有 Cache Miss 之后，核心直接将内存中的行取到其 MLC 中，而不再同时在 MLC 和 LLC 中保存一份拷贝。当然，Cache Line 被逐出后仍会放入 LLC 中。

![Generational cache comparison](https://software.intel.com/sites/default/files/managed/55/90/xeon-processor-scalable-family-tech-overview-fig11.png)


((256 / 1024  + 2.5) - (1 + 1.375)) * 1024 = 384。没错，本代平均每个核心少了 384KB 缓存！ 

对此，Intel 的解释详见原文。

### Page Protection Keys

复杂的多线程应用程序常因意外的写操作而导致内存崩溃问题。例如，数据库应用的各个部分不需要具有相同级别的特权。日志写入器应该具有对日志缓冲区的写入权限，但它应该仅对其他页面具有读取权限。类似地，在某些生产者与消费者线程应用中，生产者线程可以具有特定页面上的额外权限。

基于页的内存保护算法可以用于更复杂的应用，然而改变页表十分影响性能，因为这些改变会导致 TLB 失效并随后引发 TLB 不命中。现在「Protection keys，保护键」能够提供用户级的、以页为粒度的方式，来授予及撤销访问权限，而无需修改页表。

保护键为用户的页提供 16 个 Domain，也即 「Protection Domain，**PKEY**，保护域」，每个保护域都在一个新的、名为 **PKRU** 的线程私有寄存器中设有两个许可位。保护键使用页表叶结点（如 PTE）的 62:59 位来识别保护域。

访问内存时，页表查询阶段将确定本次访问的保护键，而相应保护域确定的访问权限，即本次访问是否将授予读写权限，将由 PKRU 的内容确定。只有当保护键和传统页保护同意本次访问时，该访问才允许执行。当保护键不同意本次访问时，处理器将给出页错误异常并返回一个新的错误码。

> 有关 supervisor 的部分略。

![Diagram of memory data access with protection key](https://software.intel.com/sites/default/files/managed/d7/b6/xeon-processor-scalable-family-tech-overview-fig12.png)

为了受益于保护键，要求来自 VM Manager、OS 及 Compiler 三者共同的支持。使用本特性不会带来额外性能开销，因为它是内存管理架构的扩展。
