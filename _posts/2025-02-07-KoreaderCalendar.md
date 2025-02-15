---
layout: post
title: 将KOReader中的日历视图转换为ICS文件
date: 2025-02-07 00:28:09 +0800
---
上个月兴致勃勃地把自己的Kindle越狱了，但是现在的Kindle本身的系统已经较为完善，当下这个时间点越狱的收益已经不是十分大，许多以前抓耳挠腮的小功能都已经内置，能好好玩玩的就是安装上第三方的阅读器KOReader，阅读器本身的功能不用多说，即使是作为亮点的PDF重排功能由于我依旧是阅读EPUB文件本身较多所以也没有体验到。在KOReader中更加令人眼前一亮的功能是它的阅读统计，可以单独统计一本书的整个的阅读时长、阅读页数、预估剩余时长；也可以每次将你的每次阅读都记录下来，放置在日历的时间轴中一览无余。作为一个想把自己的每项活动都加入日历的人，看到这个日历时间轴自然是狂喜，但很遗憾的是这个日历的功能只在KOReader本身上显示，并不能和我自身的日历进行联动。但是现在有了AI的帮助，我想应该可以想个办法来处理。  

大致树立了一下要干的流程：
获取KOReader中储存阅读记录的文件——读取这个文件中的数据——使用程序进行处理成包含书名、阅读开始时间、阅读结束时间的txt文件——导入iPhone中使用Shortcut来循环处理加入我的日历。

