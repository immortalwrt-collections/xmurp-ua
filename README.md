2024年3月7日更新：**听说我上b站了？**

---

大家好，我是这个模块的作者。好久不见。上次 commit 好像还是在两年前的上次。

很开心这个模块帮到了大家。我最开始折腾的时候，完全没想到居然可以有这么大的影响。再次非常非常感谢大家捧场。

但是，因为三次元生活繁忙，我决定宣布放弃这个模块。——你是不是以为我要这样说？并不是。

现在看自己几年前写的代码，感觉就是一坨粑粑。虽然它确实也能凑合用，但是，嗯，我觉得不行，辜负了大家的期待和热情。所以我计划重写，在 [reborn 分支](https://github.com/CHN-beta/xmurp-ua/tree/reborn)。如果你注意到 commit 的日期的话，其实我去年底就有这个想法了。至于现在的进度嘛，你们可以点进去看看。毕竟，三次元生活繁忙加上技术不如人家专门的程序猿，也没有人因此给我发工资，身边也有比这个模块更好玩的玩具，所以进度肯定不会快的。

今天说这番话呢，主要是看到[糖喵](https://nya.one/@Candinya)转发的他们公司的[招聘广告](https://rss3.notion.site/RSS3-d1eb85405c9f47a397039b8f145a40a3)，觉得非常诱人，有点感慨。当然我是不会去应聘的啦，毕竟专业不对口，就算是编程，也更喜欢做些底层的活（不然这个模块也不会诞生了）。

以下是原先的内容。当然很多我都加了删除线，删除线后面的东西是新写的。

---

~~有新的测试版本可用（在 dev 分支）。这个版本将不会正式发布，而是作为新的工具的一部分发布，敬请期待。~~ 当年因为考研，就鸽了。考研结束后又没了兴趣。哎，物是人非。

~~另外，希望 dnsdizhi.com 的站长看到这里后把复制粘贴我的那篇文章删掉，谢谢。如果有人知道如何可以联系到站长也可以告诉我。那篇文章的地址在[这里](https://www.dnsdizhi.com/tag/xmurp-ua)。那篇是完全复制粘贴的我的文章（的一个历史版本），但却是百度结果的第一篇文章（似乎也是唯一一篇）。我的文章在[这里](https://catalog.chn.moe/%E6%95%99%E7%A8%8B/OpenWrt/%E5%9C%A8%E5%8E%A6%E5%A4%A7%E5%AE%BF%E8%88%8D%E5%AE%89%E8%A3%85%E8%B7%AF%E7%94%B1%E5%99%A8)。~~ 不必了，当时我还年轻，没见过这些垃圾站，现在已经见怪不怪了。

点击[此处](https://catalog.chn.moe/%E8%87%AA%E6%88%91%E4%BB%8B%E7%BB%8D/%E6%8A%95%E5%96%82%E5%9C%B0%E5%9D%80/)捐赠，欢迎投喂～

---

喵喵喵，这里是一个修改 UA 的小模块。细致地讲，就是用在 OpenWrt 上修改发给外网 80 端口 GET 和 POST 请求的 UA 字段为 `XMURP/1.0` 再加很多个空格的内核模块，用来防止学校检测到使用代理（接路由器）。当然，你改一下 `Makefile`，用到其它 Linux 系统上也是可以的。如果你不了解 OpenWrt 的食用方法或者还是不明白这个插件是用来干什么的的话，可以看[这里](https://catalog.chn.moe/教程/OpenWrt/在厦大宿舍安装路由器)。如果你不知道怎么编译，可以把 SDK 发给我我给你编译。如果要自己编译，看[这里](compile.md)。~~如果你使用官方稳定版，我把编译好的丢到[这里](https://github.com/CHN-beta/rkp-build)了，自取。~~ 后来的版本都懒得编译了，你要是需要直接联系我就可，我帮忙编译。

到现在为止，在 `4.x` 和 `3.x` 的内核上好像用得都没问题。

~~27 及以前的版本会导致路由器随机卡死，注意更新到最新版本。~~ 27 之前卡死或者重启是因为当时没意识到临界区需要加锁，后来加了，但是还是卡死了，我暂时还不知道为啥。

这个模块因为设计上的缺陷，不能修改到所有的 UA，只能修改绝大多数的，对于厦门大学的情况，够用了。dev 分支已经解决了这个问题，~~完美~~理论上是完美的，但实际上还是会卡死，我暂时也不知道为啥。

如果有一些包不希望被改 UA，只要在防火墙规则里将 MARK 的第九位设置为 1 就可以了。例如：

```
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 80 -m mac --mac-source f8:94:c2:85:e8:14 -j MARK --set-xmark 0x100/0x100
```

在之前的版本中，使用的是 `0x1/0x1` 位，但是与 luci-app-shadowsocks 冲突，所以改到了 `0x100/0x100`。

另外，不要在 luci 中启用 flow offloading（流量分载，即 nat 加速），否则这个模块会失效。可以通过下面的命令（二选一，不需要两句都写）来对不需要这个模块的流量启用。

```
iptables -t filter -I FORWARD -p tcp ! --dport 80 ! --sport 80 -m conntrack --ctstate RELATED,ESTABLISHED -j FLOWOFFLOAD --hw
iptables -t filter -I FORWARD -p tcp ! --dport 80 ! --sport 80 -m conntrack --ctstate RELATED,ESTABLISHED -j FLOWOFFLOAD
```

两句的区别的话，大概是前者用硬件，后者用软件。具体的东西我也不熟悉。

---

有人问我为啥不能编译个二进制文件放出来，因为这是内核模块啊，和大多的 OpenWrt 模块是不一样的，要和内核版本（~~精确到 commit id~~ 只要patch没改动内核头文件就行，应该不需要这么严格）严格对应。至于为啥要写到内核里，是历史原因：最开始我想修改 ipid，这当然在内核里更方便；后来又想改 ua，就继续写在内核里了。最开始的功能还很简单，后来代码一点点变复杂了，~~我也后悔开始时写到内核里了~~我不后悔，甚至非常骄傲。