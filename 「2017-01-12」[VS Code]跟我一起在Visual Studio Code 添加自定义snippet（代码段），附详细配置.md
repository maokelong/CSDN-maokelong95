

# Add code snippets for CLANG in VS Code

> **日志**：
> 1. 2019.01.19 VSCode 自 v1.30 起，开始支持注释变量（comment variables），这些变量会随着当前语言而进行适应；自 v1.28 起，开始支持工作目录专属代码片（Project level snippets）及多前缀（multiple prefixes）特性。本次更新即旨于介绍这些新特性。另外笔者觉得中英文混杂的排版很扎眼睛，所以移除了大量中英文混杂的语句；
> 1. 2018.07.16 VSCode 自 v1.25 起，开始支持占位符转换（placeholder transformations）了，其用于在进行占位符跳转时（1→2）对当前占位符（1）适用正则替换。新特性听起来和变量转换很像，它们的区别在于占位符转换适用于占位符，而变量转换适用于变量。前者更灵活，后者更省心。本次更新即旨于介绍这个新特性，并再次对排版进行适应性调整；
> 1. 2018.05.13 VSCode 自 v1.20 起，开始支持更多变量，其用于读取剪贴板内容及插入当前日期。本次更新即旨于介绍这些新变量。此外，本次更新还移除了一些废话，并对排版进行了微调；
> 1. 2017.10.11 VSCode 自 v1.17 起，其代码片引擎开始支持变量转换（variable transformations）特性，变量的值可以经过格式化处理后，再插入预定的位置。这是一个很强大的特性；自 v1.15 起，开始支持 Choice 了。本次更新即旨于介绍这些新特性。