- 获取KOReader中的阅读记录
按照KOReader的说明，即使是软件进行了升级，阅读记录也不会丢，那么这份记录必然是单独储存的一份文件，网上稍微搜寻了一下，在[Github](https://github.com/koreader/koreader/issues/6454)上找到了其他人类似的问题，知道了数据存储的位置是：koboreader/settings/statistics.sqlite3。将Kindle连上电脑，把这个文件复制出来即可。  
- 数据处理
作为一个小白，对这个文件我毫无概念，甚至不知道该如何看到里面的数据，于是抱着试一试的心态用VSCode打开这份文件，但是显示出来的是一团乱码，着实难办只能再去网络上搜寻sqlite文件如何分析，才知道在VSCode中下载了sqlite view插件能够顺畅阅读这份文件。效果如下：
![](https://cdnfile.sspai.com/2025/02/07/f1f3855eff8310de24c9e33d89871d12.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
很明显，即使是能显示出所有数据但是如何处理它们对我来说真的是如坠天书，本想自己细细处理以后再让AI做编程的工作，现在以后的路只能完全依赖AI了。  
要处理一份数据库的文件，那么现在爆火的Deepseek网页端似乎只支持图片与文字自然不行，兜兜转转还是回到ChatGPT让它来帮我一把，果然他能够直接对SQLite3文件进行分析。
![](https://cdnfile.sspai.com/2025/02/07/b17eacd4beab1b998bbce2a8e45956e6.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
先尝试让它能够摘取出来一段我需要的数据形式来让我审查，比较惊喜的是AI现在并不只局限于回答我的问题，而会反过来问我要不要进行进一步的分析。
![](https://cdnfile.sspai.com/2025/02/07/8d75e3ad7e72f1561efbe2fb4ef10490.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
依照我的要求，它生成了整个Python程序，依旧让人吃惊它现在的能力，居然可以直接在网页上运行程序然后在线debug，但让人遗憾的是它自己的程序点击运行的时候一直报错“运行 ModuleNotFoundError: No module named micropip”，让它自己排查了几次仍然是报这同一个错误但是仔细看了看代码（代码看不懂，英文我还是能看懂的），程序里面压根没有要求调用名为 micropip 的 module。因此我认为这个报错的信息应该是ChatGPT自身的问题，直接把代码复制下来跑跑看，很幸运居然一个错误都没有报，直接获得了我想要的文件。
![](https://cdnfile.sspai.com/2025/02/07/feaf8ca38e3f1c91685e4ffa801a61ab.jpg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
![](https://cdnfile.sspai.com/2025/02/07/030dedbad601cb35498a4d98b8801fad.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
就这样，在不到三十分钟内的我，也能够直接生成一段代码来进行处理了，但是仔细看看这份txt文件依旧是有问题的，我不确定是不是KOReader的记录本身就是这样还是sqlite处理的问题，连续的一个文档，上一段的阅读结束时间就是下一段的阅读开始时间，明显这是一连串的阅读记录。
例如：我摘出输出文件的段落中
>仿生人会梦见电子羊吗？ | 2025-02-02 09:20:08 | 2025-02-02 09:20:15 
>仿生人会梦见电子羊吗？ | 2025-02-02 09:20:15 | 2025-02-02 09:20:39 
>仿生人会梦见电子羊吗？ | 2025-02-02 09:20:39 | 2025-02-02 09:21:22 
>仿生人会梦见电子羊吗？ | 2025-02-02 09:21:22 | 2025-02-02 09:21:50 
>仿生人会梦见电子羊吗？ | 2025-02-02 09:21:50 | 2025-02-02 09:22:30

照例询问ChatGPT，不需要我思考应该怎么改进，而是它给我提示一些优化的路径，接下来我的要求只需要根据它的优化路径进行相应的回复即可。
![](https://cdnfile.sspai.com/2025/02/07/eb9211ec687af501a5a26b2df135dd6c.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
重新修改后的代码我依旧直接复制到VSCode中运行，很快就获得了一份满意的txt记录文件。
- 使用Deepseek继续优化
这时候我本想大功告成，后面自己去琢磨琢磨Shortcut怎么处理就可以了，但是灵光一闪，我想到了[ICS文件](https://zh.wikipedia.org/wiki/ICalendar)，它是一种常见的日历文件，可以直接将批量日程导入各类常见的日历客户端中，我准备在继续“压榨”一下AI的能力，于是我下达了一个要求：
>让我们大胆点，有可能不输出这个txt文件，而是直接输出一个可以导入日历的ICS文件吗？另外间隔时间改为600秒

很遗憾，我没有购入ChatGPT Plus，文件处理这样的高级功能并不能无限使用，新的代码生成到一半的时候直接给我罢工了，但是山不转水转，既然我已经有了代码，这份SQLite文件本身能不能读已经不重要了，直接把代码复制给Deepseek，看看它能不能胜任接下来的工作。
![](https://cdnfile.sspai.com/2025/02/07/7a739a1510766def1610faeae7f9f13c.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
经历了几次服务器繁忙以后，我选择了深度思考选项居然不提示服务器繁忙了，在它思考了128s以后给了我一段代码，和优化内容的说明。
![](https://cdnfile.sspai.com/2025/02/07/d196119e91c382798b34878b6f6f2fc1.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
前两次的尝试已经让我对AI生成代码的能力十分信任了，按照提示在终端里安装了 icalendar 的依赖以后，我直接将代码复制了过去，依旧无比顺畅完成了这项任务，没有任何报错我就获得了一份ICS文件。
- 导入日历
这时候我按住了一下自己的兴奋，这份输出的文件我要校验只能用日历程序导入来进行校验，如果它数据不对，直接导入势必会污染掉我的整个日历，因为ICS的日历日程和日历订阅不一样，要删除我只能手动一个个日程进行删除，这将会无比麻烦，我选择在完全不用的日历上先试试。
![](https://cdnfile.sspai.com/2025/02/07/1e4769e08d42607c046aeae64677c8bc.JPEG?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1/format/webp)
很幸运，得到的结果完全符合我的期待，就这样在ChatGPT和Deepseek的双重帮助下，我一个电脑小白在一个小时内就完成了我的需求。

AI生成的代码我放在了我的[Github 仓库](https://github.com/mivansaka/koreadCalendar)中，有需要的可以自行取用，毕竟我只负责提供想法，而AI帮我完成了这一切。
