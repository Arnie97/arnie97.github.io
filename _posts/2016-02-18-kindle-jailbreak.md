---
layout: post
title: Kindle越狱笔记
---
Kindle 5.6.5的软件越狱方法已于月初公开，但是恰逢过年，昨天才注意到。

这是我第一次对Kindle进行Hack。
虽然由「[Kindle伴侣](http://kindlefere.com/)」提供的[教程](http://kindlefere.com/post/307.html)简单易懂，但是黑盒操作总归是不放心的。
就像我从来不用「引导自动修复工具」一样，鬼知道它乱动了些什么！

虽然目前我还造不出来轮子，但是拆开看看也不错。
于是，我决定打开教程中提供的几个文件，看看里面都装了啥。

## 0x00 Public Developer Key
为了越狱，首先要下载一个[`jb.zip`](http://www.mobileread.com/forums/showthread.php?p=3253662#post3253662)，把其中的文件`jb`复制到Kindle里。

让我们打开`jb`看看：

```bash
#!/bin/sh

# 此处省略两百字…

install_key()
{
    # Install our public key
    cat > "/etc/uks/pubdevkey01.pem" << EOF
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDJn1jWU+xxVv/eRKfCPR9e47lP
WN2rH33z9QbfnqmCxBRLP6mMjGy6APyycQXg3nPi5fcb75alZo+Oh012HpMe9Lnp
eEgloIdm1E4LOsyrz4kttQtGRlzCErmBGt6+cAVEV86y2phOJ3mLk0Ek9UQXbIUf
rvyJnS2MKLG2cczjlQIDAQAB
-----END PUBLIC KEY-----
EOF
    [ $? != 0 ] && return 1
    return 0
}

# 此处省略五百字…
```

原来就是一个Shell脚本啊，并且只做了一件事，导入开发者公钥。
既然用对应的私钥就能通过Kindle的固件校验，那么就能利用原生的更新机制来修改系统了，岂不妙哉！

那么，问题来了：我们怎么让Kindle执行这个脚本呢？

## 0x01 Launchpad
下一步我们要用Kindle浏览器打开`jb.zip`中的HTML文档，从而执行上面的脚本。
越狱分三步，Stage1中有一个域名长达495个字符的HTTP链接，而Stage2中有一段JavaScript脚本。
显然这与Kindle系统中某处漏洞相关，是本次越狱的核心。
但是我并没有查到这个漏洞相关的资料，只能暂且跳过这段了。
反正都是为执行上面的`jb`做铺垫，不会带来不可预知的后果。

第三步是在搜索框中执行`;fc-cache`。
搜索框能调用很多Kindle的隐藏功能，不知道怎么用的话可以看[这里](http://kindlefere.com/post/155.html)。
根据[MobileRead Wiki](http://wiki.mobileread.com/wiki/Kindle_Touch_Hacking#Search_Bar_Shortcuts)，这条命令的字面意思是更新fontconfig缓存，但是同时也会重启框架。
完成后，一行「Jailbreak succeeded」映入眼帘，现在可以刷入非官方的更新包啦。

## 0x02 Kindlet JailBreak
回到教程，我们需要将[`JailBreak-1.14.N-FW-5.x-hotfix.zip`](http://www.mobileread.com/forums/showpost.php?p=3004892&postcount=1597)中的bin文件放入Kindle，然后手动更新。
显然这是一个二进制文件，我们可以用[KindleTool](http://www.mobileread.com/forums/showthread.php?t=187880)来对付它：

```bash
$ mkdir jb_bridge
$ kindletool extract Update_jailbreak_bridge_1.14.N_install.bin jb_bridge
$ cd jb_bridge
$ ls -A
```

一共解压出来16个文件，其中只有8个是真正起作用的，另外一半用于对应文件的完整性校验。
这8个文件中，又有一半都是Shell脚本，包括`install-bridge.sh`，`libotautils5`，`bridge`和`bridge.conf`。
另外四个文件分别是：记录升级包中各文件信息的纯文本文件`update-filelist.dat`，ELF可执行文件`gandalf`（呃，「指环王」爱好者？），Java包`json_simple-1.1.jar`及密钥库`developer.keystore`。

我们先打开`install-bridge.sh`看看。
首先，加载了`libotautils5`：

```bash
# Pull libOTAUtils for logging & progress handling
[ -f ./libotautils5 ] && source ./libotautils5
```

之后，除了把更新包里的文件备份到`/var/local/mkk`和`/mnt/us/mkk`以外，主要就是将`bridge`安装到了`/var/local/system/fixup`，并且通过[upstart](http://upstart.ubuntu.com)随框架启动：

```bash
# Install the bridge
cp -f bridge "/var/local/system/fixup"
chmod a+x "/var/local/system/fixup"

logmsg "I" "install" "" "Installing bridge job"
cp -f bridge.conf "/etc/upstart/bridge.conf"
chmod 0664 "/etc/upstart/bridge.conf"
```

关于`json_simple-1.1.jar`的作用，日志中作出了说明：

```bash
logmsg "I" "install" "" "Storing kindlet jailbreak"
cp -f json_simple-1.1.jar "${MKK_PERSISTENT_STORAGE}/json_simple-1.1.jar"
```

而`gandalf`的功能也很明显了：

```bash
logmsg "I" "install" "" "Setting up gandalf"
chmod a+x "${MKK_PERSISTENT_STORAGE}/gandalf"
chmod +s "${MKK_PERSISTENT_STORAGE}/gandalf"
ln -sf "${MKK_PERSISTENT_STORAGE}/gandalf" "${MKK_PERSISTENT_STORAGE}/su"
```

至此，已经将3个二进制文件复制进系统，并且重启之后`bridge`会随框架启动。
所以，我们接下来看看`bridge`的内容。

`bridge`的大部分代码都是在确认上两节的结果，不再赘述。
最后，`developer.keystore`被复制到`/var/local/java/keystore/`里，`json_simple-1.1.jar`被安装到`/opt/amazon/ebook/lib/`。至此，越狱成功完成！

## 0x03 KUAL & MRPI
这并不是越狱过程的一步。
但是，正如一个发行版不会只有Kernel，而没有Shell和Package Manager一样，Kindle Unified Application Launcher通常会成为你安装的第一个Kindlet，而MobileRead Package Installer会成为KUAL的第一个插件。

不过，说是安装，其实KUAL就是本电子书，复制进Kindle就行了。
MRPI也是复制到KUAL的`extensions`文件夹里就OK。

## 0xFF 总结
- 导入公钥，是为了安装非官方的更新包；
- 安装更新，是为了修改`su`，`bridge.conf`，从而使`bridge`能随机启动；
- 启动「桥梁」，是为了导入`developer.keystore`和`json_simple-1.1.jar`；
- 导入Java包，是为了使它被框架加载，支持KUAL等Kindlet的运行；
- 有了KUAL，就能通过它的插件MRPI方便的安装其他应用了！

多米诺骨牌绕来绕去，环环相扣，终于达到了目的。
想必Lab126为了封锁Kindle也是煞费苦心啊。
