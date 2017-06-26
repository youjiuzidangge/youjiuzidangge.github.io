---
layout: post
title:  Linux 的文件权限与目录配置 学习总结
#时间配置
date:   2017-06-26 23:13:00 +0800
#大类配置
categories: Linux
#小类配置
tag: 知识体系
---

* content
{:toc}


自己看了《鸟哥Linux私房菜》也有一阵子了，书好歹也看了一大半了，然后回想一下，能记忆起来的确实寥寥。如我看完书就忘记，那我还何必要看书？
如果我自己做的笔记从来不看，那我又何必要做？为了强化自己，所以要把之前每章的内容再梳理一遍。重在层次结构，重在知识点的联系，重在分析。

`书要先读薄，再读厚才对。`

----------------------------------------------

思路分析
============================================
---------------------------------------------

鸟哥的这一章节还是思路很清晰的。

　
#### 1.上来先用通俗的例子来系统的介绍用户与用户组的关系就像

&nbsp;&nbsp;&nbsp;&nbsp;*家人与家的关系，用户组与用户组的关系就像家与家之间的关系.*

　
#### 2.然后介绍 Linux 权限的具体内容：

&nbsp;&nbsp;&nbsp;&nbsp;文件的属性，如下图示例：

<p><img src="{{ '/styles/images/2017-06-26_Linux-file_attributes.png' | prepend: site.baseurl }}" /></p>

&nbsp;&nbsp;&nbsp;&nbsp;紧接着介绍如何改变文件属性与权限
&nbsp;&nbsp;&nbsp;&nbsp;涉及到的命令如下：

	chgrp [-R]  newgroup dirname/filename
	要改变的组名必须要在 /etc/group 中才行，即该组要存在
	chown [-R] newowner dirname/filename
	chmod [-R] xyz dirname/filename
	(r:4 w:2 x:1)

&nbsp;&nbsp;&nbsp;&nbsp;接着以上的意义：

&nbsp;&nbsp;&nbsp;&nbsp;先来讲讲具体的属性——

+ r: 可读取此文件的实际内容。
+ w: 可编辑、新增或者是修改该文件的内容（但不含删除该文件）。
+ x：该文件可以被系统执行的权限。

&nbsp;&nbsp;&nbsp;&nbsp;以此谈其重要性，说白了，就是为了 安全性，以及方便管理。

&nbsp;&nbsp;&nbsp;&nbsp;接下来进行了一定的扩展，系统的介绍 Linux 文件种类与扩展名。

	普通文件（regular file）、纯文本文件（ASCII）、二进制文件（binary）、
	数据格式文件（data）、目录（directory）、连接文件（link）、
	设备与设备文件（device）、套接字（sockets）,管道（FIFO，pipe）

&nbsp;&nbsp;&nbsp;&nbsp;至于扩展名，Linux 是不存在的。

　
#### 3.最后介绍了 Linux 目录配置结构，这个另外一篇博客已经介绍了。
