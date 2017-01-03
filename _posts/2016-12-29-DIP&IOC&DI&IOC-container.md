---
layout: post
title:  DIP、IoC、DI 及 IOC 详细梳理与深入理解
#时间配置
date:   2016-12-29 19:30:00 +0800
#大类配置
categories: 代码
#小类配置
tag: 思想汇总
---

* content
{:toc}



在网上看了好多这一块的教程，感觉讲的太粗略。对于一些高手来说，自然是绰绰有余，但是对于我这种小菜鸟，感觉总差那么一点点东西。
这个东西是什么呢？就是细节，包括事情的来龙去脉，包括复杂代码的分析等等。本篇博客就是基于此，在一些前辈领路人的基础上所写的。

-------------------------------------------

概念总览
===========================================
首先让我们理一理这几个概念的先后关系。

我们都知道我们的设计思想可以分为两种，即面向对象设计（OOD）以及面向过程设计（OPD）。

其中 OOD 有什么好处呢？它有助于我们开发出高性能、容易扩展以及易复用的程序。
由此，应运而生一大批软件设计原则，而今天的主角 DIP 就是其中之一。

具体，见以下：

	DIP ：依赖倒置原则，基于 OOD 的一种软件设计的原则
	IoC : 控制反转，DIP 的一种具体实现方式，一种反转流、依赖和接口的方式。
	DI  : 依赖注入，IoC 控制反转的一种方法
	IOC容器 ：依赖注入(DI)的自动化实现。DI的一种框架，用来映射依赖，管理对象创建和生存周期。

由以上可知，`由 OOD 思想，产生了 DIP 设计原则；由 DIP 原则，发展出 IOC 接口方式来实现这个原则；而 DI 则是这个 IoC`

`的接口方法，我们运用该方法即做出了一个 IoC 容器`。

具体案例
===========================================

依赖倒置原则（DIP）
-------------------------------------------

说依赖倒置之前我们谈谈依赖不倒置的情况，比如说我们建设楼房，肯定是要先打地基的，假设地基上对应有规范，而房子就是根据规范来建设的。
我们可以说地基是基础（底层模块），而向上的一层层房子则是在这个基础上做出来的（高层模块）。显然，`高层模块必须依赖于底层模块`。
假设我们有这样的需求，即我现在需要需要修改地基的规范，在该地基上做的房子自然要跟着变化。这样比较浪费资源。

那么，我们可以让依赖反转，`让底层模块来依赖高层模块`。这个时候，我是先建设房子，来根据房子的接口需要来打地基。因为我是先有的房子，
所以这个时候的地基是什么样子对我来说无所谓，只要有我房子需要的接口就可以了，当然最终我们还是需要地基和房子合起来才是完整的楼房。
这个时候我们具体怎么做呢？就是我来规定我们的房子需要什么样的接口，只要你地基上有这种接口，我们就可以用。

为什么我们一定要需要修改底层代码呢？因为一个新的框架出来，功能肯定不够完善，我们以后为其添加功能，不能影响已经写好的代码。
<p><img style="width:100%" src="{{ '/styles/images/DIP.png' | prepend: site.baseurl }}" /></p>

控制反转（IoC）和 依赖注入(DI)
-----------------------------------------
上面 DIP 是一种 软件设计原则，它仅仅告诉你两个模块之间应该如何依赖，但是它并没有告诉如何做。
而 IoC 则是一种 软件设计模式，它告诉你应该如何做，来解除相互依赖模块的耦合。

`控制反转（IoC），它为相互依赖的组件提供抽象，将依赖（低层模块）对象的获得交给第三方（系统）来控制，`

`即依赖对象不在被依赖模块的类中直接通过new来获取。`

懒的敲代码，摘录另一位博主的例子:

	//依赖不倒置，其中 Sword 相当于本人例子中的地基（应该是某个地基），Hero 相当于房子。
	class Sword
	{
		private $title;
		public function __construct($title)
		{
			$this->title = $title;
		}
	}
	class Hero
	{
		private $weapon;
		public function __construct()
		{
			// Hero依赖Sword
			$this->weapon = new Sword('倚天剑');
		}
	}
	
	--------------------------------------------------------------
	//依赖倒置
	//假设我们现在需要一个枪类，上面的代码就不好使了
	//这里把各类不同的武器抽出来，统统做成一个武器的接口
	interface I_Weapon
	{
		public function __construct($title);
		/**
		 * 进攻
		 */
		public function attack();
	}
	 
	/**
	 * 宝剑类
	 */
	class Sword implements I_Weapon
	{
		private $title;
		public function __construct($title)
		{
			$this->title = $title;
		}
		public function attack()
		{
			return '唰唰唰';
		}
	}
	 
	/**
	 * 枪类
	 */
	class Gun implements I_Weapon
	{
		private $title;
		public function __construct($title)
		{
			$this->title = $title;
		}
		public function attack()
		{
			return '砰砰砰';
		}
	}
	 
	/**
	 * 英雄类
	 */
	class Hero
	{
		private $weapon;
		public function __construct(I_Weapon $weapon)
		{
			// Hero依赖的武器, 来自于构造时的参数输入,
			// 这就将依赖的对象通过注入的方式, 注入到Hero中
			$this->weapon = $weapon;
		}
	}
	
	//使用如下
	//用宝剑的英雄
	new Hero(new Sword('倚天'));
	// 用枪的英雄
	new Hero(new Gun('沙漠之鹰'));
	
