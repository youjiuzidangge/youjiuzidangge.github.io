---
layout: post
title:  Linux 系统服务
#时间配置
date:   2017-07-13 10:30:00 +0800
#大类配置
categories: Linux
#小类配置
tag: 知识体系
---

* content
{:toc}


我有时候会想自己会被问到哪些问题，结果我自己都感觉回到不到，这说明盲点很多啊。

+ 这本书主要讲的什么？
+ linux 它比较 window 到底有哪些压倒性的优势？不足又有哪些？
+ 你认为整个 linux 的重点在于哪里？理解哪些知识点就可以把整个 linux 系统串起来了？

`书要先读薄，再读厚才对。`

----------------------------------------------

思路分析
============================================
---------------------------------------------

## 什么是 daemon 与 service

 service：常驻在内存中的进程，且可以提供一些系统或网络功能，那就是服务。
 daemon：系统为了某些功能必须要提供一些服务，但 service 的提供总是需要程序的运作，达成这个 service 的程序我们就称为 daemon。
 
### daemon 分类

 启动与管理方式来分： stand alone（独立启动），super daemon 统一管理
	1. stand alone（独立启动）
	   + 可以自行启动而不需要通过其他程序的协助，一旦启动就一直存在于内存中，提供服务。因此当客户需要该服务能够快速响应。
	2. super daemon：一直特殊的 daemon 统一管理
	   + 即由一个 super daemon 来统一进行管理。而对于被管理的 daemon，只有客户端请求，服务才会被启动。
	   + 又分为两种：multi-threaded（多重线程）、single-threaded（单个线程）
	   
 工作形态来分：
	1. signal-control
		+ 透过讯号来管理，只要有需求进来，便立即启动
	2. interval-control
		+ 每隔一段时间就主动去执行某项工作
		
 daemon 命名规则：{xxx}d，例如例行性命令的建立的 at，与 cron 这两个服务，文件名会被取为 atd，crond。
 
### service 和 端口的关系

 由以上知，`系统所有的功能都是某些程序所提供的，而程序则是透过触发程序产生的`。
 
 对于网络服务也是如此。ip 是主机在互联网上面的‘门牌号码’，但网络服务有多项功能，当不同的任务请求同一个 ip 时 ip 只有一个，怎么区别这同一个网络服务的不同功能呢？
这个时候就需要用到端口号（port number）了。事实上，为了统一整个因特网的端口号对应服务的功能，好让所有的主机都能够使用相同的机制来提供服务与要求服务，
所以就有了‘通讯协议’。

### daemon 的启动脚本与启动方式

 daemon 的启动涉及很多选项与参数，因此通常 distribution 会给我们一个简单的 shell script 来进行启动的功能。该 script 可以进行环境的侦测、配置文件的分析、PID 档案的
