---
layout: post
title: 命令行与Git入门概述
---
本文由五部分组成，用Git发布在Github Pages上。

- 第一部分简述了UNIX及类似的终端环境。
- 第二部分是有关命令行操作的基础知识。
- 第三部分针对Git的简单操作进行讲解。
- 第四部分是有关Github平台的补充知识。
- 第五部分给出了一些学习建议:smile:


## 10分钟基础知识 -  warming up
1.  在UNIX中，无论是程序，磁盘，网络，设备，一切皆文件。始终要牢记这一点，这样当我把目录叫做文件时，你就不会感到惊讶，因为目录也是一种文件！

    在所有文件中，文本文件具有着重要地位。对UNIX而言，无论是系统的配置还是应用程序的编制，实质上都是对文本文件的修改。因此UNIX上有一系列围绕文本文件这一主题的强大工具。

2.  用户通过`终端(Terminal)`与计算机交流。UNIX诞生于半个世纪以前，早期的终端是打印机，后来是用电缆连接到大型机的显示器和键盘。现在大型机和小型机被PC所取代，而终端演化为一个仿真硬件终端的软件，也就是`Terminal Emulator`。现在硬件终端已经成为古董了，然而`命令行(Command Line)`仍然焕发着活力，今日所称的Terminal通常就代指终端模拟器，例如`cmd`。

3.  终端只是一个输入输出设备，没有计算能力，命令的解释是由`Shell`完成的。Shell解释用户输入的命令，并通知`操作系统内核(Kernel)`执行操作。GNU/Linux操作系统的默认Shell是GNU Bash，内核是Linux。

4.  UNIX是一个符合IEEE 1003.1国际标准（通称为POSIX，就像IEEE 802.11可以叫做WiFi）的商业操作系统。GNU致力于为开发与UNIX兼容的应用程序，Bash（Bourne Again Shell）就是UNIX中Bourne Shell的开源、增强版。Linux是目前世界上应用最广泛的开源操作系统，也和POSIX、UNIX兼容。

5.  早期的微软Windows就是DOS平台上的一个App，直到Windows NT才成为一个现代意义上的操作系统，但仍保留了DOS的部分传统。在较新版本的Windows NT，比如Windows XP、Windows 7、Windows 8.1中，微软引入了部分与UNIX相近的概念，但Windows仍是一个封闭的操作系统，与POSIX标准不兼容。

6.  2016年，微软在Windows 10中引入了Linux子系统。启用这一子系统后，可以直接安装和运行Ubuntu Linux软件源中的绝大多数工具，进行Linux的学习与开发。

