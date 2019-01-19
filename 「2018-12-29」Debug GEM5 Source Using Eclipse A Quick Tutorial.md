# 如何使用 Eclipse 调试 GEM5 代码

[toc]

Found DPRINTF based debugging unable to meet your needs?

Found GDB based debugging unfriendly to human beings?

Want to debug gem5 source with the help of modern IDEs like Eclipse?

Failed in getting help from GEM5 community?

Come on, dude! Here is the up-to-date tutorial for you!

## Environment Configuration

Note that this is just the environment we are currently working under, and it's at your own risk to adopt the other environments.

* OS：Ubuntu 16.04.1 Desktop
* Eclipse： eclipse-cpp-2018-12-R-linux-gtk-x86_64
* Gem5 Commit ID：c428c220fd351626e2ee0005dda696940261793b
* OpenJDK: 1.8.0_191

## Steps to Work

1. Build gem5.debug
    ```bash
    git clone https://github.com/gem5/gem5
    cd gem5
    scons build/X86/gem5.debug -j$(cat /proc/cpuinfo | grep "processor" | wc -l)
    cd util/term
    make && sudo make install
    cd ../../..
    ```
1. Install JRE
    ```bash
    sudo apt install default-jre
    ```
1. Setup eclipse
    ```bash
    wget http://ftp.yz.yamagata-u.ac.jp/pub/eclipse/technology/epp/downloads/release/2018-12/R/eclipse-cpp-2018-12-R-linux-gtk-x86_64.tar.gz
    tar -xvzf eclipse-cpp-2018-12-R-linux-gtk-x86_64.tar.gz
    ```
1. Config eclipse
    1. Open eclipse
        ```bash
        cd eclipse
        ./eclipse
        ```
	1. Import gem5.debug as the C/C++ executable file

        Menu -> File -> Import -> C/C\++ folder -> C/C++ Executable 
        ![alt](https://img-blog.csdnimg.cn/20181229085920483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
        Select gem5.debug as the executable

        > eg. at gem5/build/X86/gem5.debug
        
        ![alt](https://img-blog.csdnimg.cn/20181229085955148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
	1. Create a lauch configuration

        Just Leave every thing **AS IT IS**.
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122909004745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)	
1. Debug configurations
	1. Set arguments for gem5.debug to bootup gem5 in the way you expected.
            		![在这里插入图片描述](https://img-blog.csdnimg.cn/20181229090107299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
	1. Set the variable M5_PATH to point out the location of your binaries if you are to use gem5 FS mode.
            ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181229090120932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
          > It's beyond of this tutorial's scope to describe where to put these binaries and disk images .
	1. Tell Eclipse where to lookup the source.
            ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181229090140147.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
            ```
            eg. at gem5/build/X86
            ```
1. Just be ready for THE ENDLESS NIGHTMARE gem5 will bring!
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181229090217414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21hb2tlbG9uZzk1,size_16,color_FFFFFF,t_70)
