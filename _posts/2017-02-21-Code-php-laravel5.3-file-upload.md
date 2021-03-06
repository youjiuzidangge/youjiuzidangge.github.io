---
layout: post
title:  laravel5.3 文件上传
#时间配置
date:   2017-02-21 20:33:00 +0800
#大类配置
categories: PHP
#小类配置
tag: laravel
---

* content
{:toc}


写在前面的话——

最近总是焦急，其实是没有必要的。能力到哪里，就思考到哪里。 ————作者

今天做 laravel 文件上传才发现自己好久没有写过图像上传的代码了，竟然忘记的一干二净。网上搜索 laravel 上传的教程，

搜索了半天也没有一个特别合适的。不过在综合了几篇文章以及 laravel 的官方文档后，自己也总算是摸清楚了。因此自己写

一个教程来巩固一下。

这里同时也暴露了自己一个很大的问题——

我一直告诉自己，开始学习一件新的事物时，一定要从整体把握其脉络开始。但我学习 laravel 框架，首先应该做的第一步

应该就是把 laravel 框架整体的教程了解一遍。但我却迟迟没有开始，每次遇到问题都是一顿瞎搜，事倍功半。

一般上传
-----------------------------------------
首先来说说文件上传这件事，上传本身是很简单的，只需要如下即可
~~~ html
	<form enctype="multipart/form-data" action method>
		<input type='file' name="filename" />
		<input type="submit" value="提交" />
	</form>
~~~
然后用 $_FILE 来获取上传的信息。再根据上传的信息来判断是不是我们需要或者允许的文件。（这一步其实是可有可没有，可简单可复杂，看个人需求。）

这里需要注意的是，上传的文件首先上传到服务器的临时目录中，即 php.ini 中 upload_tmp_dir 设置的临时目录。（但一般不允许直接修改 php.ini,因为是买的服务器。

可以用函数 ini_set('upload_tmp_dir', 'C:/temp') 来设置。）这个临时的复制文件会在脚本结束时消失。要保存被上传的文件，我们需要把它拷贝到另外的位置，使用

move_uploaded_file($_FILES["file"]["tmp_name"]) 即可。后面对该文件怎么处理就是每个人自己的事啦~

laravel5.3 上传
------------------------------------------
使用框架的好处就是把相对复杂的事情简单话，或者是免去自己写的麻烦。

这里，第一步是
~~~ Textile
	在 config/filesystems.php 文件中配置一个本地磁盘空间。（laravel 目前支持 "local", "ftp", "s3", "rackspace" 4种磁盘中间）
~~~	
第二步
~~~ Textile
	拿到上传的信息，$request->file('filename')。它应该是一个 UploadedFile 类。
~~~	
第三步
~~~ Textile
	$request->file('photo')->isValid() //验证是否上传成功
	然后根据需要调用 UploadedFile 类方法 或者自己写方法 进一步处理即可
~~~	
最后
~~~ Textile
	保存文件，用该类的 store 或者 storeAs 方法即可。
	//返回的文件名是 md5 加密的
	$path = $request->photo->store('images'); //默认的磁盘空间
	$path = $request->photo->store('images', '自己配置的本地磁盘空间');

	//自定义文件名
	$path = $request->photo->storeAs('images', 'new_filename');
	$path = $request->photo->storeAs('images', 'new_filename', 's3');
~~~	

总结
------------------------------------------

1.是不是该了解下 config 下各项配置的意思？