7.  在此前，为了在Windows环境上享受UNIX的便利，人们编写了`Cygwin`和`MSYS(MinGW)`。它们是Windows上两种主要的UNIX仿真环境。不同于仿真硬件的虚拟机，它们只是将程序对Linux内核的调用翻译为Windows内核可理解的形式。[`Git for Windows`](https://git-for-windows.github.io/)就是基于`MSYS`平台的，而`Cygwin`中也能通过包管理器安装Git。本文中的示例基于`MSYS2`编写，但也适用于其他各种POSIX环境。


## 命令行操作实战 - hello world
1.  请打开一个Bash Shell，并正确调教你的输入法。本文使用的符号全部为半角ASCII字符。
2.  光标前面的`$`称为`命令提示符(Prompt)`。

    `$`代表Bourne Shell或Bash，常见的提示符还有C Shell的`%`。
    如果在资料中看到这样的代码段：

    ```bash
    $ ls -al --show-control-chars --color=auto
    ```

    表示需要在Bash或其他Shell的提示符后，输入该指令并按`<Enter>`确认，而不用输入提示符`$`本身。

3.  请试着输入以下指令：

    ```bash
    $ clear         # 清空屏幕。"#"后面是注释，不必输入。
    $ cd /etc       # Change Directory（切换目录）
    $ pwd           # Print Working Directory（显示当前所在目录）
    $ ls -a         # List（列表显示当前目录下的文件）
    $ printf 'Hello, world!\n'  # 咦，奇怪的东西混进来了？
    ```

4.  根目录用`/`表示，上述`/etc`指的就是根目录下的etc文件夹。

    当前目录用`.`表示，上一层目录则用`..`表示。例如`./git-completation.bash`表示当前目录下的一个Bash脚本。

    为了方便访问Windows中的盘符，MSYS中可以用`/x`来访问X盘。试着运行：

    ```bash
    $ ls /c/User
    ```

    将会显示出`C:\User`中的内容。

5.  MSYS中的Bash已经开启了自动补全功能，用`<Tab>`键触发。

    如果你想进入`/share/vim/vim74`这个文件夹，不妨这样按键：

    ```bash
    $ cd /sh <Tab> v <Tab> v <Tab> 7 <Tab>
    ```

    此时，系统会自动补齐为

    ```bash
    $ cd /share/vim/vim74
    ```

    当有多个匹配项时，按下`<Tab>`键，系统会将所有匹配项显示出来，供你选择。在上例中，如果你按了

    ```bash
    $ cd /s <Tab>
    ```

    由于根目录下有多个名称以s开头的文件，系统会显示出所有文件。这时候你应该再输入`h`，此时只有一个匹配项`/share`了，按`<Tab>`即可补全。

6.  操控命令行程序的方式主要有两种：
    - 用命令行参数调用
    - 交互式运行

    两种方式可以结合使用，其中第二种方式与图形界面相近，因此在此只说明第一种。

    命令行参数以空格分隔。在上文`cd /etc`中，运行了`cd`这条命令，第一个参数是`/etc`这个路径。`cd`本身就是这条命令的第零个参数。
    在`ls -a`中，实质上打开了名为`ls`的程序，第一个参数是`-a`，表示显示隐藏文件（show All files）。在UNIX中，文件名以点开头的文件（如`.git`）即为隐藏文件，而非文件属性的一部分。

    UNIX中，命令行参数通常有两种形式，短参数和长参数。长参数通常以两道短线开头，短参数以一道短线开头。请执行以下命令：

    ```bash
    $ git --version
    $ ls --help
    ```

    第一条命令显示了Git的版本号，第二条命令显示了`ls`的所有命令行参数。大部分UNIX命令都支持这两种参数，忘记时可以查看。

    注意，短参数可以合为一条。`ls -alh`和`ls -l -a -h`是等价的，也和长参数形式`ls -l --all --human-readable`效果相同。

7.  现在到你喜欢的地方建立一个临时文件夹，用于练习Git。为了您和他人的安全，请勿使用中文文件名。

    ```bash
    $ cd /d
    $ mkdir something  # 新建目录（Make Directory）
    $ cd ./something
    ```

## 版本控制系统Git - the stupid content tracker
1.  做完上面的操作，你应该已经在你自己建立的实验用文件夹里了。请在文件管理器中打开该文件夹，以方便随时查看其中内容的变化。如果你愿意用上文介绍的`ls`命令来查看，那就更好了:wink:

2.  Git允许多个用户对同一个项目进行协作开发，所以首先我们需要报一下户口：

    ```bash
    $ git config user.name "Arnie97"                # 用户名
    $ git config user.email "me@arnie97.progr.am"   # 邮箱
    ```

    注意可不要照抄我的命令噢，不然你做的所有贡献都要算到我头上来了。

3.  建立一个版本库

    ```bash
    $ git init      # 在当前目录初始化一个版本库
    $ git status    # 显示当前状态
    ```

    可以看到目录中新增了一个隐藏子目录`.git`，称为版本库(repo)。相对的，`.git`所在的父目录称为工作区(working directory)，你可以在里面写文章，写代码，或者干别的工作…

4.  开始工作

    由于现在工作区里空无一物，我就建立两个空白文件充数好了。

    ```bash
    $ touch justice
    $ touch tiger-ass   # 突发奇想^_^
    $ git status        # 状态发生改变，工作区有新文件没有提交
    ```

    touch命令可以安全的建立一个新文件——所谓「安全」是指当同名的文件已存在时，touch只修改该文件的修改时间来把它「变新」，而不会清空这一文件。

    如果你愿意，完全可以将原有的文件拷贝过来，或者拿起记事本写一段散文，也算对工作区进行修改了。

    注意，虽然Git也能处理二进制文件（如图片，压缩包，编译过的程序），但这并不符合Git设计的初衷。Git是按行对文本文件进行对比的，而二进制文件只能整个儿储存，频繁修改的二进制文件会使版本库的体积快速增加。

5.  将文件添加到暂存区

    不同于其他版本控制系统，在Git中，所有提交的文件必须先添加到暂存区(index / staged / cached)。
    于是，你可以自由选择提交的内容，避免了同时进行多项修改时，由于部分模块没有完成而不愿意提交的情况:blush:

    通常来讲，不要把不相关的多项修改一起提交，这样不利于寻找和修复Bug。

    ```bash
    $ git add justice
    $ git status
    ```

    从状态信息中可以看到，当前对`justice`的修改已经放到暂存区准备提交了，而`tiger-ass`不会被提交。

6.  开始提交

    提交本身很简单：

    ```bash
    $ git commit
    ```

    不过提交以前有一个小问题：MSYS中默认的文本编辑器是Vim。

    作为有几十年光辉历史的神级编辑器，Vim今天仍然是很多人的最爱，本文就是在Vim中完成的。
    不过，这个编辑器对新手可是不太友好…

    如果你不介意先学一下Vim（或者你已经开始提交了，不知道怎么退出Vim:frowning::sweat:），Vim自带了高端大气的教程：

    ```bash
    $ vimtutor
    ```

    如果你暂时不想学的话，你可以放弃交互界面，改为在命令行参数里指定提交信息：

    ```bash
    $ git commit -m "Added justice to the repo."
    ```

    或者换一个编辑器：

    ```bash
    $ git config --global core.editor "notepad"
    ```

    这里只是一个示例，墙裂不建议使用Notepad，它没有语法高亮，而且也无法正确显示UNIX的换行，看起来一团糟！

7.  对Git本地命令的入门介绍到此结束，如果你还想继续深入，请参考网络教程，如[Github提供的交互式教程](https://try.github.io)。Git主要开发者和维护者濱野純的书估计大多数人都语言不通，就不提了。蒋鑫写的《Git权威指南》极为详尽的介绍了Git的使用和实现细节，值得一读。以下是我认为较为重要的Git本地操作命令：

    ```bash
    $ gitk
    $ git log
    $ git diff
    $ git add --update
    $ git commit --amend
    $ git status --short
    $ git reset
    $ git checkout
    $ git branch
    $ git tag
    $ git merge
    $ git rebase --interactive
    $ git cherry-pick
    # ......
    ```

累了，停笔睡觉了……

## 2016年9月11日更新
本文已弃坑，本文已弃坑，本文已弃坑。
但前三章内容仍然根据mSysGit和Windows 10的最新进展进行了更新。

推荐以下两篇中文教程：

- [快乐的Linux命令行](https://billie66.github.io/TLCL/)
- [廖雪峰的Git教程](https://lvwzhen.gitbooks.io/git-tutorial/content/)