放置，以及相关重要交换文件案的锁住（lock）动作，你只要执行该 script 就可以了。 那么这些 script 又是放在哪里呢？

	/etc/init.d/* : 启动脚本放置处
	  系统上几乎所有的服务启动脚本都放置在这里！事实上这是公认的目录，我们的 CentOS 实际上放置在 /etc/rc.d/init.d/ 啦！不过还是有设定连接到 /etc/init.d/ 的。
	/etc/sysconfig/*：各服务的初始化环境配置文件
      几乎所有的服务都会将初始化的一些选项设定写入到这个目录下，举例来说，登陆档的 syslog 这支 daemon 的初始化设定就写入在 /etc/sysconfig/syslog 这里呢！而
      网络的设定则写在 /etc/sysconfig/network 这个档案中。
	/etc/xinetd.conf，/etc/xinetd.d/*：super daemon 配置文件
	  super daemon 的主要配置文件（其实是默认值）为 /etc/xinetd.conf，不过我们上面就谈到了，super daemon 只是一个统一管理的机制，他所管理的其他 daemon 的设定则
	  写在 /etc/xinetd.d/*
	/etc/*：各服务各自的配置文件
	/var/lib/*：各服务产生的数据库
	  一些会产生数据的服务都会将他的数据写入到 /var/lib/ 目录中。举例， Mysql 的数据库默认就是写入 /var/lib/mysql/ 中。
	/var/run/*：各服务的程序之 PID 记录处
	
#### 如何启动 daemon
  Stand alone 的启动方式：
	1. 通过 /etc/init.d/* 中的脚本来启动
	2. 通过 service 这个程序来启动
  Super daemon 的启动方式：
	Super daemon 本身就是一支 stand alone 服务，但它管理的 daemon 不是，如果要启动，必须在配置文件中设定为启动该 daemon 才行。配置文件就是 /etc/xinetd.d/*。
	
## 解析 super daemon 配置文件

 前面谈到的 super daemon 是一支总管程序，这个 super daemon 是 xinetd 这一支程序所达成的。而由下图我们知道这个 xinetd 可以进行安全性或者是其他管理机制的控管，另外它也
能够控制联机的行为（多线程单线程），这些控制手段可以让我们的某些服务更为安全，资源管理更为合理。而由于 super daemon 可以作这样的管理，因此一些对客户端开放较多权限的服务
（例如 telnet），或者本身不具有管理机制或防火墙的服务，就可以透过 xinetd 来管理。
 
 <p><img src="{{ '/styles/images/2017-07-13_Linux-super-daemon_theory.png' | prepend: site.baseurl }}" /></p>

### 默认值配置文件： xinetd.conf

 <p><img src="{{ '/styles/images/2017-07-13_Linux-super-daemon-xinetd_config.png' | prepend: site.baseurl }}" /></p>
 
 该默认值配置文件指的是 super daemon 管理下的 service，如果其设定值没有指定上述的项目，则该 service 的项目默认值就是上面的。上述意义：
 
> 一个服务最多可以有 50 个同时联机，但每秒钟发起的新联机最多仅能有 50 条，若超过 50 条则该服务会暂时停 10 秒钟。同一个来源的用户最多仅能达成 10 条联机。而登入的成功与
> 失败所记录的信息并不相同。

 其余参数设定就略去了。
 
### 一个简单的 rsync 范例设定
 略。
 
### 服务的防火墙管理 xinetd，TCP Wrappers

 linux 默认提供的软件分析工具： /etc/hosts.deny，/etc/hosts.all，另外如果安装 tcp wrappers 套件时，可以加上一些额外的追踪功能。
 
#### /etc/hosts.allow，/etc/hosts.deny 管理
 /etc/hosts.{allow|deny} 能够管理某些程序的网络使用，任何以 xinetd 管理的服务都可通过 /etc/hosts.allow，/etc/hosts.deny 来设定防火墙。那么什么是防火墙呢？简单来说，
就是针对来源 IP 或局域网进行允许或拒绝的设定，以决定该联机是否能够成功达成连接的一种方式就是了。

 其实 /etc/hosts.{allow|deny} 也是 /usr/sbin/tcpd 的配置文件，而这个 /usr/sbin/tcpd 则是用来分析进入系统的 TCP 网路封包的一个软件。TCP 是一种面向连接的网络联机封包，
包括 www,email,ftp 等等都是使用 TCP 封包来达成联机的。

 TCP Wrappers 来控管的是：

	1. 来源 IP 或 整个网域的 IP 网段。
	2. port
	
 基本上只要一个服务受到 xinetd 管理，或该服务程序支持 TCP Wrappers 函式的功能时，该服务的防火墙方面的设定就能够以 /etc/hosts.{allow，deny}来处理了。反过来说，不支持
TCP Wrappers 则无法使用。

### 配置文件语法 
 
 <p><img src="{{ '/styles/images/2017-07-13_Linux-hosts-allow-grammer.png' | prepend: site.baseurl }}" /></p>
 
### TCP Wrappers 特殊功能 略
### 系统开启的服务与观察

	ps,top：来找寻已经启动的服务的程序与它的 PID
	netstat：检查 port
	
### 设定开机后立即启动服务的方法
 Linux 主机开机过程：
	1. 打开计算机电源，开始读取 BIOS 并进行主机的自我测试
	2. 通过 BIOS 取得第一个可开机装置，读取主要开机区（MBR）取得开机管理程序
	3. 透过开机管理程序的设定，取得 kernel 并加载内存且侦测系统硬件
	4. 核心主动呼叫 init 程序
	5. init 程序开始执行系统初始化（/etc/rc.d/rc.sysinit）
	6. 依据 init 的设定进行 daemon start （/etc/rc.d/rc[0-6].d/*）
	7. 加载本机设定
	
	chkconfig：管理系统服务默认开机启动与否
	ntsysv：类图形接口管理模式（Red Hat 系统特有）
	