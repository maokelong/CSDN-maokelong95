# Memory Models Series - Memory Persistency (Slides)

> **日志**：
>
> 1. [2019-02-03] PPT 版式升级，阅读面积提升约三分之一，提供沉浸式阅读体验；文字描述从图片上方替换到下方，使更符合阅读 PPT 的习惯；补全文字描述；
> 1. [2018-07-25] 完成了本文的 PPT 框架。

---

> **作者按**：内存模型系列 - 内存一致性模型。本文为内存模型系列下篇，主要深入浅出地介绍了用于约束内存持久化指令完成顺序的内存持久性模型（Memory Persistency）。该模型面向未来的 NVMM（Non-volatile Main Memory，非易失主存）系统，其核心主张为：通过将 Memory Consistency 与 Memory Persistency 解耦，而 1) 帮助架构师挖掘指令级并行度，2) 帮助编程者推理崩溃一致性。
>
> - 本系列的上篇为：[内存模型系列（上）- 内存一致性模型（Memory Consistency）](https://blog.csdn.net/maokelong95/article/details/80727952)。
> - 本博客所有文本及胶片均公开在了 [Github](https://github.com/maokelong/CSDN-maokelong95) 上，如需转载，注明出处即可。如若建议、勘误等，发起 Issue 或在下方评论均可。
---

[toc]

## 1. 引言
![大纲](https://img-blog.csdn.net/20180725111923573?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2. 术语

![这里写图片描述](https://img-blog.csdn.net/20180725111937867?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180725111949914?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 3. Strict Persistency
![这里写图片描述](https://img-blog.csdn.net/20180725112000334?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180725112012767?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 4. Relaxed Persistency


### 4.1 Epoch Persistency
![这里写图片描述](https://img-blog.csdn.net/2018072511204068?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180725112049335?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180725112057105?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180725112104629?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180725112114403?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 4.2 Strand Persistency
![这里写图片描述](https://img-blog.csdn.net/20180725112136813?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## Reference
![这里写图片描述](https://img-blog.csdn.net/20180725112143903?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
