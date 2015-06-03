---
layout: post
title: 在VPS中利用Tunnel Broker建立IPv6连接
---
我用的几家平价VPS和云主机都不支持原生IPv6。
想要使用IPv6，可以借助Tunnel Broker等隧道技术。

通常，主机供应商不会在OpenVZ中启用sit设备（因为需要重新编译内核等等）。
此时，可以通过`tb-tun`这个用户空间的程序来建立隧道。

## 启用`tun`设备

```bash
$ cat /dev/net/tun
cat: /dev/net/tun: File descriptor in bad state
```

如果你看到的结果和我一样，就说明条件已经满足。
其他情况我没有遇到过，你自己看着办…

## 下载并编译编译`tb-tun`

```bash
$ wget http://tb-tun.googlecode.com/files/tb-tun_r14.tar.gz
$ tar zxf tb-tun_r14.tar.gz
$ cd tb-tun_r14
$ gcc tb_userspace.c -l pthread -o tb_userspace
```

## 选择Tunnel Broker服务器

我用的是这家：[HE Free IPv6 Tunnel Broker](https://tunnelbroker.net)。
注册以后，选择一个离你的VPS比较近的服务器节点。
我的机房在加州，因此选择了Fremont节点。

注册完成后，记录以下信息备用：

```
Server IPv4 Address: 64.62.134.130
Routed /64:          2001:470:66:233::/64
```

## 建立隧道

你可以在`/64`块里随便给VPS分配一个IP。
细思恐极，全人类的IPv4地址好像也没有这么多啊。

注意，不能使用`::1`结尾的这个地址，这个地址预留给HE的服务器了。
在这里，我分配的是`2001:470:66:233::3/64`。

然后，你需要知道你VPS的公网IP。
呃…地球人都应该知道了吧…
如果你由于某些原因不知道的话，这个网站挺好用的：

```bash
$ curl ifconfig.co
104.194.77.51
```

好了，隧道的配置正式开始。

```bash
$ setsid ./tb_userspace tb0 64.62.134.130 104.194.77.51 sit
$ ifconfig tb0 up
$ ifconfig tb0 inet6 add 2001:470:66:233::3/64
$ ifconfig tb0 mtu 1480
$ route -A inet6 add ::/0 dev tb0
```

网卡的名字`tb0`可以随便起。
我之所以取这个名字，是因为我建立了连接不同服务器的若干隧道，用于测速…

用`ping6`测试一下，网络不通…
后来发现，默认的以太网卡上没有IPv6连接，却启用了IPv6协议…

如果你的VPS也是OpenVZ架构的话：

```bash
$ ip -6 del default dev venet0
$ ip -6 add default dev tunb
```

参考文献：
http://ichon.me/post/659.html
