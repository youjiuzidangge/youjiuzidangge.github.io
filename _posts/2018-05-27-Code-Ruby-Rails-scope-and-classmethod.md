---
layout: post
title:  Ruby 中各种继承方法探究以及单例方法分析
#时间配置
date:   2017-10-18 20:30:00 +0800
#大类配置
categories: Ruby
#小类配置
tag: 技术实战
---

* content
{:toc}

好久没有更新博客了。
`世间最难的事是不改初衷，持之以恒的做某件事`

今天温习自己的笔记，查找文档之余，无意间看到了这篇[译文](https://blog.csdn.net/lissdy/article/details/51107883)，觉得很不错，所以转载过来。

--------------------------------------
我们在项目中应该使用scopes还是类方法来保持统一性？网上关于这个问题的讨论有很多。经典的言论往往归结于“两者没有不同”或者“口味问题”。我相信这种说法，但还是想要展示这两者之间存在的略微差异。

## 定义一个scope

首先，让我们来深入了解一下scope的使用。

在Rails 3中，可以使用以下两种方式定义scope：

	class Post < ActiveRecord::Base
		scope :published, where(status: 'published')
		scope :draft, -> { where(status: 'draft') } 
	end

两种定义方式的主要差别是：前者在类初次加载时生效；后者在被调用时才生效。因此，在Rails 4中，第一种定义方式将被弃用，这意味着以后需要一个可调用的对象作为参数来声明scope。这样做是为了避免声明带时间的scope时所引发的问题：

以下代码不会像期望的那样去运行，1.week.ago在类初次加载时生效，而不是在scope每次被调用时生效。

	class Post < ActiveRecord::Base
		scope :published_last_week, where('published_at >= ?', 1.week.ago)
	end

## Scope就是类方法

Active Record内部将scope转换为类方法。其在Rails中的简易实现如下：

	def self.scope(name, body)
		singleton_class.send(:define_method, name, &body)
	end

就像下面这个类方法一样，它（scope）是一个以name和body为参数的类方法：

	def self.published
		where(status: 'published')
	end

我想，这就是为什么大部分人认为“既然scope只是类方法的语法糖，为什么我还要用它？”，而下面就是针对这个问题的几个有趣的例子。

## Scope总是可链接的

让我们来设想以下场景：用户通过状态来过滤文章，通过最近更新来排序。很简单，下面是我们为这个需求编写的scope：

	class Post < ActiveRecord::Base
		scope :by_status, -> status { where(status: status) }
		scope :recent, -> { order("posts.updated_at DESC") }
	end

通过以下方式来调用：

	Post.by_status('published').recent
	# SELECT "posts".* FROM "posts" WHERE "posts"."status" = 'published' 
	# ORDER BY posts.updated_at DESC

目前为止，一切都没什么问题。现在我们将逻辑移到类方法中来做一个对比。

	class Post < ActiveRecord::Base
		def self.by_status(status)
			where(status: status)
		end

		def self.recent
			order("posts.updated_at DESC")
		end
	end

除了代码行数变多之外，看起来没什么不同。但是现在如果status参数为nil或者blank时，会发生什么呢？

	Post.by_status(nil).recent
	# SELECT "posts".* FROM "posts" WHERE "posts"."status" IS NULL 
	# ORDER BY posts.updated_at DESC

	Post.by_status('').recent
	# SELECT "posts".* FROM "posts" WHERE "posts"."status" = '' 
	# ORDER BY posts.updated_at DESC

糟糕，我不认为我们想做这样的查询。使用scope，我们可以通过为其添加一个是否存在的判断条件来轻松修复这个问题。

	scope :by_status, -> status { where(status: status) if status.present? }

现在的调用结果是酱的：

	Post.by_status(nil).recent
	# SELECT "posts".* FROM "posts" ORDER BY posts.updated_at DESC

	Post.by_status('').recent
	# SELECT "posts".* FROM "posts" ORDER BY posts.updated_at DESC


太棒了，现在我们来对类方法做同样的改造：

	class Post < ActiveRecord::Base
		def self.by_status(status)
			where(status: status) if status.present?
		end
	end

运行如下：

	Post.by_status('').recent
	NoMethodError: undefined method `recent' for nil:NilClass

炸裂，两者的区别是scope总是返回一个relation对象，然而类方法并不是这样（此处返回空：译注）。

类方法可以替代成这样：

	def self.by_status(status)
		if status.present?
			where(status: status)
		else
			all
		end
	end


注意，当status为nil/blank时，返回的结果为all，在Rails 4中返回的是一个relation对象（之前的版本返回的是结果集的数组），因此，在Rails 3.2.X中，你应该使用scope来替代这种需求（Rails 3.2.X返回的是数组，并不是relation对象，因此对于这种需求，必须使用scope：译注）。

这里给出的建议是：

永远不要在一个希望其具有可链接性的类方法中返回nil，应该总是返回一个relation对象，否则将会破坏其可链接性。

##  scope是可扩展的

我们以分页来作为下一个例子（使用kaminari这个gem来做支持），在对一个集合进行分页的时候，最重要的是确定要对哪一页的数据进行抓取。

	Post.page(2)


以及每一页的条目数

	Post.page(2).per(15)

另外，你可能还想要知道页面总数，或者当前是否位于首页或尾页。

	posts = Post.page(2)
	posts.total_pages # => 2
	posts.first_page? # => false
	posts.last_page?  # => true

顺序执行上述代码将得到与注释相同的结果，但是在对一个没有分页的集合调用这些方法时，没有任何效果。在编写scope时，可以加特殊的扩展：只有当对象中的scope被调用时才生效。在kaminari中，只对继承了Active Record的model添加了page这个scope，通过scope扩展这一特性，在page这个scope被调用时，加入其他scope（例如：per\total_pages\first_page?\last_page?等：译注）。从概念上来说，这里的实现代码如下：

	scope :page, -> num { # some limit + offset logic here for pagination } do
		def per(num)
			# more logic here
		end

		def total_pages
			# some more here
		end

		def first_page?
			# and a bit more
		end

		def last_page?
			# and so on
		end
	end

scope扩展是一个强大且灵活技术。当然，这种需求我们也可以借助于类方法以比较野蛮的方式实现：

	def self.page(num)
		scope = # some limit + offset logic here for pagination
		scope.extend PaginationExtensions
		scope
	end

	module PaginationExtensions
		def per(num)
			# more logic here
		end

		def total_pages
			# some more here
		end

		def first_page?
			# and a bit more
		end

		def last_page?
			# and so on
		end
	end 

虽然产生的结果相同，但是相比于scope，这种实现方式比较啰嗦

## 总结

当逻辑是简单的where/order这样的查询时使用scope；涉及到复杂查询时，使用类方法（是否接收参数倒不是问题的关键）。
另外，当需要做一些扩展时，作为Active Record提供的一个特性，我还是推荐使用scope。

我认为阐明scope和类方法的不同是一件重要的事情，这样才能在工作中得到最优方案。