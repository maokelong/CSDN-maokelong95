# Visual Studio 2017：Linux C++ 开发

[TOC]

## 1. 前言

Macrc Goodner，Visual Studio C++ Team 的 PM，在 2016 年 11 月发布在 Youtube 的视频 [Visual Studio 2017  Linux development with C++](https://www.youtube.com/watch?v=XIiFuBczd6A) 中为我们介绍了 Visual Studio 2017 的 Linux 编程特性。该视频主打十分钟上手，并在片尾介绍了该特性支持的几种工作负载：

- Linux Servers and desktops
- Azure hosted or local VMs
- Docker containers
- Windows Linux Subsystem
- Linux devices
 
其中对于我来说「 Windows 的 Linux 子系统（windows subsystem for linux，WSL）」最具有吸引力。根据我对该视频的简单理解，Visual Studio 2017 + WSL 的组合调试程序会更容易一些，+ Linux Server 需要附加进程后才能捕捉断点，但 +WSL 无需附加进程，就像本地调试一样。

经简单的索引，发现没有人介绍过这块内容，因此我打算在这里分享一下我使用 Visual Studio 2017 + WSL 的经历。

## 2. 安装配置 WSL

### 2.1. 安装 WSL

WSL 是出现很久的 Windows 10 特性了，我在 16 年 7 月份就曾使用过（不过还是 Cygwin 更好用）。网上已经有很多介绍如何安装 WSL 的教程了，这里就不赘述了。

1. Win 键，输入 bash，在筛选器中点选；
2. Win+R，在「运行」窗口中输入 bash，回车；

> 万一你之前安装过 WSL，但是和我一样「不幸地」忘掉了账号密码，这里有一份方案：
> 
> 1. 进入 命令提示符（管理员）：`Win + X`，接着点选或按 `A`
> 2. 卸载 WSL：`lxrun /uninstall /full `
> 3. 安装 WSL：`lxrun /install`
 
### 2.2. 配置 WSL

#### 2.2.1. 配置 SSH

介绍下我们为什么要配置 SSH。在 Visual Studio 2017 中构建跨平台项目的时候，按下 F5 触发的是「远程GDB调试程序」按钮（与「本地Windows调试器」相对），然后弹出一个弹窗让我们填写远程客户端的 SSH 信息。

这里给出一份翻译自 StackOverflow：[How can I SSH into “Bash on Ubuntu on Windows 10”?](https://superuser.com/questions/1111591/how-can-i-ssh-into-bash-on-ubuntu-on-windows-10) 采纳回答的方案：

> 我成功地通过 127.0.0.1:22 连接上了 WSL，希望这个能帮到你：
> 
> 1. `sudo apt-get remove openssh-server`
> 2. `sudo apt-get install openssh-server`
> 3. `sudo vim /etc/ssh/sshd_config` ，然后禁止 root 登录： `PermitRootLogin no`
> 4. 接着在这行下面加一行：
>      `AllowUsers yourusername`
> 	（如果你想使用密码登录的话，请保证`PasswordAuthentication `被设置成`yes`）
> 5. 通过添加/修改 `UsePrivilegeSeparation no`  关闭 Privilege Separation
> 6. `sudo service ssh --full-restart`
> 7. 哦啦！

#### 2.2.2. 配置开发套件

我不清楚我到底需要安装哪些东西，但我在安装完 gcc / g++ / gdb /gdbserver 之后再摁 F5 就能 run 了。WSL 实际就是个 Ubuntu 系统，我的显示内核版本 3.4.0+，使用`sudo apt-get install`获取并安装就行。好吧，忍不住继续写了下去：

```
sudo apt-get install gcc g++ gdb gdbserver
``` 

## 2.3. 使用 Visual Studio 2017

### 2.3.1. 新建跨平台项目

模板 - 其他语言 - VIsual C++ - 跨平台 - Linux，然后从四种模板：空项目、控制台应用程序，闪烁，生成文件项目中选一个吧。我鼠标指针停留在闪烁上只是因为我想知道这玩意儿到底是个啥，作为 demo 我们最好选择控制台应用程序。

好吧，还是简单介绍下各种模板的特征吧：

- 空项目：
- 控制台应用程序：
	提供一个主要语句为 printf 的 main 文件，控制台版的 helloworld；
- 闪烁：
	提供闪烁 LED 的 DEMO，ARM 开发版的 helloworld；
- 生成文件项目：
	它的英文名叫做「Makefile Project」；
	
![新建跨平台项目](http://img.blog.csdn.net/20170321204404495?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
### 2.3.2. 编码

这是我写的一份用来测试 demand paging 的代码，通过 mmap 系统调用向操作系统申请 1G 的空间并迭代写入。mmap 是 Linux 提供的系统调用。

```C
#include <stdio.h>
#include <sys/mman.h>
#include <unistd.h>

// 数组声明
char *HUGE_ARRAY;
size_t HUGE_ARRAY_SIZE = 1024 * 1024 * 1024;

int main() {
  // 初始化数组（使用映射段而非数据段）
  HUGE_ARRAY = (char *)mmap((void *)0x600000000000, HUGE_ARRAY_SIZE,
                            PROT_READ | PROT_WRITE,
                            MAP_PRIVATE | MAP_ANON | MAP_FIXED, -1, 0);
  // 打印数组基址
  printf("%p\n", HUGE_ARRAY);

  // 测试 demand paging
  size_t i = 0;
  for (; i < HUGE_ARRAY_SIZE; i += 1) {
    // 数组迭代写入
    HUGE_ARRAY[i] = 1;

    // 每当写入地址与128MB对齐的时候就等待确认一次
    if ((size_t)&HUGE_ARRAY[i] % (128 * 1024 * 1024) == 0){
      printf("Just input something:")
      getchar();
    }
  }

  return 0;
}
```

### 2.3.3. 录入 SSH

如果你是第一次摁 F5，VS 会弹出一个窗口帮助你填写 SSH，否则你就没有这个待遇了。你至少有两种方法进行补救：

1. 工具 - 选项 - Cross Platfrom - Connection Manager
2. 在快速启动窗口（Ctrl + Q），输入 Connect，然后在自动补全窗口点选

![录入 SSH](http://img.blog.csdn.net/20170321210726943?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
### 2.3.4. 调试程序

在添加断点/运行程序之后，你的程序理应驻留在断点位置，并且你可以通过 **测试-Linux Console** 打开终端，同程序进行交互。终端见右下角黄色标题框部分。
![调试程序](http://img.blog.csdn.net/20170321211000523?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2.3.5. 为代码补全添加对 Linux 库函数的支持

虽说上述代码可以直接运行，然而宇宙第一 IDE 竟然不提供代码补全，反而告诉你头文件找不到/函数未定义，并且打了一大堆的波浪线~~~~~~~~~~~~~~~~~~~~！想逼死强迫症么？

我们可以通过将 Linux Headers 拷贝到 VS Linux 头文件目录中为 VS 添加对 Linux 库函数的支持！

通过如下步骤打开下图窗口：

解决方案 - 项目名 - （右键） - 属性 - 配置属性 - VC++目录 - 常规 - 包含目录 - （单击） - （下箭头） - 编辑
 
![目录](http://img.blog.csdn.net/20170321211732144?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFva2Vsb25nOTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

找到 Linux 头文件所在目录。我的目录是这样的：
```
D:\Software\Studio\Visual Studio Community 2017\Common7\IDE\VC\Linux\include\usr\include\
```
要知道 WSL 是可以直接访问 Windows 目录的。WSL 根目录下的`mnt/`下保存着各逻辑分区的卷标，你会看到`c`、`d`、`e`  etc. 这样的目录名。我们可以直接把 WSL 中的 Linux 头文件拷贝到 VS 的 linux 头文件目录中，比如：

```
cp usr/include/ /mnt/d/Software/Studio/Visual\ Studio\ Community\ 2017/Common7/IDE/VC/Linux/include/usr/include/ -R
```
（当然，我是拷贝到 D 盘再手工拷过去的）

然后我们的 VS 就没了那一堆下划线了，宇宙第一的代码补全也回来了 (^ _^)v。
 
---

> 吐槽：CSDN 的文本编辑器真是 巨！难！！用！！！