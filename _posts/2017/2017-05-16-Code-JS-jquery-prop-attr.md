---
layout: post
title:  jquery中attr和prop的区别
#时间配置
date:   2017-05-18 22:06:00 +0800
#大类配置
categories: JavaScript
#小类配置
tag: jquery
---

* content
{:toc}


今天设置更改 input 单选状态时，怎么都改不了，网上无意间搜索到 prop 方法，一试即灵。但本着节约劳动成本最省时间原则，
我就不自己一点点写啦。转载仔[〖芈老头〗的技术空间]http://www.cnblogs.com/Showshare/p/different-between-attr-and-prop.html) 

--------------------------------------------------

#jquery中attr和prop的区别

在高版本的jquery引入prop方法后，什么时候该用prop？什么时候用attr？它们两个之间有什么区别？这些问题就出现了。

关于它们两个的区别，网上的答案很多。这里谈谈我的心得，我的心得很简单：

+ 对于HTML元素本身就带有的固有属性，在处理时，使用prop方法。
+ 对于HTML元素我们自己自定义的DOM属性，在处理时，使用attr方法。
 
上面的描述也许有点模糊，举几个例子就知道了。 
~~~ html
	<a href="http://www.baidu.com" target="_self" class="btn">百度</a>
~~~		
这个例子里 a 元素的DOM属性有“href、target和class"，这些属性就是 a 元素本身就带有的属性，也是W3C标准里就包含有这几个属性，
或者说在IDE里能够智能提示出的属性，这些就叫做固有属性。处理这些属性时，建议使用prop方法。
~~~ html
	<a href="#" id="link1" action="delete">删除</a>
~~~
这个例子里 a 元素的DOM属性有“href、id和action”，很明显，前两个是固有属性，而后面一个 “action” 属性是我们自己自定义上去的，
a 元素本身是没有这个属性的。这种就是自定义的DOM属性。处理这些属性时，建议使用attr方法。使用prop方法取值和设置属性值时，
都会返回undefined值。

再举一个例子：
~~~ html
	<input id="chk1" type="checkbox" />是否可见
	<input id="chk2" type="checkbox" checked="checked" />是否可见
~~~
像checkbox，radio和select这样的元素，选中属性对应“checked”和“selected”，这些也属于固有属性，因此需要使用prop方法去操作才
能获得正确的结果。
~~~ javascript
	$("#chk1").prop("checked") == false
	$("#chk2").prop("checked") == true
~~~		
如果上面使用attr方法，则会出现：
~~~ javascript
	$("#chk1").attr("checked") == undefined
	$("#chk2").attr("checked") == "checked"
~~~		
全文完。
	