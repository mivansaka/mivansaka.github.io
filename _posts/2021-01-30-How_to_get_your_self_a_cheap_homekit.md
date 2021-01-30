---
layout: post
title:  "How to get yourself a cheap homekit device"
date:   2021-01-30 22:21:09
---
## How to get yourself a cheap homekit device 

**写在前面：**  

**1. 本篇的全部准备环境均为 Linux，如果是 Windows/Mac os 使用者，请自行网上寻找固件刷写教程（当然在这两种情况下的教程要多得多）**      

**2. 本教程需要一定的单片机基础知识，比如至少知道什么是引脚。。。**

**3.目前 ESP 8826 仅支援 2.4Ghz Wi-Fi，如果有强烈的 5Ghz Wi-Fi 日常使用需求可以不看本文**  

  

**材料准备**

- 一台有 USB A 接口的电脑  
- ESP-01/s 无线收发模块    ¥5.5
- ESP-8266 串口烧录下载器    ¥8.2 
- DHT11 温湿度传感器    ¥3.65

（如果有兴趣研究，可以购入 继电器模块，网上均有关于如何搭配继电器制作智能开关的教程） 
**引脚连接方式在文章最后** 

1. **模块介绍**

  ESP8266 是一款由上海乐鑫信息科技开发的可以作为微控制器使用的成本极低且具有完整TCP/IP协议栈的Wi-Fi IoT控制芯片。而 ESP-01/s 可以理解为该款芯片的一个子集，下图为该设备的引脚位置及对应的引脚功能。你所需要知道的就是 GND-接地、VCC-供电、GPIO-输入输出，这三个引脚位置及功能。

