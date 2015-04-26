---
layout: post
title: 进击的高墙
---
早在一个月前，GFW就已经开始由防御转为攻击的尝试了。
通过在百度的页面中注入访问Github的JavaScript脚本，
GFW让所有在墙外访问国内网站的用户都沦为了这次大规模DDoS的帮凶。

说来也巧，那几天我不知道在忙些什么，反正没有登录Github。
因此，这次事情我并没有亲历，只是在Solidot的新闻中有所耳闻。
没想到，今天我就邂逅了传说中那堵「有进攻性」的防火墙——

今天晚上，Arduino提示有更新，习惯性的点了Cancel，结果再也找不到更新的选项了。无奈，用Google「手气不错」来到了[Arduino的官网](http://www.arduino.cc/en/Main/Software)。

这时候，有意思的事情开始了：我一点击下载链接，页面不久就跳转至[WPKG的主页](http://wpkg.org)了。这个页面介绍了一个名不见经传的工具，其用途是Windows域环境下的软件部署。

我一开始还在奇怪，难道我误点了赞助商的软件？但是Arduino社区素来没有这种流氓习气啊，何况我点击的是Linux版本的链接，为什么要推荐一个用于Windows的工具。后来，我发现情况有点不对劲：即使我不去点击，也会跳转到这个网站！

在Google中搜索「wpkg redirect」，映入眼帘的是[Reddit上刚刚发起的一篇讨论](http://www.reddit.com/r/techsupport/comments/33wnu6/almost_all_websites_in_both_chrome_and_ie_keep
)，看来这不是个例。仔细一看，好家伙，楼下回复的网友纷纷表示自己是大陆居民，顿时明白了，又是方校长在捣鬼。

但是问题仍没有解决。我的网络链接通过IPv6直接接入美国的Hurricane Electronic，平日里访问优酷时，总会被「由于版权原因，该视频仅限中国大陆地区观看」拒之门外，反倒是Youtube的访问十分流畅。
那么问题来了：~~学挖掘机那家强？~~
我在美国访问美国的网站，方校长是怎么漂洋过海，找上门来的？

仔细一想，DHCP为我分配了银桥的DNS服务器，而银桥转发的是OneDNS的解析结果，这个比较小众的DNS能够过滤一部分广告和恶意网站，且国内访问的延迟较小，因此被我采用。难不成这次是它被伟大的墙污染了？

把DNS调整为`8.8.8.8`。刷新页面，下载正常开始了…

我的探索还没有结束。本着打破沙锅问到底的精神，我在浏览器中按下了`F12`。
不久之后，我看到了这样一段话：

```html
<script id="facebook-jssdk" src="//connect.facebook.net/en_US/all.js#xfbml=1&amp;appId=139445636250370"></script>
```

凭着感觉，我点击了那个外部js的链接，然后看到了神奇的内容：

```javascript
window.location.href = 'http://www.ptraveler.com/';
```

我刷新了这个页面，果不其然，这次我被重定向到了`ptraveler.com`…

至此，这个问题基本告一段落了。
有意思的是，这次跳转到的两个网站都没有什么让中共不快的内容，为什么要跳转至这些网站还是个迷。
难道只是随机抽取的？这个问题留到以后再探讨吧，困了，睡觉去了。