> **推广**：
> * [「VS Code」Visual Studio Code 菜鸟教程：从入门到精通](https://blog.csdn.net/maokelong95/article/details/88805589)。你能找到的最好的 VSC 教程。
> * [「VS Code」如何在 Visual Studio Code 中通过跳板机连接远程服务器：Remote-SSH 篇](https://blog.csdn.net/maokelong95/article/details/91801944)。你能找到的最好的 VSC  SSH 教程。

---

> **前记**：今天试着用了下 Atom，发现 Atom 居然预装了 CLANG 的 snippets，而且远比 VSCode 的已有拓展「C/C++ Snippets」中的丰富！身为 VSCode 的死忠粉，我决定立马把 Atom 的 C snippets 搬到 VSCode 上来。

---

[toc]
既然你点开了这个页面，那就说明要么你不知道 VSCode 上已有拓展「C/C++ Snippets」，要么你对这个拓展不甚满意。对于后者，本文将为你介绍如何在 VSCode 上设置 snippets，并为你提供一套可以直接用的 C 语言 snippets。

## 1. 代码片简介

snippet[ˈsnɪpɪt]，或者说「code snippet」，也即**代码片**，指的是能够帮助输入重复代码模式，比如循环或条件语句，的**模板**。通过 snippet ，我们仅仅输入一小段字符串，就可以在代码片引擎的帮助下，生成预定义的模板代码，接着我们还可以通过在预定义的光标位置之间跳转，来快速补全模板。

当然，看图更易懂。下图将 `aja` 补全为 JQuery 的 ajax() 方法，并通过光标的跳转，快速补全了待填键值对：

![这里写图片描述](https://img-blog.csdn.net/20171011222545141?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2. 代码片配置流程

首先，进入代码片设置文件，这里提供了三种方法：

- 通过快捷键「Ctrl +  Shift + P」打开**命令窗口**（All Command Window），输入「snippet」，点选「首选项：配置用户代码片片段」；
- 点击界面最左侧竖栏（也即活动栏）最下方的**齿轮**按钮，在弹出来的菜单中点选「用户代码片段」；
- 按下「Alt」键切换**菜单栏**，通过文件 > 首选项 > 用户代码片段；

接着，在设置文件里补全代码片。以 C 语言为例，选中后你将打开一个设置文件，c.json，在文件头部你会看见一个注释，这其实是一个示例和对它的介绍。你可以试着将第 7~14 行反注释掉（选中后 Ctrl + "/"），从而尝试使用它。了解过「json」就不会对此感到奇怪。

```json
{
	// Place your snippets for c here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	"Print to console": {
		"prefix": "log",
		"body": [
			"console.log('$1');",
			"$2"
		],
		"description": "Log output to console"
	}
}
```
 
这个示例中定义了一个描述为「Print to console」的代码片，其功能为：在 IntelliSense 中输入 `log` 并选中对应代码片后，可将原文本替换为 `console.log('');`。效果预览如下：

![log](https://img-blog.csdn.net/20170112145950068?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 3. 代码片详细介绍

### 3.1 语法结构

在第二章中，你已经接触到了代码片最简单的功能，而 VSCode 的代码片引擎所能做的远不止这些。本章将以官方教程[^vscode]为本，详细地为大家介绍代码片。

代码片由四部分组成：

1. prefix：前缀。代码片从 IntelliSense 中呼出的「关键字」;
   > 注：支持 N:1，数组中的每一项都能作为本条代码片的前缀。
1. scope: 域。代码片适用的「语言模式」；
   > 注：可选，但只有「全局代码片」才能使用。不填代表适用于所有语言模式。
1. body：主体。代码片的「布局与控制」；
   > 注：每个字符串表示一行。
1. description：描述。代码片在 IntelliSense 中的「介绍」。
   > 注：可选。未定义的情况下直接显示对象名，上例中将显示 `Print to console`。

### 3.2 Prefix 部分

前缀部分没有什么好介绍的，唯一值得注意的是，前缀支持 N:1，也即允许多条前缀对应同一条代码片。在使用时，只需将前缀定义为数组即可，数组中的每一个前缀都能对应本代码片。下面就是一个很简单的示例。

```json
{
  "prefix": [ "header", "stub", "copyright"],
  "body": "Copyright. Foo Corp 2028",
  "description": "Adds copyright...",
  "scope": "javascript,typescript"
}
```

### 3.3 Scope 部分

前缀部分没有什么好介绍的，不过在引入了域的概念之后，会不由自主地想起一些问题，比如如何让同一条代码片根据语言进行微调，从而取得良好的可移植性。一个很容易想到的例子就是注释风格了。

某公司希望所有代码文件的头部都有公司的版权声明，但 python 风格的注释是 `#` 而 C 风格的注释是 `//`，在每个语言的设置文件下都定义类似但注释风格不同的代码段显然会引入巨大的冗余。对此，VSCode 提供的解决方案是提供一些在不同语言下表现不同的变量。下一小节中已经更新了相关介绍。

### 3.4 Body 部分

#### 3.4.1 基本结构

Body 部分可以使用特殊语法结构，来控制光标和要插入的文本，其支持的基本结构如下：

 - **Tabstops**：制表符
     
   用「Tabstops」可以让编辑器的指针在 snippet 内跳转。使用 `$1`，`$2` 等指定光标位置。这些数字指定了光标跳转的顺序。特别地，`$0`表示最终光标位置。相同序号的「Tabstops」被链接在一起，将会同步更新，比如下列用于生成头文件封装的 snippet 被替换到编辑器上时，光标就将同时出现在所有`$1`位置。
   ```json
    "#ifndef $1"
    "#define $1"
    "#end // $1"
    ```
 - **Placeholders**：占位符
   
    「Placeholder」是带有默认值的「Tabstops」，如`${1：foo}`。「placeholder」文本将被插入「Tabstops」位置，并在跳转时被全选，以方便修改。占位符还可以嵌套，例如`${1:another ${2:placeholder}}`。
    
    比如，结构体的代码片主体可以这样写：

    ```
    struct ${1:name_t} {\n\t$2\n};
    ```
    作为「Placeholder」的`name_t`一方面可以提供默认的结构名称，另一方面可以作为输入的提示。
 - **Choice**：可选项
    
    「Choice」是提供可选值的「Placeholder」。其语法为一系列用逗号隔开，并最终被两个竖线圈起来的枚举值，比如 `${1|one,two,three|}`。当光标跳转到该位置的时候，用户将会被提供多个值（one 或 two 或 three）以供选择。

 - **Variables**：变量

    使用`$name`或`${name:default}`可以插入变量的值。当变量未赋值时（如），将插入其缺省值或空字符串。 当`varibale`未知（即，其名称未定义）时，将插入变量的名称，并将其转换为「Placeholder」。可以使用的「Variable」如下：

    - `TM_SELECTED_TEXT`：当前选定的文本或空字符串；
      注：选定后通过在命令窗口点选「插入代码片段」插入。
    - `TM_CURRENT_LINE`：当前行的内容；
    - `TM_CURRENT_WORD`：光标所处单词或空字符串
       注：所谓光标一般为文本输入处那条闪来闪去的竖线，该项可定制。单词使用 VSCode 选词（Word Wrap）器选择。你最好只用它选择英文单词，因为这个选择器明显没有针对宽字符优化过，它甚至无法识别宽字符的标点符号。
    - `TM_LINE_INDEX`：行号（从零开始）；
    - `TM_LINE_NUMBER`：行号（从一开始）；
    - `TM_FILENAME`：当前文档的文件名；
    - `TM_FILENAME_BASE`：当前文档的文件名（不含后缀名）；
    - `TM_DIRECTORY`：当前文档所在目录；
    - `TM_FILEPATH`：当前文档的完整文件路径；
    - `CLIPBOARD`：当前剪贴板中内容。

    此外，还有一些用于插入当前时间的变量，这里单独列出：
    
    - `CURRENT_YEAR`: 当前年份；
    - `CURRENT_YEAR_SHORT`: 当前年份的后两位；
    - `CURRENT_MONTH`: 格式化为两位数字的当前月份，如 02；
    - `CURRENT_MONTH_NAME`: 当前月份的全称，如 July；
    - `CURRENT_MONTH_NAME_SHORT`: 当前月份的简称，如 Jul；
    - `CURRENT_DATE`: 当天月份第几天；
    - `CURRENT_DAY_NAME`: 当天周几，如 Monday；
    - `CURRENT_DAY_NAME_SHORT`: 当天周几的简称，如 Mon；
    - `CURRENT_HOUR`: 当前小时（24 小时制）；
    - `CURRENT_MINUTE`: 当前分钟；
    - `CURRENT_SECOND`: 当前秒数。

    此外，还有一些用于插入行/块注释的变量，其将根据当前文件的语言模式自动调整：

    - `BLOCK_COMMENT_START` 块注释上半段，输出示例: 
       - PHP: `/*`
       - HTML: `<!--`
    - `BLOCK_COMMENT_END` 块注释下半段，输出示例: 
       - PHP: `*/`
       - HTML: `-->`
    - `LINE_COMMENT` 行注释，输出示例: 
       - PHP: `//`
       - HTML: `<!-- -->`

> 注：这些都是变量名，不是宏，在实际使用的时要加上 `$` 符。

#### 3.4.2 变量转换
    
变量转换可将变量的值格式化处理后插入预定的位置。

##### 语法结构

我们可以通过 `${var_name/regular_expression/format_string/options}` 插入格式化后的代码片。显然，「variable transformations」由 4 部分构成：
        
1. `var_name`：变量名；
2. `regular_expression`：正则表达式；
3. `format_string`：格式串；
4. `options`：正则表达式匹配选项。
    
其中正则表达式的写法和匹配选项部分不在本篇博文的讲解范围之内，具体内容请分别参考 javascript 有关 `RegExp(pattern [, flags])` 构造函数中的 `pattern` 及 `flags` 参数项的说明[^jsexp]。
    
本文只对 `format_string` 部分进行详细介绍。

##### format_string 部分
    
根据其 EBNF 范式，我们可以知道 `format_string` 其实是 `format` 或 `text` 的**线性组合**：
    
- `text`：也即没有任何作用的普通文本，你甚至可以使用汉字；
- `format`：格式串，分为 7 种：
  - `$sn`：表示插入匹配项；
  - `${sn}`：同 `$sn`；
  - `${sn:/upcase}` 或 `${sn:/downcase}` 或 `${sn:/capitalize}`：表示将匹配项变更为「所有字母均大写/所有字母均小写/首字母大写其余小写」后，插入；
  - `${sn:+if}`：表示当匹配成功时，并且捕捉括号捕捉特定序号的捕捉项成功时，在捕捉项位置插入「if」所述语句；
  - `${sn:?if:else}`：表示当匹配成功，并且捕捉括号捕捉特定序号的捕捉项成功时，在捕捉项位置插入「if」所述语句；否则当匹配成功，但当捕捉括号捕捉特定序号的捕捉项失败时，在捕捉项位置插入「else」所述语句；
  - `${sn:-else}`：表示当匹配成功，但当捕捉括号捕捉特定序号的捕捉项失败时，在捕捉项位置插入「else」所述语句；
  - `${sn:else}`：同 `${sn:-else}`。
    
`format` 的后三条理解起来可能比较困难。这里我们以倒数第三条为例进行说明。假设我们有一个「make.c」文件，我们有这么一条 snippet: `"body": "${TM_FILENAME/make.c(pp|\+\+)?/${1:?c++:clang}/}"`。整个模式串匹配成功，但是捕捉括号**捕捉**后缀名中的 pp 或 ++ 失败，因此判断条件在捕捉括号的位置插入**捕捉**失败时应插入的字符串，也即「clang」。
    
> **注**：
>
> - 其中 `sn` 表示捕捉项的序号
> - 其中 `if` 表示捕捉项捕捉成功时替换的文本
> - 其中 `else` 表示捕捉项捕失败时替换的文本

##### 案例分析

下面笔者再介绍一个简单的例子，帮助大家理解「variable transformations」。

假设有一个名为「make.c」的文件中，并且我们已经定义如下 snippet。

```json
"#ifndef HEADER … #define … #endif":{
"prefix": "defheader",
"body": "#ifndef ${1:${TM_FILENAME/(.*)\\.C$/${1:/upcase}_H/i}} \n#define $1 \n${2:header content}\n#endif\t// $1"
}
```
这段 snippet 将生成下图所示代码：
    
![这里写图片描述](https://img-blog.csdn.net/20171011222459311?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    
其中最复杂的模式为：`${1:${TM_FILENAME/(.*)\\.C$/${1:/upcase}_H/i}}`，我们将之拆解为如下五部分：
 1. `${1:...}`：嵌套的 `placeholder`；
 1. `${TM_FILENAME/.../.../.}`：「variable transformations」中的「var_name」，表示带后缀的文件名；
 1. `${.../(.*)\\.C$/.../.}`：「variable transformations」中的「regular_expression」，表示匹配任意以「.C」为后缀的字符串；
 1. `${.../.../${1:/upcase}_H/.}}`：「variable transformations」中的「format_string」，表示将第一个捕捉项替换为大写的，并添加「_H」的后缀；
 1. `${.../.../.../i}`：「variable transformations」中的「options」，表示无视大小写。
    

#### 3.4.3 占位符转换

##### 语法结构

我们可以通过 `${int/regular_expression/format_string/options}` 插入格式化后的代码片。显然，与变量转换，「placeholder transformations」也由 4 部分构成：
        
1. `int`：占位符相应光标序号；
2. `regular_expression`：正则表达式；
3. `format_string`：格式串；
4. `options`：正则表达式匹配选项。

上述全部内容我们都在前文介绍过了，因此此处不做赘述。我们唯一需要关注的是转换**触发的时机**：占位符转换将在进行占位符跳转（假设 1→2）的时候自动适用到当前占位符（1）。

##### 案例分析

假设我们已经这样的 Snippet：

```json
"HelloWorld": {
  "prefix": "say_hello",
  "body": "${1} ${2} -> ${1/Hello/Hallo/} ${2/World/Welt/}"
}
```

那么我们在两个制表位同时键入 Hello 并跳转的时候，第一个制表位依然保持 Hello 不变，而第二个制表位（占位符）被替换为 Hallo。键入 Welt 亦然。效果图：

![Placeholder Transformations](https://img-blog.csdn.net/20180716103448157?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 4. 代码片语法定义

官网给出了 snippet 的 EBNF 范式的正则文法，注意，作普通字符使用时，`$` , `}` 和 `\` 可使用 `\`（反斜杠）转义。

```
any         ::= tabstop | placeholder | choice | variable | text
tabstop     ::= '$' int
                | '${' int '}'
                | '${' int  transform '}'
placeholder ::= '${' int ':' any '}'
choice      ::= '${' int '|' text (',' text)* '|}'
variable    ::= '$' var | '${' var '}'
                | '${' var ':' any '}'
                | '${' var transform '}'
transform   ::= '/' regex '/' (format | text)+ '/' options
format      ::= '$' int | '${' int '}'
                | '${' int ':' '/upcase' | '/downcase' | '/capitalize' '}'
                | '${' int ':+' if '}'
                | '${' int ':?' if ':' else '}'
                | '${' int ':-' else '}' | '${' int ':' else '}'
regex       ::= JavaScript Regular Expression value (ctor-string)
options     ::= JavaScript Regular Expression option (ctor-options)
var         ::= [_a-zA-Z] [_a-zA-Z0-9]*
int         ::= [0-9]+
text        ::= .*
```

## 5. 代码片设置文件

我们在第二章中就已经理解了代码片设置文件的概念，但当时这并不是我们的核心关注点。现在，你已经对代码片有了深刻的理解了，你可能会面临着这样新的需求，也即只为某工程，某语言，或所有语言单独配置代码片的进阶要求。这里就将为你这样的需求提供解决思路。

### 5.1 Project level snippets

当你使用 VSCode 打开一个文件夹时，这个文件夹就成了所谓的 Project 或 Workspace。所以，如果你希望单独为某工程配置 snippet 时，你首先应该打开一个目录。在打开目录之后，你只需按照第二章中介绍的方法，在进入代码片设置文件时点选「新建"xxx"文件夹的代码片段文件」。VSCode 会使用 GUI 引导着你在当前工程下的「.vscode」中新建一个「*.code-snippets」的文件，这就是当前工作目录的设置文件。

在呼出代码片的时候，IntelliSense 会注明哪些代码片是「Workspace Snippet」。如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190119210632163.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

## 6. 一些建议

默认情况下 snippet 在 IntelliSense 中的显示优先级并不高，而且在 IntelliSense 中选择相应 snippet 需要按「enter」键，这对于手指短的人来说并不是什么很好的体验。

所幸，VSCode 意识到了这一点，并为我们提供了改进的方式。我们可以在 VSCode 的**用户设置**（「Ctrl+P」在输入框中写「user settings」后点选）中，检索**代码片**，然后根据提示修改代码片的相关设置。

我们可以设置在 IntelliSense 中优先显示代码片，并可以通过「TAB」补全。

![修改设置](https://img-blog.csdn.net/20170112155152376?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

修改后设置文件中会多出这两行：
```
"editor.snippetSuggestions": "top",
"editor.tabCompletion": true
```

> 注：v1.28 之后，`editor.tabCompletion` 得到了增强。现在 TAB 能补全所有前缀了，而非仅 snippet。另外，在插入非代码片的前缀 **之后**，可以使用 TAB 向下切换别的建议，或使用 Shit + TAB 向上切换。

## 附录

说好的附录。

另，我对 Atom 的 C snippet[^atom] 作了部分修改，使之更适合我的习惯，若有兴致你可自行修改，反正也不难。

```json
{
/*
	 // Place your snippets for C here. Each snippet is defined under a snippet name and has a prefix, body and 
	 // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	 // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	 // same ids are connected.
	 // Example:
	 "Print to console": {
		"prefix": "log",,
		"body": [
			"console.log('$1');",
			"$2"
		],
		"description": "Log output to console"
	}
*/
  "#ifndef … #define … #endif":{
    "prefix": "def",
    "body": "#ifndef ${1:SYMBOL}\n#define $1 ${2:value}\n#endif\t// ${1:SYMBOL}"
  },
  "#include <>":{
    "prefix": "Inc",
    "body": "#include <${1:.h}>"
  },
  "#include \"\"":{
    "prefix": "inc",
    "body": "#include \"${1:.h}\""
  },
  "#pragma mark":{
    "prefix": "mark",
    "body": "#if 0\n${1:#pragma mark -\n}#pragma mark $2\n#endif\n\n$0"
  },
  "main()":{
    "prefix": "main",
    "body": "int main(int argc, char const *argv[]) {\n\t$1\n\treturn 0;\n}"
  },
  "For Loop":{
    "prefix": "for",
    "body": "for (${1:i} = 0; ${1:i} < ${2:count}; ${1:i}${3:++}) {\n\t$4\n}"
  },
  "Define and For Loop":{
    "prefix": "dfor",
    "body": "size_t ${1:i};\nfor (${1:i} = ${2:0}; ${1:i} < ${3:count}; ${1:i}${4:++}) {\n\t$5\n}"
  },

  "Header Include-Guard":{
    "prefix": "once",
    "body": "#ifndef ${1:SYMBOL}\n#define $1\n\n${2}\n\n#endif /* end of include guard: $1 */\n"
  },
  "Typedef":{
    "prefix": "td",
    "body": "typedef ${1:int} ${2:MyCustomType};"
  },
  "Typedef Struct":{
    "prefix": "tst",
    "body": "typedef struct ${1:StructName} {\n\t$2\n}${3:MyCustomType};"
  },
  "Do While Loop":{
    "prefix": "do",
    "body": "do {\n\t$0\n} while($1);"
  },
  "While Loop":{
    "prefix": "while",
    "body": "while ($1) {\n\t$2\n}"
  },
  "fprintf":{
    "prefix": "fprintf",
    "body": "fprintf(${1:stderr}, \"${2:%s}\\\\n\", $3);$4"
  },
  "If Condition":{
    "prefix": "if",
    "body": "if ($1) {\n\t$2\n}"
  },
  "If Else":{
    "prefix": "ife",
    "body": "if ($1) {\n\t$2\n} else {\n\t$3\n}"
  },
  "If ElseIf":{
    "prefix": "iff",
    "body": "if ($1) {\n\t$2\n} else if ($3) {\n\t$4\n}"
  },
  "If ElseIf Else":{
    "prefix": "iffe",
    "body": "if ($1) {\n\t$2\n} else if ($3) {\n\t$4\n} else {\n\t$5\n}"
  },
  "Switch Statement":{
    "prefix": "sw",
    "body": "switch ($1) {\n$2default:\n\t${3:break;}\n}$0"
  },
  "case break":{
    "prefix": "cs",
    "body": "case $1:\n\t$2\n\tbreak;\n$0"
  },
  "printf":{
    "prefix": "printf",
    "body": "printf(\"${1:%s }\\n\", $2);$3"
  },
  "scanf":{
    "prefix": "scanf",
    "body": "scanf(\"${1:%s}\\n\", $2);$3"
  },
  "Struct":{
    "prefix": "st",
    "body": "struct ${1:name_t} {\n\t$2\n};"
  },
  "void":{
    "prefix": "void",
    "body": "void ${1:name}($2) {\n\t$3\n}"
  },
  "any function":{
    "prefix": "func",
    "body": "${1:int} ${2:name}($3) {\n\t$5\n\treturn ${4:0};\n}"
  },
  "write file":{
    "prefix": "wf",
    "body": "FILE *${1:fp};\n${1:fp} = fopen (\"${2:filename.txt}\",\"w\");\nif (${1:fp}!=NULL)\n{\n\tfprintf(${1:fp},\"${3:Some String\\\\n}\");\n\tfclose (${1:fp});\n}"
  },
  "read file":{
    "prefix": "rf",
    "body": "FILE *${1:fp};\n${1:fp} = fopen (\"${2:filename.txt}\",\"r\");\nif (${1:fp}!=NULL)\n{\n\tfscanf(${1:fp},\"${3:Some String\\\\n}\", ${3:&var});\n\tfclose (${1:fp});\n}",
    "description": "read file opeartion including fopen, fscanf and fclose."
  }
}

```
[^vscode]:https://code.visualstudio.com/Docs/customization/userdefinedsnippets

[^docatgithub]:https://github.com/Microsoft/vscode/blob/fd8eb3bd8a9f032ec0051e8bb4f1d8eb6dcd3f4b/src/vs/editor/contrib/snippet/browser/snippet.md

[^sourcecode]:https://github.com/Microsoft/vscode/blob/fd8eb3bd8a9f032ec0051e8bb4f1d8eb6dcd3f4b/src/vs/editor/contrib/snippet/browser/snippetParser.ts#L963

[^atom]:https://github.com/atom/language-c/blob/master/snippets/language-c.cson

[^jsexp]:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp
