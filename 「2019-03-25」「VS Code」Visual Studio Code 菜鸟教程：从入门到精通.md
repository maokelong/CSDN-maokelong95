# 「VS Code」Visual Studio Code 教程：从入门到精通

> **日志**：
>
> 1. 「2019-05-26」补全了全文的剩余部分，并引入了对 Remote Development 插件的介绍；
> 1. 「2019-03-25」提交了本文的 PPT 框架。

---

> **作者按**：Visual Studio Code，或简称为 VSCode，是我最喜欢的代码编辑器，我希望能有更多人享受到 VSCode 的便捷与强大。在希望学习一门新技术的时候，我们往往会想着去搜索一份教程。根据二八定律，我们往往只需了解一门技术的部分最常用的功能，就足以应对大多数开发场景，因此介绍最常见功能的教程，而非介绍全部特性的手册，成了我们的学习技术的首选。然而已存教程不足以成为我们了解 VSCode 的窗口，很多具有广泛使用场景的功能，比如命令窗口和终端，这些教程都没有涉猎。该种情况促成了本文的诞生。本文的愿景是，哪怕是从未使用过 VSCode 的读者，也能在阅读本文后便精通 VSCode 的使用。
>
> - 本博客已经发布了大量优质教程，其中包括阅读量超过五万的 [《[VS Code]跟我一起在Visual Studio Code 添加自定义snippet（代码段），附详细配置》](https://blog.csdn.net/maokelong95/article/details/54379046)。
>
> - 本博客所有文本及胶片均公开在了 [Github](https://github.com/maokelong/CSDN-maokelong95) 上，如需转载，注明出处即可。如若建议、勘误等，发起 Issue 或在下方评论均可。

---

[toc]

## 大纲

![2](https://img-blog.csdnimg.cn/201905261608197.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

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

当然，对于受支持的代码文件，一般还会支持别的操作，比如：

- 行注释：`Ctrl + /`；
- 块注释：`Alt + Shit + A`；
- 格式化：`Alt + Shit + F`。

> **注意：** 你能在 https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf 中找到本文提及的大多数快捷键。

![5](https://img-blog.csdnimg.cn/20190325181514464.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

对于缩进，VSCode 也有相应的快捷键。相信写 Python、markdown 的同学和有强迫症的同学都会无比感激这个设定。在 VSCode 中进行缩进操作很简单：

`Ctrl + ]/[` 即可完成整行的向右缩进一个单位 / 向左缩进一个单位。缩进完成后，会显示列对齐线帮助你把握缩进信息。同样地，选择多行的时候，可以同时对多行进行缩进。

![6](https://img-blog.csdnimg.cn/20190325181521534.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在开发中我们经常会有这样的需求：跳到工程中某个文件的某个位置之后再回到之前的位置。按照一般的操作逻辑，我们会思考之前打开了哪个文件，然后打开它，再找到之前的位置。这样的事情做一次两次还好，多了就让人心烦了。所以编辑器为什么不在光标位置「大幅度改变」时记录光标的位置呢？这些位置记录成队列，让我们可以自由地从从这些位置中切换。

因着这样的需求而出现的功能就叫做「编辑点回退」，上面所说的位置就是「编辑点」。

我们使用 `Alt + → / ←` 切换到下一个 / 上一个编辑点。

![7](https://img-blog.csdnimg.cn/20190325181529355.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在阅读代码时，我们常常会希望跳过一些不需要关注的分支。遇到一些不规范的代码时，我们为了找到相应的括号可能会翻上百、甚至上千行。回想一下，我们是怎么找到另一个括号的？我们在点击其中一个括号时，代码编辑器会高亮相应的括号！

代码编辑器能够解析代码！它已经知道了另一个括号的位置！那它为什么不提供给我们接口，让我们直接跳到另一个括号那里去？？

这就是 `Ctrl + Shift + \` 了。点击一个括号，在使用这个快捷键后，光标就能跳转到相应括号位置。

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

- 鼠标中键框选：摁住鼠标左键画矩形，矩形相交及包含的字符均被选中；
- `Alt + Shit + 左键`：一定程度上等同上一条，区别在于初始光标位置作为矩形的一角，而点击位置作为矩形的对角；
- `Alt + 左键`：实际上这个才是我们最常使用的方式，它提供了最大的灵活性。长按 Alt 后，每个点击的位置都是编辑点。注意，这些编辑点是有序的！

值得一提的是，这个功能可以搭配 Insert Numbers 插件。这个插件可以在选中的位置上生成格式化的字符串组。如上图所示，我们在一份代码的 19~22 行分别插入了 0~3。发散一下思维，我们怎么插入 2 3 1 0 这样的序列？依次让第四、三、一、二处成为编辑点就好了。

具体怎么安装插件这里，我觉得 VSCode 的插件功能做得很棒，不必介绍大家都应该懂。唯一需要注意的是，有一些特殊的插件，需要额外安装一些工具包，然后在 VSCode 的插件配置里给定路径。相信有 Linux 开发经验的朋友都不会对此感到陌生。

![11](https://img-blog.csdnimg.cn/20190325181613493.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

好了，终于到了使用频率爆炸的检索与替换了。与最基本的检索替换相比，VSCode 支持正则查找/替换，相信用过的人都懂得它的好。图中我们给出了一个例子，利用正则表达式查找所有以 CBGet 开头的符号名，并将前缀替换为 CBSet。那么如何在 VSCode 中使用查找/替换呢？

- `Ctrl + F / H` 文件内查找 / 替换；
- `Ctrl + Shift + F / H` 工作区内查找 / 替换。

前者很容易理解，后者是什么意思？这难道不就是全局替换吗？实际上在执行工作区内查找时，可以在「包含的文件」一项填写「文件夹的路径」，从而仅从某文件夹内查找。这在某些情况下，将会是很好用的技巧。

对了，匹配栏上有三个图标，激活后分别代表：

- 区分大小写；
- 开启全词匹配；
  - 基于 VSCode 的分词（试试 `Ctrl + Shift + → / ←` 选中一个单词）技术，匹配项应能与一个单词完全匹配；
  - VSCode 的分词技术目前仍十分基本，大概原理是将半角符号处理为单词的边界，所以并不支持中文的分词；
    - **我能做什么？** 为 ISSUE（[传送门](https://github.com/microsoft/vscode/issues/50045)） 点赞，提高问题的解决优先级。
- 开启正则表达式。

![12](https://img-blog.csdnimg.cn/20190325181621200.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

比较神奇的是 VSCode 还支持代码的运行和调试。由于调试是 VSCode 左侧栏提供的四大功能之一，因此这里把它作为了 VSCode 中的基本操作。相关的快捷键有：

- F5 运行；
- Ctrl + Shift + D 打开调试面板。调试面板就是图中左边的那一栏。

但快捷键不是使用这个特性的主要难点，主要难点在于环境的搭建。PPT 中以较为简单的 Python 为例，我已经预先配置好了 Python 的环境变量，并在一个 python 文件的第二行加了断点。断点的添加方式十分简单，和大多数 IDE 相似地，在行号左边的空列点一下就好了。运行之后所有变量都被打印了出来，并且提供了控制调试进度的浮窗。

### 终端

![13](https://img-blog.csdnimg.cn/20190325181630878.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

终于！终于到终端了！在我心里，终端完全是化腐朽为神奇的东西！恩，腐朽的是 WSL。单独的 WSL（Windows Subsystem for Linux）似乎并没有什么存在的必要（但是还是可以用 VS2017 + WSL 调试 Linux 代码，[传送门](https://blog.csdn.net/maokelong95/article/details/64523303)），如有可能，我更愿意选择 xshell。毕竟每次光 `cd /mnt/d/xxx` 进入当前目录就能阻止我打开 WSL 了。但 VSCode 的终端可以配置成 WSL 后事情变得有些不一样了。

举一个很简单的例子，我在 VSCode 里唤起终端（配制成 WSL 了），编译并运行当前编辑的 C 文件。终端打开时的当前目录就是工作区目录！当 VSCode 打开目录时，这个目录就称为工作区。不仅仅是编译，make、ping、gdb、git，想用什么用什么，就很开心了。

默认情况下 VSCode 唤起的终端是 Poweshell，这可能是因为 WSL 是需要用户安装的，而 Powershell 则是每个 windows10 用户都有的。将内置终端修改为 WSL 的方法是将设置修改成：`"terminal.integrated.shell.windows": "C:\\WINDOWS\\System32\\wsl.exe"`。

修改设置的方法依然见后文。

## 高级操作

### 个性化

![14](https://img-blog.csdnimg.cn/2019032518163956.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

几乎每个程序猿都有自己倾向的审美及编码风格。

宋体好看还是微软雅黑好看？用 TAB 还是空格？TAB 长两个字符还是四个字符？文件的换行符是 windows 风格的 \r\n 还是 *nix 风格的 \n？文件结尾要不要空新行？代码的格式化风格是用自己的格式化器，还是预设的 LLVM、Google、Chromium？这些都是可以定制的。

作为新手，可以通过 GUI 的 **导航栏 → 文件 → 首选项 → 设置** 找到修改这些设置的接口。不得不说这些年里 VSCode 的 GUI 做得越来越好了，早年 VSCode 还只能通过修改 json 文件进行设置，而且有些设置的含义只能靠名字猜。而现在 VSCode 已经提供了完备的 GUI，帮助用户点选了。

另外在使用任何一款软件时，我们往往都会试着找一下自己喜欢的皮肤或主题。VSCode 也不例外。我们主要关注两种主题：

* 颜色主题：有人喜欢暖色调（我最喜欢的就是 `Quiet Light` 主题）、有人喜欢冷色调、还有人喜欢暗色调，更有人不得不选择高对比度方案。这些当然是可以定制的；
* 文件图标主题：当我们在浏览一个陌生的项目时，我们可能想着要去找其中某种特定类型的文件，一般而言，我们的判断方式是用肉眼逐一搜索文件后缀，当然后面还会介绍更高级的方法。但文件图标可能是更好的方案。图形这种信息呈现方式能比文本提供更多的信息量，有利于快速检索。这同样是可以定制的。

需要定制这些主题的话，作为新手，我仍推荐 **导航栏 → 文件 → 首选项 → 设置 → 颜色主题 / 文件图标主题**。

![15](https://img-blog.csdnimg.cn/20190325181647904.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

相信大多数人都有过这样的编程体验，也即输入一小段我们字符串，然后这些字符可以展开成一大段模板。这样的特性称之为 snippet，也即代码段。作为新手，你使用别人提供好的 snippet 插件就好了。如果你实在想定制属于自己的 snippet 的话，可以参考笔者另外的博客（传送门：[[VS Code]跟我一起在Visual Studio Code 添加自定义snippet（代码段），附详细配置](https://blog.csdn.net/maokelong95/article/details/54379046)）。

### Diff

![16](https://img-blog.csdnimg.cn/20190325181655115.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

有时候我们会有这样的需求：手上拿到了两份相差无几的文件，想要了解到底改了哪些地方。VSCode 内置了很好的解决方案，称之为 diff。操作的方法也很简单，首先 **右键 → 选择进行比较** 选择要比较的基准文件，然后  **右键 → 与已选项目进行比较** 选择要比较的目标文件。然后就会在界面中展示两份文件的异同。不一样的地方会被高亮出来。

![17](https://img-blog.csdnimg.cn/20190325181703711.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

在更一般的情况下，我们会想与 git 仓储的 head 比较，对自己所作的修改形成较为全面的把握。这时候只需在活动栏（界面中最左边的竖栏）中点选源代码管理，并单选更改过的文件即可。只要你 PC 中安装过 git，并且工作区中包含仓储、你也曾进入过仓储所在文件夹，源代码管理就会自动启用。

### 命令窗口

![18](https://img-blog.csdnimg.cn/20190325181710513.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

这一部分是 VSCode 的精髓。可以说掌握了这个，才算真正掌握了 VSCode。众所周知，随着软件功能的增长，检索软件功能的接口成了必需品。VSCode 也提供了这样的接口，但更高级。

我们通过 `Ctrl + Shift + P` 打开命令窗口，命令窗口时根据首字符的不同，能提供截然不同的功能：

* `>`：默认接口。可匹配到并快捷执行 VSCode 提供的所有的接口。这些接口中包括自己安装的插件提供的接口；
* `:`：跳转到指定行号；
* `@`：跳转到指定符号；符号（Symbol），包括函数、变量、字段、结构等；
* `@:`：依然是跳转到指定符号，但区别在于所有符号归类显示；
* ：跳转到指定文件。没错，删除首字符，直接输入文件的名字！能直接跳转到文件！

## 进阶操作

### 多人协作

![19](https://img-blog.csdnimg.cn/20190325181718384.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

VSCode 提供了丰富的插件，这里介绍其中的部分。

VS Live Share，一个下载量接近千万的插件，能允许**实时地**同别人一同编辑及调试文件。与只注重结果的 Git 相比，VS Live Share 能展示同伴编辑的过程。

### 写作

![20](https://img-blog.csdnimg.cn/20190325181726688.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

LaTeX，可能是最流行的重量级文本标记语言了。对于很多科研工作者来说，LaTeX 可能是第一款让他们去思考，有哪些排版上的细节需要注意的工具。这里不介绍相关特性，也不介绍如何在 VSC 中编写 LaTeX，更多地，这里只是提供一种可能。

![21](https://img-blog.csdnimg.cn/20190325181734970.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

Markdown，可能是最流行的轻量级文本标记语言了。当大家都是 Git Noob 的时候，应该都接触过一个神奇的文件：readme.md。这也可能是大多数人与 markdown 的初次相会。同样地，这里不介绍相关特性，这里只是提供一种可能。

虽然 VSCode 内置了对 Markdown 的支持，直接点击文本右上角的「打开侧边预览」就可以查看 Markdown 渲染的模样。但这里还是建议诸位尝试一下一下 「Markdown Preview Enhanced」。推荐的理由在于，该插件能支持 TOC、脚注等扩展语法，而且渲染效果更好，行间距等处理得更好。

![22](https://img-blog.csdnimg.cn/20190325181741483.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

AsciiDoc 作为 LaTeX 及 Markdown 之间的一种文本标记语言，不似 LaTeX 过于繁琐，也不像 Markdown 一样过于简单。很多人可能只是想找一种能在 Git Readme 中插入目录并自动标注序号的方法，却意外地爱上了这个语言。同样地，这里不介绍相关特性，这里只是提供一种可能。

### 作为 XFtp 默认编辑器

![23](https://img-blog.csdnimg.cn/20190325181748799.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

VSCode 能替换成 XFtp 的默认编辑器，使得能基于 VSCode 强大的特性去编辑云端上的单个文件。文件一经保存即自动同步到云端上。设置方法很简单，**工具 → 选项 → 高级** 中，你很快就能找到相关选项。

### 直接开发云端项目
![24](https://img-blog.csdnimg.cn/20190613102111240.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

最后谈一下 VSCode 于 2019-05-02 发布的最新特性，Remote Development。通过 Remote Development 插件连接上云端，然后打开云端上的文件夹作为 workspace。作为结果，你无需将云端上的各种头文件和源码拷贝到本地上，就能使用包括 IntelliSense (completions)、code navigation 及 debugging 在内的各种功能。最后，我们还能使用编辑器内置的终端操纵云端，去做一些 build、release 这样的事情。笔者专门为此写了一篇图文并茂的教程，帮助你在最短时间内上手该插件。（传送门：[「VS Code」如何在 Visual Studio Code 中通过跳板机连接远程服务器：Remote-SSH 篇](https://blog.csdn.net/maokelong95/article/details/91801944)）
