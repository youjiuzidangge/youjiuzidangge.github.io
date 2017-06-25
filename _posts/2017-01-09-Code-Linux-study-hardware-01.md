---
layout: post
title:  Linux 学习总结——硬件篇（一）
#时间配置
date:   2017-01-09 23:33:00 +0800
#大类配置
categories: Linux
#小类配置
tag: 知识体系
---

* content
{:toc}


写在前面的话——

原计划上周六本篇博客即完成，奈何贪于安逸懒散，拖延至今才开始。该打！该打！

本篇博客，重在梳理思路。觉不可陷入纯碎复制粘贴原作者的原话来假装是自己的认知。

总览
-----------------------------------------
简单来说，计算机由三大部分组成，例如人——输入单位，输出单元，以及中间的处理单元。再说的更细一点，可以把处理单元分为，CPU内部的控制单元、
算术逻辑单元与内存。

输入与输出单元，即比如人的眼睛及其成像，很简单不赘述。

因此，我们从计算机的处理单元开始。

所谓处理单元，即中央处理器（CPU）即是人的一个大脑。在人的头脑中，又有各个负责的部分，好比各大中枢神经系统。相交配合，复杂而有秩序。
那么配合大脑的这些部分是什么呢？

	1.内存 = 头脑中的记录区块，能将信息的互动暂时记录起来，所有cpu的数据都是从内存中读取的。
	2.硬盘 = 头脑中的记忆区块：将重要的数据予以保存，需要的时候可以使用。但是使用一定是先保存在内存中，然后给cpu来使用。
	3.显卡 = 脑袋中的影像：将来自眼睛的刺激转成影响后在脑袋中呈现，所以显卡中所产生的数据也是来自于CPU控制的。
	（这里我们有必要厘清显卡（GPU）和 CPU 的关系。首先，他们是相对独立的两个部分。显卡更像是人体的一个器官，该器官接受大脑传过来的信息，经过处理后进行输出。）

然后就是连接各个单元的部分，即主板-神经系统，可将整个机体连接，并使之成为一个完整的个体。

CPU
-----------------------------------------
为什么我们说 CPU 像大脑呢，因为他负责处理、运算计算机内部的所有数据。而它为什么可以实现这样牛逼的功能，因为其内容被写入了微指令集。
分为`精简指令集`和`复杂指令集`。（这是两种不同的 CPU）

精简指令集（RISC）
=========================================
顾名思义，该微指令集较为精简，每个指令的执行时间都很短，完成的操作也很短，指令的执行性能比较好。但是如果要做复杂的事情，就要由多个指令来完成。常使用的各品牌手机、
PAD、导航系统、网络设备（交换机、路由器等），几乎都是使用ARM架构的CPU。（ARM 系列是基于微指令集CPU)

复杂指令集（CISC）
=========================================
同理，在 CISC 的微指令集中，每个小指令可以执行一些较低阶的硬件操作，指令数目多而且复杂，每条指令的长度并不相同。因为指令执行较为复杂，所以每条指令话费的时间较长，但每个
指令可以处理的工作较为丰富。常见的 CISC 微指令集 CPU 为 X86 架构的 CPU。

	RISC 与 CISC 只是大体的区分，但 RISC 之间，CISC 之间又有很大的不同，因此可以构成不同型号，不同性能的 CPU。

这里要提出一个问题，什么是 x86? x86_64?
	
	x86 计算机即个人计算机
	x86_64 即i686之后的所有 64 位计算机

上面说了 CPU 微观下的分类。那么在宏观角度上，我们又是怎么样来比较各个 CPU 之间的性能呢？那就是 CPU 的频率，即 CPU 每秒可以进行的工作次数。
说到这里，我们就要谈谈 CPU 的“外频”、“倍频”与“超频”了。

###“外频”、“倍频”与“超频”###
主板上，通过南北桥将各个部分连接起来。这就类似于工厂上的流水线，各个部分处理数据传递信息的速度一致才能充分发挥资源的作用。所以 CPU 与各部分的速度理论上一致才好。
但是由于 CPU 处要处理的信息量较大，为了满足其速度需求，开发商就在 CPU 内部再加上一个加速功能，这就是所谓的外频与倍频。

	外频：CPU 与外部组件进行数据传输的速度。
	倍频：CPU 用来加速工作效率的一个倍数。
	两者相乘就是 CPU 的频率速度。
	
以Intel Core 2 Duo E8400 CPU 为例，他的频率是 3.0GHz， 而外频是 333MHz，因此倍频就是 9 倍！(3.0G=333Mx9, 其中 1G=1000M)
而所谓的超频指的是，玩家通过主板的设定，人为的调高外频的手段。（倍频一般都在出厂时被锁定，无法更改）

内存
-------------------------------------------
内存：即内部存储器，可以是ram（随机存储器），也可以是rom（只读存储器），我们平常装机时说的内存专指内存条，即电脑的主存，他也是内存（内部存储器）

我们知道 CPU 所处理的数据都来自与主存储器（RAM,DRAM为其组件，叫做动态随机存取内存）。但注意，除了主存储器之外，计算中还有很多的内存存在。比如说，正常情况下，数据走
主存储器到 CPU，必须经过北桥。但是如果我们有一些常用的数据直接放到 CPU 内部，岂不是更有效的利用性能，而这部分就是 SRAM，静态随机存储内存。

我们再来谈谈 ROM（只读存储器）。比如我们熟悉的 BIOS 本质上是一套程序，这套程序被写死到一个内存芯片中，这个芯片在没有通电的情况下也能把数据记录下来，这个就是ROM。很的
韧体都是通过 ROM 来写入的，BIOS就是一种韧体。

显示适配器
------------------------------------------
即 VGA,它上面有一个内存的容量，用于记录图像影像的分辨率与颜色，其大小直接影响输出图像的效果。另外它还有自己的一套运算系统，即 GPU。