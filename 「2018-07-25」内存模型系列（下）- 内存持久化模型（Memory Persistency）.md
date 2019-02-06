
# Memory Models Series - Memory Persistency (Slides)

> **日志**：
>
> 1. [2019-02-06] PPT 版式升级，阅读面积提升约三分之一，提供沉浸式阅读体验；
> 1. [2018-07-25] 完成了本文的 PPT 框架。

---

> **作者按**：内存模型系列 - 内存一致性模型。本文为内存模型系列下篇，主要深入浅出地介绍了用于约束内存持久化指令完成顺序的内存持久性模型（Memory Persistency）。该模型面向未来的 NVMM（Non-volatile Main Memory，非易失主存）系统，其核心主张为：通过将 Memory Consistency 与 Memory Persistency 解耦，而 1) 帮助架构师挖掘指令级并行度，2) 帮助编程者推理崩溃一致性。
>
> - 本系列的上篇为：[内存模型系列（上）- 内存一致性模型（Memory Consistency）](https://blog.csdn.net/maokelong95/article/details/80727952)。
> - 本博客所有文本及胶片均公开在了 [Github](https://github.com/maokelong/CSDN-maokelong95) 上，如需转载，注明出处即可。如若建议、勘误等，发起 Issue 或在下方评论均可。

---

[toc]

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