![](https://i.niupic.com/images/2021/01/30/9aAT.jpg)

  ESP-8266 串口烧录下载器是一个简单的用于烧录 ESP-01/s 的部件，背面同样有引脚标识。将 ESP-01/s 插入下载器，并将 USB 插入电脑即可和电脑进行通讯。

![](https://i.niupic.com/images/2021/01/30/9aAZ.jpg)

  DHT-11 温湿度模块本质上就是一个传感器，背面印有三个引脚的指示，+ out - 分别代表电压正极，信号输出，电压负极。

![](https://i.niupic.com/images/2021/01/30/9aAU.jpg)

2. **准备工作**   

+ 将 ESP-01/s 和烧录器固定好，和电脑连接。（如果你想也可以现在就把温控接好，没有影响）

+ 电脑上安装好 [esptool](https://github.com/espressif/esptool) 程序，Linux 下我只需要一条指令:  

     sudo pacman -S esptool   

     注意：该程序是用 python 写的，所以建议提前确认一下电脑的 python 环境。
     
+ 前往[Github](https://github.com/RavenSystem/esp-homekit-devices/wiki/Installation)主页下载 [fullhaaboot.bin](https://github.com/RavenSystem/haa/releases/latest/download/fullhaaboot.bin) 文件备用
  
+ 检查设备端口
     插入烧录器后自然会在设备列表中列出，在终端下我们使用如下指令查看，在我这插入的设备显示为 ttyUSB0,将它记下来  
     cd /dev   
     ls  
3. **正式刷入**
+ 清除 ESP-01/s 中的出厂缓存，在终端中输入：  
  esptool.py -p /dev/ttyUSB0 erase_flash  
  
+ 将刚刚准备好的 fullhaaboot.bin 文件，在终端中输入如下指令，通过串口刷入程序  
     esptool.py -p /dev/<你的串口端口> --baud 115200 write_flash -fs 1MB -fm dout -ff 40m 0x0 fullhaaboot.bin
     
+ 烧录完成过后等待一段时间，使用你的任意一台设备，搜寻 Wi-Fi 信号，可以找到一个以 HAA 开头的无密码信号，连接它，之后在浏览器中访问 192.168.4.1 进入设备管理界面。（由于我写本篇文章的时候设备已经在使用中，之前设置的时候忘记截图，因此使用网图进行说明）。  
   ![](https://i.niupic.com/images/2021/01/30/9aB4.jpg)
   第一行是 JSON 的配置，他将决定该芯片给 HomeKit 输出什么信息。笔者本身也不会写 JSON，但是开发者甚至提供了一个自动配置网站来帮助我们：[haa-configurator](https://glumb.github.io/haa-configurator/)。该网站提供了大量的已经写好的模板，但是据我观察这些模板供给的是全引脚的 ESP-8266 设备，但是不要担心，比较我们只有一个温度和湿度的传感器，可以自己来进行配置。
   
  ![](https://i.niupic.com/images/2021/01/30/9aB5.jpg)
   
+ 在 sample config 中选择 Empty Accessory，下面的 General Config 所有选项都不添加（似乎默认空模板也是加入 LED Pin 的，把它去掉）。下面的 Accessory Config 中添加 Thermometer & Humidity Sensor Accessory config，Sensor Type 选择 DHT11，Sensor Input GPIO 选择2号。之后剩下的其他所有选项都不用添加。当然另外提一嘴的是理论上所有传感器在出厂前都已经校准好，如果需要可以自己做校准后在下方 Offset 中加入偏移量。所有的参数选择好以后，下方的代码框会自动生成好 JSON 代码，例如我得到的就是：  
   {"c":{"o":0},"a":[{"t":24,"b":[],"n":1,"g":2}]}  
   
   将这段代码填入之前打开的网页的 JSON 框中。如果读者想要尝试更多可能，可以试着自己找些模块来连接上再使用这个工具来帮助生成配置。
+ 在设置页面，往下看能看到 Wi-Fi 名称，选中你所用的网络，输入密码点击 Save ，即可等待它自动重启。网络上普遍教程中写的是要等待八到十分钟，但按我几次刷入的情况来看大概一两分钟就好了，仅供参考。

1. **手机设置**
+ 手机连接好家中的同一个 Wi-Fi（再次注意不支援 5Ghz 的）， 打开家庭应用，点击+号，选择添加设备，在弹出的窗口中选择 I Don't Have a Code or Cannot Scan，等待手机自动扫描，会有一个 HAA 的设备显示，点选以后会让你扫描或输入数字序列号，别慌张，作者依旧在这为我们提供了。
  ![](https://i.niupic.com/images/2021/01/30/9aBa.png)
扫描它以后再等待设备的连接，之后跟着 HomeKit 的引导走就好。如果幸运的话，几十秒后你就能看到和笔者一样的温度和湿度信息了。把它拔下来插在家中任何一个 USB 接口上即可开始使用。

5. **效果展示**
不出意外你的 Home 应用中将会显示信息，下面两张图是我开空调前后十分钟的数据对比，还是蛮明显的，而且本身我把它放在家里比较偏僻的角落，如果放在正对的地方，变化会更加明显。
![](https://i.niupic.com/images/2021/01/30/9aBb.jpg)  
![](https://i.niupic.com/images/2021/01/30/9aBd.jpg) 
另外笔者已经测试过，如果家中有 HomePod 这类控制中枢，离开家在数据流量的情况下依旧能正常访问温度和湿度。  
最终硬件成品：  
![](https://i.niupic.com/images/2021/01/30/9aBg.jpg)

**TroubleShooting**
在我刷入的过程中碰到过一些问题，写在这里，如果有疑问可以参考一下
+ 关于 Linux 端口无法被识别/写入。作为 Arduino 的一份子，官网上给出了权限的解决方案：https://playground.arduino.cc/Linux/All/#Permission
+ 设备连接家庭应用添加成功后显示温度湿度均为0:引脚的 I/O 口接错输出位置了，请重新检查接线，或是 JSON 配置问题。
  
**引脚接线**
购入时一般会附赠一根杜邦线用于接线，如果没有请提前和客服确认好后再另外购入杜邦线。其他教程中往往是给接口，我为了偷个小懒，直接给出接线图片，根据线的颜色一对一连接上就好。
![](https://i.niupic.com/images/2021/01/30/9aBj.jpg)
![](https://i.niupic.com/images/2021/01/30/9aBi.jpg)

写些屁话：  
其实这个模块我已经知道两三年了，但是一直拖着迟迟没有下手，终于前两天决定来实现一下，折腾了一个下午发现确实可用。写出这篇教程来主要是为了大家能够在最低的成本下用上 HomeKit 设备。另外的继电器制作智能插座，由于手头没有烙铁和闲置的插线板所有短时间内也不会去做，我更多的期待是大家来自己动手试试看SrF3CgbC8r3jzYZ，看用这些廉价模块能不能创造更多的东西。 

Reference:  
https://github.com/RavenSystem/esp-homekit-devices  
https://medium.com/@eliu01011/%E5%8E%9F%E7%94%9F-homekit-with-nodemcu-da5c8f21913  
https://www.wandianshenme.com/play/esptooolpy-flash-espeasy-firmware/  
https://github.com/espressif/esptool  
https://www.taoidle.com/%E4%BD%BF%E7%94%A8esp-01s%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%94%9Fhomekit%E6%8E%A7%E5%88%B6.html
   



