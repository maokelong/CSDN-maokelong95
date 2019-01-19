# Intel PinTools安装使用简单教程

[toc]

> **日志**：
>
> - 2018-08-02：修正相关资源链接。现在 Intel 恢复对 Pintool 的维护了，经测试，Linux 4.15 内核可以使用 Pintool；
> - 2018-01-04：发布了第一个版本；

---

> **Note**：本文称 Instrumentation 为「插桩」。

## 安装测试 Pin

### About

有关资源及资料见下述链接：

- [中文介绍](https://huirong.github.io/2015/12/30/Intel-Pin-introduction/)
- [Intel 文档](https://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool)
- [下载地址](https://software.intel.com/en-us/articles/pin-a-binary-instrumentation-tool-downloads)

### 安装 Pin

依次输入如下指令完成 pin 的下载、解压：

```bash
wget http://software.intel.com/sites/landingpage/pintool/downloads/pin-2.14-71313-gcc.4.4.7-linux.tar.gz
tar -xzf pin-2.14-71313-gcc.4.4.7-linux.tar.gz
```

> **Note**：你的内核版本可能不受支持。

### 测试 Pin

我们可以 **先运行一个 demo** 进行测试。在这个例子中我们将生成一个名为 insmix 的工具，这个工具是用来统计应用程序执行的 x86 指令的数量的。通过如下指令打开 insmix 工具所在目录，并生成该工具：

```bash
cd pin-2.14-71313-gcc.4.4.7-linux/source/tools/Insmix/
make
```

如果执行成功的话，应该可以在 `source/tools/Insmix/obj-intel64/` 目录下找到文件`insmix.so`。此时我们返回到 pin 的根目录，然后输入如下指令将 insmix 工具附加在 ls 指令上并由 pin 运行 ls 指令：

```bash
cd ../../../
./pin.sh -t source/tools/Insmix/obj-intel64/insmix.so -- ls
```

运行结束后结果文件输出在 pin 根目录下，即 `insmix.out`。

## 自定义 PinTool

仅仅使用 Pin 平台提供的已有 PinTools 肯定无法满足我们所有的需求，有时甚至对已有 PinTool 进行自定义也无法满足我们的需求，这时候我们就需要自己自定义 PinTool。

下面就将介绍一种基于镜像的函数插桩式 PinTool 的代码结构及生成、使用方式。

### 完成 PinTool 源码

假定我们需要一种获取程序所有内存分配、释放情况的 PinTool。

我们先查找 pin 中已经提供的 pintools，可以发现 `source/tools/ManualExamples/malloctrace.cpp`这个 PinTool 完全可实现我们的需求。这个 PinTool 可以实现函数粒度的插桩并实现对 malloc 与 free 的简单分析：记录 malloc 的参数及返回值，记录 free 的参数。

该源码内容如下：

```C++
/*BEGIN_LEGAL 
Intel Open Source License

Copyright (c) 2002-2015 Intel Corporation. All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

Redistributions of source code must retain the above copyright notice,
this list of conditions and the following disclaimer.  Redistributions
in binary form must reproduce the above copyright notice, this list of
conditions and the following disclaimer in the documentation and/or
other materials provided with the distribution.  Neither the name of
the Intel Corporation nor the names of its contributors may be used to
endorse or promote products derived from this software without
specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE INTEL OR
ITS CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
END_LEGAL */

#include "pin.H"
#include <iostream>
#include <fstream>

/* ===================================================================== */
/* Names of malloc and free */
/* ===================================================================== */
#if defined(TARGET_MAC)
#define MALLOC "_malloc"
#define FREE "_free"
#else
#define MALLOC "malloc"
#define FREE "free"
#endif

/* ===================================================================== */
/* Global Variables */
/* ===================================================================== */

std::ofstream TraceFile;

/* ===================================================================== */
/* Commandline Switches */
/* ===================================================================== */

KNOB<string> KnobOutputFile(KNOB_MODE_WRITEONCE, "pintool",
    "o", "malloctrace.out", "specify trace file name");

/* ===================================================================== */


/* ===================================================================== */
/* Analysis routines                                                     */
/* ===================================================================== */

VOID Arg1Before(CHAR * name, ADDRINT size)
{
    TraceFile << name << "(" << size << ")" << endl;
}

VOID MallocAfter(ADDRINT ret)
{
    TraceFile << "  returns " << ret << endl;
}


/* ===================================================================== */
/* Instrumentation routines                                              */
/* ===================================================================== */

VOID Image(IMG img, VOID *v)
{
    // Instrument the malloc() and free() functions.  Print the input argument
    // of each malloc() or free(), and the return value of malloc().
    //
    //  Find the malloc() function.
    RTN mallocRtn = RTN_FindByName(img, MALLOC);
    if (RTN_Valid(mallocRtn))
    {
        RTN_Open(mallocRtn);

        // Instrument malloc() to print the input argument value and the return value.
        RTN_InsertCall(mallocRtn, IPOINT_BEFORE, (AFUNPTR)Arg1Before,
                       IARG_ADDRINT, MALLOC,
                       IARG_FUNCARG_ENTRYPOINT_VALUE, 0,
                       IARG_END);
        RTN_InsertCall(mallocRtn, IPOINT_AFTER, (AFUNPTR)MallocAfter,
                       IARG_FUNCRET_EXITPOINT_VALUE, IARG_END);

        RTN_Close(mallocRtn);
    }

    // Find the free() function.
    RTN freeRtn = RTN_FindByName(img, FREE);
    if (RTN_Valid(freeRtn))
    {
        RTN_Open(freeRtn);
        // Instrument free() to print the input argument value.
        RTN_InsertCall(freeRtn, IPOINT_BEFORE, (AFUNPTR)Arg1Before,
                       IARG_ADDRINT, FREE,
                       IARG_FUNCARG_ENTRYPOINT_VALUE, 0,
                       IARG_END);
        RTN_Close(freeRtn);
    }
}

/* ===================================================================== */

VOID Fini(INT32 code, VOID *v)
{
    TraceFile.close();
}

/* ===================================================================== */
/* Print Help Message                                                    */
/* ===================================================================== */

INT32 Usage()
{
    cerr << "This tool produces a trace of calls to malloc." << endl;
    cerr << endl << KNOB_BASE::StringKnobSummary() << endl;
    return -1;
}

/* ===================================================================== */
/* Main                                                                  */
/* ===================================================================== */

int main(int argc, char *argv[])
{
    // Initialize pin & symbol manager
    PIN_InitSymbols();
    if( PIN_Init(argc,argv) )
    {
        return Usage();
    }

    // Write to a file since cout and cerr maybe closed by the application
    TraceFile.open(KnobOutputFile.Value().c_str());
    TraceFile << hex;
    TraceFile.setf(ios::showbase);

    // Register Image to be called to instrument functions.
    IMG_AddInstrumentFunction(Image, 0);
    PIN_AddFiniFunction(Fini, 0);

    // Never returns
    PIN_StartProgram();

    return 0;
}

/* ===================================================================== */
/* eof */
/* ===================================================================== */
```

这里简单地介绍一下这个 PinTool 的生命周期：

![这里写图片描述](https://img-blog.csdn.net/20180716205403951?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

PinTool 入口位置为 `int main(int argc, char *argv[])` ，逐次通过 `PIN_InitSymbols()`、`PIN_Init(argc,argv)` 完成符号表管理器、PinTool 的初始化。没有初始化符号表管理器，接下来 PinTool 将无法识别挂载程序中的符号，如函数名。

在初始化完成之后，PinTool 打开由 KnobOutputFile 指定的追踪结果输出文件，通过 ofstream TraceFile 进行管理。对于整数将输出为十六进制并显示基数(0x)。

接着，PinTool 通过 `IMG_AddInstrumentFunction()` 注册 `Image()`，以向挂载程序中插入插桩函数。`Image()` 函数依次在挂载程序中查找 malloc 及 free 函数入口，分别通过 mallocRtn 及 freeRtn 记录查询结果。

在通过 `RTN_Open()` 打开 RTN 之后，即可通过 RTN_InsertCall 向相应函数插入插桩函数。以这段代码为例：

```C
RTN_InsertCall(mallocRtn, IPOINT_BEFORE, (AFUNPTR)Arg1Before,
                       IARG_ADDRINT, MALLOC,
                       IARG_FUNCARG_ENTRYPOINT_VALUE, 0,
                       IARG_END);
```

表示在 malloc 函数执行前执行 Arg1Before 函数，并依次为 Arg1Before 函数传递两个参数（IARG_END 标志着参数的告结）。IARG_ADDRINT 表示第一个函数是 IARG_ADDRINT 型的常量，其值是宏 MALLOC；IARG_FUNCARG_ENTRYPOINT_VALUE 表示第二个参数是仅在函数入口处有效的函数本身参数，其中接下来的 0 表示第一个参数。

之后，PinTool 通过 `PIN_AddFiniFunction()` 注册 `Fini()`，以在挂载程序结束后执行 `Fini()` 函数。

最后，PinTool 通过 `PIN_StartProgram()` 运行挂载程序。

### 生成 PinTool

[官方](https://software.intel.com/sites/landingpage/pintool/docs/65163/Pin/html/index.html#BUILDINGTOOLS)提供的有两种编写自定义pintool的方法：在开发包目录树下建立、在开发包目录树外建立。其中前者较为简单，我们采用这种方案。
	
首先进入根目录下的 `source/tools/MyPinTool`，我们向其中放入我们的 PinTool 源代码，假定其名字为 `myMallocTrace.cpp`。

然后在该目录下输入 `make obj-intel64/myMallocTrace.so`，如果语法没有问题的话，我们就会在 `source/tools/MyPinTool/obj-intel64/` 下找到 `myMallocTrace.so` 文件，这个文件就是我们的自定义 PinTool。

> **Note**：源代码的命名及生成的 PinTool 所在目录的名称不是固定的，只要 .cpp 和 .so 的文件名一致就行 。

### 使用 PinTool

根据 `./pin.sh --help` 的反馈信息：

```bash
Usage: pin [OPTION] [-t <tool> [<toolargs>]] -- <command line>
```

我们知道了 PinTool 的使用方法。此处不再赘述。

```bash
$HOME/src/pin-2.14-71313-gcc.4.4.7-linux/pin.sh -t \
$HOME/src/pin-2.14-71313-gcc.4.4.7-linux/source/tools/MyPinTool/obj-intel64/myMallocTrace.so \
-- $HOME/Desktop/malloc-free-test.out
```

运行完毕后，将在挂载程序所在目录下生成 `malloctrace.out`，打开即为 PinTools 输出的插桩所得信息。

## 附录

插图 Markdown 扩展源码

```
graph LR
subgraph 主流程
A(初始化)
B(指定输出文件)
C(插桩)
D(运行挂载程序)
end

A-->B
B-->C
C-->D

A1[PIN_InitSymbols]
A2[PIN_Init]
A-.-A1
A-.-A2

B1[TraceFile]
B-.-B1

C1[IMG_AddInstrumentFunction]
C2[PIN_AddFiniFunction]
C-.-C1
C-.-C2

D1[PIN_StartProgram]
D-.-D1
```
