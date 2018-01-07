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

数月之前看完 ruby 元编程，觉得自己功力大进，但是今天做 active job 延迟任务时，在网上看到一篇文章，竟然有些看不懂代码，只隐隐记得之前是看过相关文章，进行过相关学习的。那一刻实时的感觉到
记录的重要性。反观自己，几个月都没有写博客了，难道我这几个月都没有什么学习，都没有什么长进吗？业精于勤而荒于嬉，老大不小了，不能总是这样对自己放任自流了。

文章地址如下——[Cool Way to Control Retry with ActiveJob](http://necojackarc.hateblo.jp/entry/2016/08/27/195440)

##### 这里先简单分析下该文章中的代码：

    module ActiveJobRetryControlable
      # rails 中的 module，引入该 module 后，就可以对当前的 module 直接使用 include 同时给一个类加入类方法和实例方法
      extend ActiveSupport::Concern
      DEFAULT_RETRY_LIMIT = 5

      attr_reader :attempt_number

      module ClassMethods
        def retry_limit(retry_limit)
          @retry_limit = retry_limit
        end

        def load_retry_limit
          @retry_limit || DEFAULT_RETRY_LIMIT
        end
      end

      # 这里的 serialize 是 active job 中自定义的方法，而不是 active record 中的方法，这里困扰了我很久
      def serialize
        super.merge("attempt_number" => (@attempt_number || 0) + 1)
      end
      
      #同上理
      def deserialize(job_data)
        super
        @attempt_number = job_data["attempt_number"]
      end

      private

      def retry_limit
        self.class.load_retry_limit
      end

      def retry_limit_exceeded?
        @attempt_number > retry_limit
      end
    end

代码分析：首先 extend ActiveSupport::Concern 模块，那么在某个类 A 中 include 进该模块，就可以实现同时给该A类扩展类方法和实例方法。然后，注意其中的 serialize 和 deserialize 方法，见[文档](http://api.rubyonrails.org/classes/ActiveJob/Core.html)。在新建对象 a 后，每次读取实例变量 @attempt_number 都会调用 serialize 方法给 attempt_number 进行赋值，然后用 deserialize 方法取得它的值。从而达到改变实例变量值的目的。

### include，extend，require，load 方法基础比较

    include： 包含进入类的方法都为实例方法
    extend： 扩展进入类的方法都为类方法
    require： 只会加载一次库，当重复加载时后面的加载会返回 false。
    load：类似 require，允许重复加载

### 扩展运用

1. 但假如像上面的例子中我想一次性加载类方法和实例方法，并且我不用 rails 自带的库，那该如何处理呢？

这里就需要用到一个钩子函数了，self.included(class_name), 例子如下

    module F
      def self.included(class_name)
        class_name.send(:include, ObjectMethods)
        class_name.extend ClassMethods
      end

      module ObjectMethods
        xxxxx
      end

      module ClassMethods
        xxxxx
      end
    end

当一个类 include F 时，会首先执行 included 方法，剩下的就一目了然了。

2. 如果想得到一个类方法，有哪些方法呢？

我们来分析基本的添加类方法：

第一，直接在 class 中添加，那么基本的有三种：self.method, class_name.method, class << self end (单例类)

第二，通过外部加入的基本方法有：extend

第三，根据上面的单例类我们可以进一步分析。

  1）通过单例类的变种来得到，即想办法得到单例类的实例方法即可
    singleton_class.class_eval，singleton_class.instance_eval

  2）所谓的类方法，就是该类所属的类的实例方法而已。所以可以进一步分析得到，即 Class 类的实例方法，是所有类的类方法。而对 Object 类更有趣，因为所有的类的父类都是 Object，包括 Class。那么一个普通类可以说是 Object 子类的一个实例而已，所以 Object 中的实例方法也一定是类方法。但同时，Object 的类也是 Class 类的对象，那么 Object 类也是一个普通类的父类，那么从这个角度来说，Object 类中的方法一定是普通类的实例方法。总上，Object 类中的方法既是普通类的类方法也是实例方法。

完。