---
layout: post
title:  "播客录制后背景噪音的简单处理办法"
date:   2022-05-15 20:20:09
---
## 博客录制中背景噪音的简单处理办法


*本方法仅限于个人在处理播客录制文件时碰到的背景音杂乱的问题的简易处理办法，并非专业音频文件处理人员，建议始终在录制时提供一个尽可能安静的环境才是最终应对方式*

听完我和顾老师还有小胡最近的《下沉消费指南-第二期》的听友们应该注意到了背景音中有许多的猫叫以及背景底噪，这是我的录音文件带来的问题，在这首先对听友们为带来了不好的收听体验表示道歉，其次对我们的合作伙伴们表示道歉。

遇上播客远程录制的时候往往会碰到一些不可抗力的因素导致背景底噪非常的大，例如我背景音中的猫叫或者是居住在马路旁边的车流声音。作为一个仅推出两期的播客的成员，我没有任何音频处理的经验，甚至一般剪辑软体也没有用过，但是碰见问题想想还是先使用技术的问题来解决它。

其实第一反应是想到猫叫和车流声音的频率与人类说话的频率并不一样，想到的是使用傅里叶变换以后再叠加上一个高通和低通的滤波器把人说话频率段的频率留下再逆变换回去，这样或许能保留最多的人声细节和保证不留下其他声音。于是先尝试使用了[ThinkDSP](https://thinkdsp-cn.readthedocs.io/zh_CN/latest/)提供的一系列音频的频域处理思路，先转换了整个音频文件的频谱。
![](https://s3.bmp.ovh/imgs/2022/05/15/a5cf6071984deee9.png)
再加上一个高通和低通滤波器只留下人说话的50-5000Hz的频率段，但非常惨，猫叫的频域和人说话的频域是有非常大的重叠的。。。滤完再逆变换回去的时候发现自己是处理个寂寞，只能再去寻求其他的办法。

感谢中文播客这几年的蓬勃发展，让人在寻找播客剪辑软件上简单易行，大致看了一些介绍的帖子，我选择了免费开源的[Audacity](https://audacityapp.net/)，全平台都有支援。带着试试看的心态来试图解决这个问题。
![](https://i.bmp.ovh/imgs/2022/05/15/53a68e134ba1f9ab.png)
界面看上去比较简陋，起初我也试图使用这个软件内置的频率处理的方式进行滤波操作，但很显然结果和之前差距不大，于是在观察音频时域谱的时候发现猫叫和底噪在波形上显示的都是较小的强度，而人声的强度都十分大，既然频率上滤不过，从时域上把强度高于某一分贝的音频直接消除掉也许能得到一个好的结果。
在翻看Audacity的效果插件的时候发现有一个插件名字叫做noise gate，单从名字上看觉得也许是我所需要实现的功能，果然效果十分美好，杂音滤完以后异常安静，猫叫、车流都听不到了。
![](https://i.bmp.ovh/imgs/2022/05/15/e853d62ae25961dd.png)

*简易操作方式如下*
1. （如果录音文件为m4a格式的文件，建议先使用格式转换为wav格式文件再进行处理。
2. 选择导入需要消噪的音频后，Ctrl+A全选需要处理的音频。
![](https://i.bmp.ovh/imgs/2022/05/15/4142dbc2f0c91194.png)
3. 在*效果*中找到Noise Gate这一插件，根据自己的需求，调节Gate threshold与Level reduction这一栏的db值，Gate threshold 值均为负值，调节它越靠近正数意味着消除的强度越大；Level reduction 为减少量，我将其值设置为都大于Gate threshold 的值，即使Gate threshold强度以下的音频波形均将为0的一根直线，至此消噪完毕。
![](https://i.bmp.ovh/imgs/2022/05/15/1e9ba23fef147f53.png)
4. 点击任务栏的文件-导出，选择自己需要的格式与编码即可导出已消噪的音频。

观察一下处理前后的音频波形即可看到经过Audacity 的noise gate插件处理过的音频文件，前段杂乱无章的小型起伏波形已经完全平平整整了，而人本身说话的声音并未被影响太多，在一些快速处理噪音强度远小于人声强度的音频文件时，使用这种方式消除噪声也许是最简单明快的方式。
![](https://i.bmp.ovh/imgs/2022/05/15/d6f5344cabb5ec1f.png)
![](https://upload.cc/i1/2022/05/15/tUWwuG.png)

