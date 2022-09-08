---
layout: post
title:  "DIY魔兽世界iphone铃声"
date:   2017-01-25 11:34:01 +0800
blurb:  "将魔兽世界里经典的声音变为手机铃声"
og_image:
categories: diy
tags:
- wow
---

# 背景

现在iphone已经烂大街了，铃声就那么几个，地铁上电话一响一帮人看手机。。。

因此我一直希望能设置一个不一样的铃声，比如魔兽世界里的各种音效。

![diy-wow-ringtong-logo]({{ site.baseurl }}/assets/img/201701/diy-wow-ringtong-logo.jpg)

此文介绍下面几个内容：

- 从哪里能找到魔兽世界音效素材
- 如何转换成iphone铃声文件
- 如何导入到iphone铃声中

--------------------

# 寻找魔兽世界音效

开始我打算从魔兽世界客户端里提取出音频文件，网上大家都用一个叫WinMPQ的windows软件来提取，

但我用的是mac啊，此路看来不通。

后来我想到了这个网站-[wowhead]，里面的数据库中包含所有wow里用到的音效

database->Other->Sounds

![wowhead-sounds]({{ site.baseurl }}/assets/img/201701/wowhead-sounds.png)

我们可以在上面搜索任意音效，比如我搜了一个拍卖行，出了两条结果：打开拍卖行和关闭拍卖行，魔兽铁粉们尤其是哪些生意人应该不会陌生吧

![wowhead-auciton]({{ site.baseurl }}/assets/img/201701/wowhead-auciton.png)

点击播放后能试听，而且会出现下载的箭头

![wowhead-sounds-test]({{ site.baseurl }}/assets/img/201701/wowhead-sounds-test.png)

下载后发现竟然是一个二进制文件，哦，不对，是“二进制”文本文件...

![wowhead-sounds-file]({{ site.baseurl }}/assets/img/201701/wowhead-sounds-file.png)

这个网站应该是做了特殊处理，只能得到音频文件的ascii版本，我还需做一系列的转换。


# 转换成音效文件

根据前几个字节“4f67 6753 0002”谷歌了一下，发现这个是ogg格式的音频文件。

我的转换策略是：txt->ogg->m4r

首先是把txt里的“16进制”文本按binary模式重新写到新文件里，变成可以播放的ogg音频。

我选择了mac上的一个二进制编辑器iHex，完全免费，直接将txt文件中的内容复制到编辑器中，

然后另存为以ogg后缀的文件AuctionWindowClose.ogg，保存后可以拖到chrome里试听一下。

最后转换ogg到m4a，我用一个叫[Online Audio Converter]的web应用，格式直接选择成iphone ringtone，Quality设置为最大。

![audio-converter]({{ site.baseurl }}/assets/img/201701/audio-converter.png)

最终能得到一个AuctionWindowClose.m4r文件，文件格式就转换完毕了。


# 导入到iphone铃声

iphone导入自定义铃声基本上靠iTunes，网上有很多教程。

打开iTunes，添加AuctionWindowClose.m4r音频文件：文件->添加到资料库

添加完成后，在铃声里就能看到了

![mac-itunes-ringtone]({{ site.baseurl }}/assets/img/201701/mac-itunes-ringtone.png)

最后连接上iphone，到铃声那页勾选所有铃声，然后点击同步

![iphone-sync-ringtone]({{ site.baseurl }}/assets/img/201701/iphone-sync-ringtone.png)

再看下iphone，铃声已经同步过去了。


# 附录
已经做好的铃声文件

- 打开拍卖行: [AuctionWindowOpen.m4r]
- 关闭拍卖行: [AuctionWindowClose.m4r]


[wowhead]: http://www.wowhead.com/
[Online Audio Converter]: http://online-audio-converter.com/
[AuctionWindowOpen.m4r]: {{ site.baseurl }}/assets/files/201701/AuctionWindowOpen.m4r
[AuctionWindowClose.m4r]: {{ site.baseurl }}/assets/files/201701/AuctionWindowClose.m4r

