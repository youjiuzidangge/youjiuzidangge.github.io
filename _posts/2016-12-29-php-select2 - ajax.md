---
layout: post
title:  Ajax无刷新怎么修改select2的选中状态 以及 ajax.deferred
#时间配置
date:   2016-12-29 23:59:00 +0800
#大类配置
categories: 代码
#小类配置
tag: php
---

* content
{:toc}


什么是 select2 就不累述了，网上一搜一大把。

	example = $(要改变选中状态的select标签).select2();
    example.val(data.supplier_id).trigger("change");
	
	
关于deferred，轮子造的太好了，不用再造了。 
推荐 [luckykun](http://www.cnblogs.com/jarson-7426/p/5511734.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)