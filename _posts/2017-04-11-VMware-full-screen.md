---
layout: post
title:  安装VMware tools后仍然不能全屏问题
#时间配置
date:   2017-04-10 22:30:00 +0800
#大类配置
categories: 代码
#小类配置
tag: linux
---

* content
{:toc}

试了很久，各种重装 VMware tools 依然没用。其实很简单(我是centos7)

apt-get install open-vm-tools open-vm-tools-desktop open-vm-tools-dkms

或者

yum install open-vm-tools open-vm-tools-desktop open-vm-tools-dkms

