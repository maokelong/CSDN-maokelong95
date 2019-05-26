# 「VS Code」 Remote Development using SSH (with a jump server to go through)

> **日志**：
>
> 1. 「2019-05-19」提交了全文初版。

@[toc]
## 简介

2019 年 05 月 02 日，Visual Studio Code（以下简称 VSC）公布了 Remote Development 插件的预览版（[公告传送门](https://code.visualstudio.com/blogs/2019/05/02/remote-development)）。只要下载 VSC v1.34 insider（[日志传送门](https://code.visualstudio.com/updates/v1_34)），也即四月份发布的内测版，就可以使用这个插件了。

通过这个插件，我们可以直接在 VSC 中打开服务器上的文件夹。不论这个服务器是一个真实的远程机，还是一个虚拟机，甚至只是一个容器，只要这个它上面能运行 SSH server 就行。这意味着传统的远程开发方法可以丢进角落了。

传统而言，如果我们要进行远程开发，我们往往会同时运行如下软件。不觉得这个方案麻烦的人一定是组了多屏。

* XShell：打开服务器的终端；
* XFtp 或 lrzsz 或 Git：同服务器传输文件；
* VSC：使用最舒服的编辑器写 BUG。

而现在，我们只需运行一个 VSC 就行了。

VSC 通过 Remote Development 插件连接上远程服务器，然后打开服务器上的文件夹作为 workspace。无需将服务器上的各种头文件和源码拷贝到本地上，我们就能使用包括 IntelliSense (completions)、code navigation 及 debugging 在内的各种功能。最后，我们还能使用编辑器内置的终端操纵服务器，去做一些 build、release 这样的事情。

单纯的使用运行 Remote Development 插件并不是什么很复杂的事情。实际上这个插件已经设计得足够优雅了。麻烦的事情在于出于各种需求，我们可能会希望先连接到一个跳板机上，再连接到我们的服务器。而 windows 平台的 openssh 套件中并不包含原生的适用于 ProxyCommand 的工具，使得传统适用于 linux 平台的教程并不能 work。

本文即针对这种场景，提出一整套 plug-and-play 的解决方案。

## S1. 准备好你的本地机器

> **注意**： 教程所基于的**本地机器**系 `windows10 1809`。

1. 安装适用于 Windows 10 的 OpenSSH，这里给出两种方案：
   * （推荐）使用 PowerShell 安装 OpenSSH：
      * 按下 win 键，输入 powershell，右键选择「以**管理员**身份打开」，分别输入：
        * `Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0`
          
          这是为了检查 OpenSSH 客户端及服务端是否安装，当均为安装时应打印：
          ```
          Name  : OpenSSH.Client~~~~0.0.1.0
          State : NotPresent
          Name  : OpenSSH.Server~~~~0.0.1.0
          State : NotPresent
          ```
        * `Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0`

          这是为了安装 ssh 客户端，安装成功时应打印：
          ```
          Path          :
          Online        : True
          RestartNeeded : False
          ```

          > 通过网络获取，而微软的网络服务质量大家是有目共睹的，所以这里为自己祈祷吧！

          ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518223959776.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
   * （不推荐）从设置在 Windows 10 1809 UI 安装 OpenSSH；
      * 按下 win 键，点选设置，点选应用，点选管理可选功能，点选添加功能，点选 OpenSSH 客户端。
      
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518224011891.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
1. 安装 Visual Studio Code Insiders（[传送门](https://code.visualstudio.com/insiders/)）；
1. 在插件市场中检索并安装 `Remote SSH`；
  
   > **注意**：安装 `Remote Development` 会自动安装 Remote - Container, Remote - SSH 及 Remote - WSL 全家桶。这里我们只需要 Remote-SSH。

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518224323233.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
  
1. 生成 SSH Key

   1. 进入 PowerShell；
   1. 输入 `ssh-keygen -t rsa -b 4096`
  
      按理说，按照上述步骤安装 ssh client 时会顺便安装 `ssh-keygen`。
   1. 在「资源管理器」中的目录栏输入 `%USERPROFILE%\.ssh\`，进入保存公私钥的目录，其中：

      * id_rsa：私钥，本地机器持有
      * id_rsa.pub：公钥，后面需要改名后上传到服务器上去
    
   > **注意**： 也有教程（[传送门](https://www.jianshu.com/p/0f2fb935a9a1)）称无需该步，但实测发现这样进入后几乎每次操作都需要输入密码，操作起来十分繁琐。因此这里采用 VSC 要求的公私钥对称加密技术。

## S2. 准备好你的远程机器

> **注意**： 教程所基于的**远程机器**系 `Ubuntu 18.04`。

将「本地机器」上的公钥保存在「远程机器」的合适位置处，这里就各凭手段了，如果你：

* 无需跳板就能 ssh 到远程机器，那么你只需在 CMD 里执行下述命令就好了，注意修改远程机器的名称：

   ```shell
   SET REMOTEHOST=your-user-name-on-host@host-fqdn-or-ip-goes-here

   scp %USERPROFILE%\.ssh\id_rsa.pub %REMOTEHOST%:~/tmp.pub
   ssh %REMOTEHOST% "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat ~/tmp.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && rm -f ~/tmp.pub"
   ```
* 反之，你需要手动完成这样的工作：
   1. 进入 `~/.ssh` 目录，将「本地机器」上的 `id_rsa.pub` 拷贝进去并更名为 `authorized_keys`；
   2. 将 `~/.ssh` 的权限设置为 `700`，将 `~/.ssh/authorized_keys` 的权限设置为 `600`。
      > **注意:** 在传输文件的时候 XFtp 或 lrzsz 都是不错的选择，这里以 lrzsz 为例：在 XShell 中依次输入：
      > ```shell
      > sudo apt install lrzsz                        # 安装 lrzsz
      > rz                                            # 调用资管管理器，自行上传公钥
      > mkdir -p ~/.ssh                               # 创建目录 ~/.ssh
      > chmod 700 ~/.ssh                              # 修改目录权限
      > cat id_rsa.pub >> ~/.ssh/authorized_keys      # 将公钥拷贝到  ~/.ssh 并命名为 authorized_keys
      > chmod 600 ~/.ssh/authorized_keys              # 修改公钥副本的权限
      > rm -f id_rsa.pub                              # 删除刚才上传的公钥
      > ```

> **注意**：当有多台本地机器都要连接远程机器怎么办？只用将本地机器的公钥 append 到 authorized_keys 中就行。

## S3. 开始远程连接

对了，想起来 Ubuntu 不会自带 openssh-server，虽然能看到这篇博客的人肯定不至于机器里连 sshserver 都没有装。如果要安装的话需要输入 `sudo apt install openssh-server`。

下面就开始编辑远程连接的配置文件了。

1. 点击界面最左边的 Remote-SSH；
2. 点击左上方 CONNECTION 窗口中的蓝字 Configure，并选择包含 `.ssh\config` 的配置文件；
  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518224045391.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
4. 如下所示填入配置

    ```shell
    Host bd07
      # 这里填入要在远程主机上登录的用户名
      User maokelong
      # 这里填入远程主机 IP
      HostName 192.168.x.xxx
      # 这里填入远程主机 ssh 端口
      Port 22
      # 这里填你私钥的路径
      IdentityFile c:\Users\mkl\.ssh\id_rsa
      # 当需要跳板机的时候就要填这个了
      # 这里代理类型根据需求填 socks4、socks5 或 http
      # 这里 xxx.xxx.xxx.xxx:xxx 表示跳板机 IP:端口
      # 这里 %h %p 无需修改，将自动分别读取 HostName 和 Port 的配置并填入
      ProxyCommand C:\bin\nmap-7.70\ncat.exe --proxy-type http --proxy xxx.xxx.xxx.xxx:xxx %h %p
    ```
    > **注意：** Windows 10 的 openSSH 里没有自带的用于支持 ProxyCommand 的工具，所以我们需要额外下载 ncat.exe 或 connect.exe。由于笔者在尝试 connect.exe 的时候遇到了无从解决的错误，所以这里仅介绍了 ncat.exe 的使用方式。这里假设你下载了（[传送门](https://nmap.org/dist/nmap-7.70-win32.zip)）并解压到了 `c:\bin`。所以你可以通过 `C:\bin\nmap-7.70\ncat.exe` 找到 ncat.exe。

配置文件编辑完就终于可以连接咱们的远程机器啦！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519230450294.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

## S4. Enjoy Yourself

Happy coding!

注意！如果你通过跳板机连接的远程机器，那么在成功连接后 VSC 可能会启动一个名称包含「ncat.exe」的 **纯黑框**。这可能是一个 BUG，和 VSC 无法在内置终端中运行这个命令有关。我们该怎么把它消除呢？在设置里开启「remote.SSH.showLoginTerminal」设置就好了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518224106952.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)

> 许愿：愿我早日得到甜甜的爱情！
