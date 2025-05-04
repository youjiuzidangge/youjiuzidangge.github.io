---
layout: post
title:  Ruby方法 dup and clone 的探究
#时间配置
date:   2017-08-05 22:44：00 +0800
#大类配置
categories: Ruby
#小类配置
tag: 技术实战
---

* content
{:toc}


今天查看 Ruby 的dup方法才发现原来 Ruby 中也有深层复制和浅层复制的区别。但和我之前学的 PHP 又有些不同，觉得很有趣，因此总结如下。同样，有前辈的轮子——[ruby复制对象的方法(dup 和 clone)](http://blog.csdn.net/cloudcraft/article/details/10348799),简单转载。

----------------------------------
Ruby内置的方法Object#clone和Object#dup可以用来copy一个对象，两者区别是dup只复制对象的内容，而clone还复制与对象相关联的内容，如singleton method 
~~~ ruby 
	[ruby] view plaincopyprint?  
	s = "cat"    
	def s.upcase    
		"CaT"    
	end    
	s_dup = s.dup    
	s_clone = s.clone    
	s_dup.upcase        #=> "CAT"  (singleton method not copied)    
	s_clone.upcase      #=> "CaT" (uses singleton method)    
~~~	  
dup和clone都是浅复制Shallow Copy，也就是只能复制接受方的内容，而如果接受方包含到其他对象的引用，那么就只是会复制这些引用了。 
~~~ ruby  
	[ruby] view plaincopyprint?  
	arr1 = [ 1, "flipper", 3 ]    
	arr2 = arr1.dup    
	arr2[2] = 99    
	arr2[1][2] = 'a'    
	arr1                #=> [1, "flapper", 3]    
	arr2                #=> [1, "flapper", 99] 
~~~	
可以看到arr1中有一个到String对象的引用，从而arr2也复制了这个引用，当arr2中修改这个引用时，arr1中的也会发生变化。  
如果要进行深复制Deep Copy，可以聪明的采用Marshal模块  
~~~ ruby 
	[ruby] view plaincopyprint?  
	arr1 = [ 1, "flipper", 3 ]    
	arr2 = Marshal.load(Marshal.dump(arr1))    
	arr2[2] = 99    
	arr2[1][2] = 'a'    
	arr1                #=> [1, "flipper", 3]    
	arr2                #=> [1, "flapper", 99]   
~~~	
现在就会发现arr2中对String对象的修改不会导致arr1的变化了，因为深了。。。不过Marshal模块并不能把所有的对象都序列化  

在class中还有一个与对象复制相关的特殊方法initialize_copy，这个方法会在信息复制完成后执行，看下面这个示例  
~~~ ruby 
	[ruby] view plaincopyprint?  
	class Document    
		attr_accessor :title, :text    
		attr_reader :timestamp    
		
		def initialize(title, text)    
			@title, @text = title, text    
			@timestamp = Time.now    
		end    
	end    
		
	doc1 = Document.new("Random Stuff", "Haha")    
	sleep 10    
	doc2 = doc1.clone    
		
	doc1.timestamp == doc2.timestamp        #=> true  
~~~	
也就是两个对象是完全一样的，构造函数initialize被跳过了，所以两个对象的时间戮timestamp是相同的。如果要采用执行复制操作时的时间，我们可以通过给Document类添加initialize_copy方法来实现。initialize_copy让程序员能完全控制对象复制的状态 
~~~ ruby 
	[ruby] view plaincopyprint?  
	class Document    #Reopen the class    
		def initialize_copy(other)    
			@timestamp = Time.now    
		end    
	end    
		
	doc3 = Document.new("More Stuff", "Haha")    
	sleep 10    
	doc4 = doc1.clone    
		
	doc3.timestamp == doc4.timestamp        #=> false    
	再次感慨Ruby的魅力。。。  
	PS：以上内容主要来自The Ruby Way  
~~~	  
用Ruby复制一个对象(object)也许没有你想像的那么容易. 今天我google了半天, 做个总结吧.  
先从最简单的开始, b = a 是复制吗? 看代码说话:  
~~~ ruby 
	>> a= [0,[1,2]]  
	>> b=a  
	>> b[0]=88  
	>> b[1][0]=99  
	>> b    
	=> [88, [99, 2]]  
	>> a    
	=> [88, [99, 2]]  
~~~	
从上面代码发现, 一但修改b, 原来的a也同时被改变了. 甚至:  
~~~ ruby 	  
	>> b.equal?(a)  
	=> true
~~~	
原来b跟a根本就是同一个object, 只是马甲不一样罢了. 所以b = a不是复制.  
那 b = a.dup呢?? 还是看代码: 
~~~ ruby 	
	>> a= [0,[1,2]]  
	>> b=a.dup  
	>> b[0]=88  
	>> b[1][0]=99  
	>> b  
	=> [88, [99, 2]]  
	>> a  
	=> [0, [99, 2]]
~~~	
情况似乎有所好转, 在修改b后, a还是有一部分被修改了.(0没有变,但原来的1变成了99).  
所以dup有时候是复制(如在Array只有一级时), 但有时不是复制哦.  
再来一个, b = a.clone呢? 上代码:  
~~~ ruby 	
	>> a= [0,[1,2]]  
	>> b=a.clone  
	>> b[0]=88  
	>> b[1][0]=99  
	>> b  
	=> [88, [99, 2]]  
	>> a  
	=> [0, [99, 2]]  
~~~	
情况几乎跟dup一模一样. 所以clone也不一定可以相信哦!   
原来ruby中的dup和clone都是shallow复制, 只针对object的第一级属性.   
汗, 难道在Ruby中没有办法复制对像吗? 也不完全是, 看这个: 
~~~ ruby 	
	>> a= [0,[1,2]]  
	>> b=Marshal.load(Marshal.dump(a))  
	>> b[0]=88  
	>> b[1][0]=99  
	>> b  
	=> [88, [99, 2]]  
	>> a= [0,[1,2]]  
	=> [0, [1, 2]] 
~~~	
修改b后a没有被改变!!! 似乎终于成功找到复制的办法了!!!  
为什么要加"似乎"呢? 因为有些object是不能被Marshal.dump的.如:  
~~~ ruby 	
	>> t=Object.new  
	>> def t.test; puts ‘test’ end  
	>> Marshal.dump(t)  
	TypeError: singleton can’t be dumped  
		from (irb):59:in `dump’  
		from (irb):59  
~~~		
更完善的复制方案可以考虑给ruby增加一个deep clone功能, 可以参考以下链接:  
http://d.hatena.ne.jp/pegacorn/20070417/1176817721  
http://www.artima.com/forums/flat.jsp?forum=123&thread=40913 



