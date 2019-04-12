# Visual Studio Code 教程：从入门到精通

> **日志**：
>
> 1. 「2019-03-25」提交了本文的 PPT 框架。

---

> **作者按**：Visual Studio Code，或简称为 VSCode，是我最喜欢的代码编辑器，我希望能有更多人享受到 VSCode 的便捷与强大。在希望学习一门新技术的时候，我们往往会想着去搜索一份教程。根据二八定律，我们往往只需了解一门技术的部分最常用的功能，就足以应对大多数开发场景，因此介绍最常见功能的教程，而非介绍全部特性的手册，成了我们的学习技术的首选。然而已存教程不足以成为我们了解 VSCode 的窗口，很多具有广泛使用场景的功能，比如命令窗口和终端，这些教程都没有涉猎。该种情况促成了本文的诞生。本文希望，哪怕是从未使用过 VSCode 的读者，也能在阅读本文后便精通使用 VSCode。
>
> - 本博客已经发布了大量优质教程，其中包括阅读量超过五万的 [《[VS Code]跟我一起在Visual Studio Code 添加自定义snippet（代码段），附详细配置》](https://blog.csdn.net/maokelong95/article/details/54379046)。请放心食用。
>
> - 本博客所有文本及胶片均公开在了 [Github](https://github.com/maokelong/CSDN-maokelong95) 上，如需转载，注明出处即可。如若建议、勘误等，发起 Issue 或在下方评论均可。

---

[TOC]

## 大纲

![2](https://img-blog.csdnimg.cn/20190325181443425.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

本文照例以一份 PPT 展开。Markdown 直接贴图太丑了，还是组织成 PPT 之后直接贴上来吧。本文所有文字介绍都在图片下方。

## 简介

![3](https://img-blog.csdnimg.cn/20190325183000669.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

Visual Studio Code 是微软出品的轻量级跨平台编辑器。数据显示，Visual Studio Code 系 Github 2018 年年度最火开源项目（[传送门](https://octoverse.github.com/projects#repositories)）。虽然社区的建设并不能代表绝对使用人数，但至少说明 VSC 真的是一个集百千宠爱于一身并不断成长的软件。

## 基本操作

### 快捷键

![4](https://img-blog.csdnimg.cn/2019032518150762.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

VSCode 首先是一个代码编辑器。因此介绍 VSCode 的时候自然应该从文本编辑功能开始。在使用文本编辑功能的时候，我们必然是重度依赖快捷键的。这是很显然的道理。所以我们通过介绍编辑快捷键介绍 VSCode 的文本编辑功能。

对于一个普通的文本而言，一般操作的最大粒度也就是行了。VSCode 支持的行操作主要包括：

- 行复制：`Ctrl + c`。当你没有选中行中任意字符时，复制出来的实际上是整行的内容；
- 行移动：`Alt + ↑/↓`。想整体移动一行怎么办？剪切粘贴？Alt 加方向键轻松移动一行！注意，如果你扩选多行时，相应地会移动多行；
- 行复制：`Shift + Alt + ↑/↓`。想在某行的基础上修改？复制粘贴？Shift 加 Alt 加方向键一步到位！

当然，对于代码文件，一般还会支持别的编辑操作，比如：

- 行注释：`Ctrl + /`；
- 块注释：`Alt + Shit + A`；
- 格式化：`Alt + Shit + F`。

> **注意：** 你能在 https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf 中找到本文提及的大多数快捷键。
 

![5](https://img-blog.csdnimg.cn/20190325181514464.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

对于缩进，VSCode 也有相应的快捷键。相信写 Python、markdown 的同学和有强迫症的同学都会无比感激这个设定。在 VSCode 中进行缩进操作很简单：

`Ctrl + ]/[` 即可完成整行的向右缩进一个单位 / 向左缩进一个单位。缩进完成后，会显示列对齐线帮助你把我缩进信息。同样地，选择多行的时候，可以同时对多行进行缩进。

![6](https://img-blog.csdnimg.cn/20190325181521534.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在开发中我们经常会有这样的需求：跳到工程中某个文件的某个位置之后再回到之前的位置。按照一般的操作逻辑，我们会思考之前打开了哪个文件，然后打开它，再找到之前的位置。这样的事情做一次两次还好，多了就让人心烦了。所以编辑器为什么不在光标位置跳动过大时记录光标的位置呢？这些位置形成队列，我们可以自由地从这些记录中进行跳转。

因着这样的需求而出现的功能就叫做「编辑点回退」。

我们使用 `Alt + → / ←` 回退到下一个 / 上一个编辑点。

![7](https://img-blog.csdnimg.cn/20190325181529355.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在阅读代码时，我们常常会希望跳过一些不需要关注的分支。遇到一些不规范的代码时，我们为了找到括号相应的括号可能会翻上百行。回想一下，我们是怎么找到另一个括号的？我们在点击其中一个括号时，代码编辑器会高亮相应的括号！

代码编辑器能够解析代码！它已经知道了另一个括号的位置！那它为什么不提供给我们接口，让我们直接跳到另一个括号那里去？？

这就是 `Ctrl + Shift + \` 了。点击一个括号，使用这个快捷键后，光标就能跳转到相应括号位置。

有时候你可能会想眼不见为净，要是编辑器能直接把这一堆东西折叠了该多好。

这就是 `Ctrl + Shift + [` 了。这个快捷键不记也没关系，看 PPT 里的第三张图，第一个花括号旁边是不是有个加/减号？点击它就可以打开/折叠代码区域。

![8](https://img-blog.csdnimg.cn/20190325181538141.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

发散一下思维，要是我们能一次性折叠所有代码就好了。这样我们无需任何插件，就能很快地了解代码的布局了。这些快捷键都在 PPT 里注明了，自己尝试一下就好了。

![9](https://img-blog.csdnimg.cn/20190325181547364.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

很多朋友更多时候是在用代码编辑器看代码，在看代码的时候怎么离得开查看定义呢？在 VSCode 里查看代码定义的方法主要有两种：

- `Alt + F12`：代码定义以浮窗的形式覆盖在当前页面上；
- `F12`：直接跳转到代码定义的位置。

实际上，我个人更喜欢的方式是后者，我宁愿在跳转后回退编辑点，也不愿意忍受浮窗带来的视觉割裂感。但为了说明前者具体是什么，我还是配了一张图。图中函数的定义以浮窗的形式出现了在函数下方，并标注了位置。

> **注意：** 很多朋友在阅读 c/c++ 代码时会觉得 VSCode 的代码跳转功能十分垃圾，找变量函数等的定义位置时瞎鸡儿跳。其实你可能冤枉它了。在默认情况下，当 include 头文件失败时，VSCode 的代码感知引擎会自动切换到「上下文无关」的模式。这在某些情况下十分有用，但如果你不知道这一点，那么代码跳转的结果可能会令你十分困惑。
> 
> 若要阻止代码感知引擎切换到「上下文无关」的模式，修改扩展的设置即可。如 PPT 所示，你只需在设置中关闭 `C_Cpp: Intelli Sense Engine Fallback` 即可。修改设置的方法见后文。

### 列选、检索与替换

![10](https://img-blog.csdnimg.cn/20190325181603438.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

提到现代文本编辑器，怎么能不提到列选呢？列选是为了解决多行编辑的问题。列选的方式的打开方式有多种：

- 鼠标中键框选：摁住鼠标左键画矩形，矩形相交和包含的字符均被选中；
- `Alt + Shit + 左键`：一定程度上等同上一条，区别在于初始光标位置作为矩形的一角，而点击位置作为矩形的对角；
- `Alt + 左键`：实际上这个才是我们最常使用的方式，它提供了最大的灵活性。长按 Alt 后，每个点击的位置都是编辑点。

值得一提的是，这个功能可以搭配 Insert Numbers 插件。这个插件可以在选中的位置上生成格式化的字符串组。如上图所示，我们在一份代码的 19~22 行分别插入了 0~3。具体怎么安装插件，我觉得 VSCode 的插件功能做得很棒，不介绍大家都懂。唯一需要注意的是，有一些特殊的插件，需要额外安装一些工具包，然后在 VSCode 的插件配置里给定路径。相信有 Linux 开发经验的朋友都不会对此感到陌生。

![11](https://img-blog.csdnimg.cn/20190325181613493.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

好了，终于到了使用频率爆炸的检索与替换了。与最基本的检索替换相比，VSCode 支持正则查找/替换，相信用过的人都懂得它的好。图中我们给出了一个例子，利用正则表达式查找所有以 CBGet 开头的符号名，并将前缀替换为 CBSet。那么如何在 VSCode 中使用查找/替换呢？

- `Ctrl + F / H` 文件内查找 / 替换；
- `Ctrl + Shift + F / H` 工作区内查找 / 替换。

前者很容易理解，后者是什么意思？这难道不就是全局替换吗？实际上在执行工作区内查找时，可以在「包含的文件」一项填写「文件夹的路径」，从而仅从某文件夹内查找。这在某些情况下，将会是很好用的技巧。

对了，匹配栏上有三个图标，激活后分别代表：

- 区分大小写；
- 开启全词匹配；
  - 基于 VSCode 的分词（试试 `Ctrl + Shift + → / ←` 选中一个单词）技术，匹配项应能与一个单词完全匹配；
  - VSCode 的分词技术目前仍十分基本，大概原理是将半角符号处理为单词的边界，所以并不支持中文的分词。
- 开启正则表达式。

![12](https://img-blog.csdnimg.cn/20190325181621200.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

比较神奇的是 VSCode 还支持代码的运行和调试。由于调试是 VSCode 左侧栏提供的四大功能之一，因此这里把它作为了 VSCode 中的基本操作。相关的快捷键有：

- F5 运行；
- Ctrl + Shift + D 打开调试面板。调试面板就是图中左边的那一栏。

在 PPT 的例子中，我已经配置好了 Python 的路径，并在一个 python 文件的第二行加了断点。断点的添加方式十分简单，和大多数 IDE 相似地，在行号坐标的空列点一下就好了。运行之后所有变量都被打印了出来，并且提供了控制调试进度的浮窗。

### 终端

![13](https://img-blog.csdnimg.cn/20190325181630878.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

终于！终于到终端了！在我心里，终端完全是化腐朽为神奇的东西！恩，腐朽的是 WSL。单独的 WSL（Windows Subsystem for Linux）似乎并没有什么存在的必要（但是还是可以用 VS2017 + WSL 调试 Linux 代码，[传送门](https://blog.csdn.net/maokelong95/article/details/64523303)），如有可能，我更愿意选择 xshell。毕竟每次光 `cd /mnt/d/xxx` 进入当前目录就能阻止我打开 WSL 了。但 VSCode 的终端可以配置成 WSL 后事情变得有些不一样了。

举一个很简单的例子，我在 VSCode 里唤起终端（配制成 WSL 了），编译并运行当前编辑的 C 文件。终端打开时的当前目录就是工作区目录！当 VSCode 打开目录时，这个目录就称为工作区。不仅仅是编译，make、ping、gdb、git，想用什么用什么，就很开心了。

默认情况下 VSCode 唤起的终端是 Poweshell，因为 WSL 是需要用户安装的，而 Powershell 则是每个 windows10 用户都有的。将内置终端修改为 WSL 的方法是将设置修改成：`"terminal.integrated.shell.windows": "C:\\WINDOWS\\System32\\wsl.exe"`。

修改设置的方法依然见后文。

## 高级操作

### 个性化

![14](https://img-blog.csdnimg.cn/2019032518163956.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![15](https://img-blog.csdnimg.cn/20190325181647904.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

### Diff

![16](https://img-blog.csdnimg.cn/20190325181655115.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![17](https://img-blog.csdnimg.cn/20190325181703711.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

### 命令窗口

![18](https://img-blog.csdnimg.cn/20190325181710513.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

## 进阶操作

### 多人协作

![19](https://img-blog.csdnimg.cn/20190325181718384.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

### 写作

![20](https://img-blog.csdnimg.cn/20190325181726688.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![21](https://img-blog.csdnimg.cn/20190325181734970.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
![22](https://img-blog.csdnimg.cn/20190325181741483.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

### 作为 XFtp 默认编辑器

![23](https://img-blog.csdnimg.cn/20190325181748799.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
