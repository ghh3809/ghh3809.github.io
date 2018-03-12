---
layout: post
title:  "同学，你掉了一份MATLAB入门教程(1)"
categories: MATLAB
tags:  MATLAB 简介 安装
author: 风之筝
---

* content
{:toc}

MATLAB是一款著名的商业数学软件，工科生基本上免不了经常和它打交道。相信很多工科生都能理解这种，每学一个软件就要多掌握一种语言的痛，所以很多人一提起编程就像要了命了一样。所以本教程就希望能够以最轻松的语言，带领大家**入门MATLAB**。我相信当你逐渐了解它、熟悉它之后，你会离不开它的。





另外，阅读本教程的你，有一些基础的编程知识（例如理解**判断、循环、函数、数组**等概念）是再好不过的啦。当然，如果你对编程一窍不通，这本教程对你来说也不会非常费力。

还有就是，笔者其实也只能说对MATLAB有所了解而已。如果这份入门教程对你来说太过容易，不妨尝试一下相关书籍，相信你会从中学到更多。如果发现本教程中的错误，欢迎在评论处指正或进行讨论！

## 简介

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_matlab.jpg "MATLAB")

MATLAB，其“全称”叫做Matrix Laboratory(矩阵实验室)。从名字上我们就可以了解到，其基本运算单位为**矩阵**(Matrix)。它是由美国mathworks公司发布的，主要面对科学计算、可视化以及交互式程序设计的高科技计算环境。嗯……听起来就很高端。事实上，MATLAB的功能确实非常强大，除了MATLAB语言的主要部分，其众多工具箱为用户提供了大量方便实用的处理工具。例如下图就是MATLAB小波工具箱的使用。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_matlab_tool.jpg "小波工具箱")

你可能会问：这么高端上档次的东西，学起来岂不是很困难？额，显然不会啊，因为…………………………高端的部分我们一般用不到啊(手动微笑)

这种废话我一般不太愿意多讲（其实就是懒）。更多相关简介，可参见百度百科[MATLAB](http://baike.baidu.com/link?url=o17VvI0QcgncZAkZPvdYNXvpbt_VFJW5rKmZDJl7kdgmAMAzuv9EeRkIynHC2XwxZFufr1GN4vp1-t6P3soNr_ "百度百科：MATLAB")。

## 为什么要学MATLAB

这位同学你提了一个好问题，其实学任何编程语言/软件之前都应该想清楚为什么要学。准确地说，MATLAB是一个介于编程语言和软件之间的一种存在，如果你愿意，利用交互式输入与鼠标点点点结合的方法往往也能满足你的需要。但当计算更加复杂时，就需要编写对应的脚本与函数实现更加高级的功能。

和其他编程语言相比，如果你有下列情况之一，不妨学习一下MATLAB：

1. 要对数据进行**数学处理与分析**（比如回归拟合各种插值）；
2. 要对数据进行不同维度的**可视化操作**（就是画折线散点各种图）；
3. 要对数据进行比Excel更加**自动化和多样化**的处理（哦，就是写代码）；
4. 要快速掌握一种语言，方便**科研工作**的进行与开展（你家科研不处理数据吗！）；
5. 经常冒出一些数学想法，希望通过编程**快速实现**（验证你的小型“哥德巴赫猜想”）；
6. 没有别的原因，就是想学会MATLAB。

而如果你存在下列情况，则MATLAB可能不是你所需要的：

1. 数据**无需大量数学操作**，求和、平均数、最值、方差功能已经足以满足需求（比如算个全班平均分……）；
2. 数据包括大量**文本内容信息**，或数据以提供信息为主而非相互关联的数字（类似产品信息清单什么鬼的）；
3. 希望通过数学运算做出一款**产品级的软件**（用MATLAB写个QQ？想想就刺激）；
4. **数据规模**非常庞大，或对数据**计算速度**有非常高的要求（要知道MATLAB是基于JAVA的）。

对于上述的1，相信Excel的功能足以满足你的要求。事实上，Excel功能之强大，甚至可以进行一些简易游戏的开发。对于2，可以尝试其他的不拘泥于数字处理的脚本语言，例如Python。第3种情况可能需要利用常规的编程语言进行产品开发，MATLAB还是主要以研究与分析为主。第4种情况，则可以寻找合适的大数据处理及高性能计算的方法。

当你明确了你的需求，那不要再犹豫，下面就开始安装MATLAB吧！

## 安装

MATLAB各个版本的安装方法大同小异，这里以MATLAB R2016a为例介绍安装方法。

### 1. 下载

首先[点此下载MATLAB R2016a的安装包及破解包](https://pan.baidu.com/s/1c20AVTI)，提取密码：abv1。下载完成后可以看到文件夹内有三个文件，其中**R2016a_win64**为安装包，**MATLAB 2016a Win64 Crack**为破解包。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_download.png)

### 2. 安装

1.成功下载后，即可进行安装。选择 R2016a_win64.part1.rar 进行解压，软件会自动完成两部分的解压。解压完成后将得到的 R2016a_win64.iso 文件加载至虚拟光驱，双击"setup.exe"即可进行安装。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_step1.jpg)

2.稍等片刻，待安装程序启动后，选择**使用文件安装密钥 不需要Internet连接**选项，并接受许可协议的条款。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_step2.jpg)

3.选择**我已有我的许可证的文件安装密钥**，输入密钥：**09806-07443-53955-64350-21751-41297**，随后一路下一步即可，直至安装完毕

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_step3.jpg)

### 3. 破解

1.打开MATLAB安装目录（默认在C:\Program Files\MATLAB\R2016a），打开文件**MATLAB安装目录\bin\win64\activate_matlab.exe**，选择**在不选择Internet情况下手动激活**，并点击下一步。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_step4.jpg)

2.解压之前下载的**MATLAB 2016a Win64 Crack.rar**，可以看到**license_standalone.lic**文件，此文件即为许可证文件。在安装程序中选择**输入许可证文件的完成路径（包括文件名）**，点击浏览，选择该许可证文件，再次点击下一步，即可激活完成。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_step5.jpg)

3.此时不要打开软件，将解压**MATLAB 2016a Win64 Crack.rar**得到的**MATLAB Production Server\R2016a\bin\win64**文件夹中的三个dll文件，覆盖到**MATLAB安装目录\bin\win64**（默认为C:\Program Files\MATLAB\R2016a\bin\win64）目录下，并选择“复制与替换”。至此，MATLAB R2016a安装完成。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_step6.jpg)

4.打开MATLAB，我们即可看到其主界面。终于打开了MATLAB是不是有点小欣喜呢！下一节，我们将对MATLAB的界面进行介绍。

![](https://raw.githubusercontent.com/ghh3809/ghh3809.github.io/master/_posts/_pic/20170814_main.png)



