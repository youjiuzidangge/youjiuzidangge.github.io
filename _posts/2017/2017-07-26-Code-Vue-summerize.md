---
layout: post
title:  Vue2+ 及相关技术总结
#时间配置
date:   2017-07-26 13:30:00 +0800
#大类配置
categories: Vue
#小类配置
tag: 技术实战
---

* content
{:toc}


最近再看阮一峰老师的博客，获益匪浅，略有感悟。总结如下

+ 文章条理非常清楚，语言简单易懂，说白了，就是他把他自己学习过的那些知识，首先揉碎，再通过总结梳理，得出一套自己的理解流程，然后再用简单的语言表述出来。
+ 把自己当作完全不懂的小白读者。每次梳理前因后果都要力求写的清楚简洁，但不能存在内容缺失情况。这里说明，阮老师肯定是做过大量的查阅工作的，同时也不放过一个细节
+ 接上，开始的时候要理清流程，画出骨架，然后再加入血肉
+ 一年的博客总量，撑死也就 10 篇左右，但质量很高，属于用心做精品

本篇博客是基于 rails5.1 的 vue2.0+ 来梳理的。

----------------------------------

## Vue 的安装及其简单配置
### 第一步：

对于新项目,首先对于 vue 一类的项目，需要使用到 webpack (前端资源加载/打包工具)，需要相应 gem , 如下执行即可

~~~ ruby
	gem install rails -v '5.1'
	rails new [your_project] --webpack
	#也可以 rails new [your_project] --webpack=vue 直接装上 vue
~~~
旧项目升级，同理，先添加相应 gem
~~~ ruby
	gem 'rails', '5.1'
	gem 'webpacker'
~~~	
接下来执行
~~~ ruby
	bundle update rails
	rails webpacker:check_yarn #添加 yarn 包管理文件
	#安装 webpack 并生成配置文件（config/webpack），这里要记得把 node_modules 加入到 .gitignore 中，不然会把开发环境里的安装包提交
	rails webpacker:install 
~~~
上面生成的 config/webpack 下的目录结构(后面补充每一个配置文件的作用)
~~~ ruby
	loader 文件夹
	configurations.js
	development.js
	production.js
	shared.js
	test.js
~~~	
接上，执行安装 vue
~~~ ruby
	#会在
	rails webpacker:install:vue
~~~	
### 第二步：
在入口文件中引入 js 打包文件等
~~~ ruby
	# 见 module Webpacker::Helper 有详细解释
	
	<%= javascript_pack_tag 'hello_vue' %> #对应的是 /app/javascript/packs/hello_vue.js
	<%= stylesheet_pack_tag 'hello_vue' %> #对应的是
	<%= javascript_pack_tag 'application' %> #同上
~~~