上一段代码即为依赖注入 DI 的`构造函数注入`的实现方式，IoC的思想包含在其中(把所有武器抽出来做个接口即是)。
还有`属性注入`的方式，大同小异，不详述。

IoC容器:依赖注入(DI)的自动化实现
-----------------------------------------

上面的代码，只有两种武器，假设我们需要第3中，甚至30种武器。每次都新加类，显然也是不实际的。

因此，我们就引入了IoC容器。每次只要我们需要新的武器，自动添加上例中的新加类的功能。

	class IoC_Container
	{
		private static $generator_list = [];
		// 绑定类的生成器
		public static function bind($class_name, $generator)
		{
			if (is_callable($generator)) {
				self::$generator_list[$class_name] = $generator;
			} else {
				throw new Exception('对象生成器不是可调结构');
			}
		}
	}
	

分析 bind 方法，有两个参数:

+ 第一个就是类的标志, 通常就是带有命名空间的类名称即可. 例如 path\to\Sword, path\to\Hero.

+ 第二个参数就是该类对应的生成器. 所谓生成器, 其实就是一段代码, 可以生成类对象的代码而已, 说白了就是执行new的代码. 
可以是匿名函数, 函数, 类方法等可执行结构都是可以的.

`以上代码其实就是我们把被调用函数（即生成器）存入类 Ioc_Container 中`。

	// 利用容器绑定对象生成器
	IoC_Container::bind('Sword', function($title='') {
		return new Sword($title);
	});
	IoC_Container::bind('Gun', function($title='') {
		return new Gun($title);
	});
	IoC_Container::bind('Hero', function($module, $params=[]) {
		return new Hero(IoC_Container::make($module, $params));
	});
	
分析这一段代码前两段，我们是先准备好各类武器。

	$generator_list[$weapon] = function($title="")
	{
		return new Weapon;
	}

而第三段的结果，核心在于生成一个 Hero 实例。而生成 Hero	实例，需要传入一个 Weapon 的实例。
即 IoC_Container::make($module, $params) 这一段代码的作用在于生成一个 Weapon 实例。

所以我们只需要调用 $generator_list[$weapon] 并传入参数，就可以新建一个对应的 Weapon 类了。

	Class Ioc_Container
	{
		private static $generator_list = [];
		// 绑定类的生成器
		public static function bind($class_name, $generator)
		{
			if (is_callable($generator)) {
				self::$generator_list[$class_name] = $generator;
			} else {
				throw new Exception('对象生成器不是可调结构');
			}
		}
		// 生成类对象
		public static function make($class_name, $params = [])
		{
			if (! isset(self::$generator_list[$class_name])) {
				throw new Exception('类没有被绑定注册');
			}
			return call_user_func_array(self::$generator_list[$class_name], $params);
		}
	 
	}

通过make生成类对象
	
	IoC容器生成对象
	$hero1 = IoC_Container::make('Hero', ['Sword', ['倚天']]);
	var_dump($hero1);
	$hero2 = IoC_Container::make('Hero', ['Gun', ['沙漠之鹰']]);
	var_dump($hero2);

分析这里通过 make 来实例化 Hero ：

先看 call_user_func_array(self::$generator_list[$class_name], $params) 。self::$generator_list['Hero'] 相当于一个函数

	function($module, $params=[])
	{
		return new Hero(IoC_Container::make($module, $params));
	}
	
然后将后面的参数 ['Gun', ['沙漠之鹰']] ，传入该函数，即 IoC_Container::make('Gun', '沙漠之鹰') 。这里是再执行一遍 make 方法，但是
参数变了，即所调用的对象变了。这个时候的 call_user_func_array(self::$generator_list[$class_name], $params) 中的 self::$generator_list['Gun'] 相当于函数

	//之前 bind 绑定的对象
	function($title="")
	{
		return new Gun($title);
	}

然后传入一个 '沙漠之鹰' 的参数，即生成一个新的 Gun 实例。

我看到有的博主称 make 的这个过程为“解析”，感觉十分贴切。这里简而言之，就是一步步解析出 Hero 所依赖的对象 Weapon，再解析 Weapon，然后再反过来生成 Weapon 到 Hero 结束。
	
写到我想吐，不完整的地方以后再补充吧。。。。。。。。。。。。。。。。。。

注：这篇博客主要参考一下两个博客写成，可参考 [木小楠.NET](http://www.cnblogs.com/liuhaorain/p/3747470.html) 和 [小韩说理](http://www.hellokang.net/2016/11/13/ioc-container/)