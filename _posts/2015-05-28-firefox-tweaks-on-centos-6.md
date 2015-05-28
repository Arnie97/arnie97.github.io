---
layout: post
title: CentOS 6安装Firefox挂机及优化
---
我的VPS有512MB内存，而选择这个套餐只是因为每月500GB的流量，平时内存占用只有100MB左右。既然闲着也是浪费，不如拿来挂机，补贴买VPS的花销。

今天的主要内容有：

- EPEL的配置
- 为Firefox安装Flash插件
- Xfce和VNC Server的安装和配置
- 用Nice调整进程优先级

## 安装
我的系统是CentOS 6 x86 minimal，并且先前就已经使用了EPEL(Extra Packages for Enterprise Linux)。
如果你没有安装，请自行下载安装：

```bash
# yum install epel-release
```

没错，这样都可以了。
网上广泛流传着[用`rpm`安装的方法，比较麻烦，不建议使用。](https://www.centosblog.com/enable-epel-repo-on-centos-5-and-centos-6)。

此外，为了[安装Adobe Flash Player](http://www.if-not-true-then-false.com/2010/install-adobe-flash-player-10-on-fedora-centos-red-hat-rhel)，可以添加这个软件包：

```bash
# rpm -ivh http://linuxdownload.adobe.com/adobe-release/adobe-release-i386-1.0-1.noarch.rpm
# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-adobe-linux
# yum check-update
```
之后就是Xfce、VNC Server、Firefox、Flash一通安装…

```bash
# yum groupinstall Xfce
# yum install tigervnc-server firefox flash-plugin
```

## 配置
首先，配置一下VNC Server：

```bash
# vi /etc/sysconfig/vncservers
# service vncserver start
```

用[VNC Viewer](https://www.realvnc.com/download/viewer)连接上去，看到X11的X形光标，貌似忘了启动Xfce了。

在X11的启动配置里，加上`/usr/bin/startxfce4`：

```bash
# vi ~/.vnc/xstartup
# service vncserver restart
```

这回看到小老鼠的Panel了，点开Firefox即可使用。
在设置里关掉历史记录，把缓存限制的小一些，装上各路插件，开始挂机。

研究Xfce和Flash怎么安装的，到这里就可以点叉了。
但是，我任务还没有完成。

## 优化
定时重启/清理日志这种我自认为很简单的问题就不讲了，只说我遇到的问题。

首先，Firefox的CPU占用太高了，导致其他服务非常不稳定。
尤其是OpenVPN、Tunnel Broker和SSHD，如果这些至关重要的服务中有一个不正常，基本只能靠Reboot了。

解决这个问题，无非就是限制指定进程的CPU占用吗。
放狗一搜，有人说`ulimit`，有人说`cpulimit`，有人说`nice`，有人说`cgroups`。
`ulimit`是Bash的功能，我的Zsh上居然不能用，直接被我忽略了。说好的与Bash兼容呢！

[这篇文章](http://blog.scoutapp.com/articles/2014/11/04/restricting-process-cpu-usage-using-nice-cpulimit-and-cgroups)对比介绍了后三种工具，果断就看它了。

`cgroups`功能强大，有点大材小用；而`cpulimit`不是系统自带工具，需要自行安装。
于是，最后我选用了`nice`。
说到`nice`，我们隔壁宿舍常常有人打着LOL，喊着nice，据说是来源于游戏解说的口头禅…

`nice`的优先级从-20到+19，我就把Firefox设为最低优先级好了。

```bash
# nice -n 19 /usr/lib/firefox/firefox
```

用`renice`可以调整已经启动进程的优先级，其中`-n`可以省略：

```bash
# ps aux | grep firefox | awk '{print $2}' | xargs renice +19
```

另一方面，Firefox会时不时的崩溃。
有时，登录挂机网站，发现分数没有增长，上VNC一看才发现Firefox已经关了。

其实我挂的网站之一，[eBesucher](http://www.ebesucher.com/?ref=trdiag)，提供了一个跨平台的[eBesucher Restarter](http://www.ebesucher.com/restarter.html)，可以监控Firefox的运行。
但是，这个软件是用Java写的！

据说，Java是我最讨厌的语言，没有之一；并且，我的机器上没装JRE。
再说了，这个软件既没开源，又没啥自定义选项，还不支持命令行界面，怎能入我的法眼啊。

于是，果断抄起Vim，自己写了一个：

```bash
#!/bin/bash

while true; do
    if [[ `ps aux | grep firefox | wc -l` -eq 0 ]]; then
        nice -n 19 /usr/lib/firefox/firefox
    fi
    sleep 15
done
```

蛋疼的是，`grep`君有时会把自己也数进去…
于是，又去[Stack Overflow](http://stackoverflow.com/questions/9375711/more-elegant-ps-aux-grep-v-grep)学了一招。
这个`ps -C`命令真心好用啊，再也不用`ps | grep`了：

```bash
#!/bin/bash

while true; do
    if [[ `ps -fC firefox | wc -l` -eq 1 ]]; then
        nice -n 19 /usr/lib/firefox/firefox
    fi
    sleep 15
done
```

好了，本文到此告一段落。

下一步打算写个爬虫，监视各网站分数的增长和兑换率的变化。
目前主要是卡在登录网站这一步了，包括验证码识别方面的问题。
