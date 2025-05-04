---
layout: post
title:  HTTP 协议
#时间配置
date:   2017-08-13 16:41：00 +0800
#大类配置
categories: web 相关
#小类配置
tag: 知识体系
---

* content
{:toc}


关于 http 请求这块，我一直在学习，一直没完全懂，属于半吊子状态，现在是时候解决这个问题了！

参考博文——
[http 长连接和短连接](http://blog.jobbole.com/93960/)
[Http 协议详解](http://m.blog.csdn.net/suifeng3051/article/details/49530389)
[HTTP 协议漫谈](http://blog.jobbole.com/88199/)
[HTTP协议和HTTPS协议详解](http://blog.csdn.net/moonpure/article/details/52984877)：

----------------------------------

# HTTP 协议概念

<p><img src="{{ '/styles/images/2017-08-13_http_content_outline.png' | prepend: site.baseurl }}" /></p>

不知道大家有没有想过，为什么当我们打开一个网址，输入网址（谷歌）时，就可以看到一个网页？本质上它的过程很简单，就是浏览器（客户端）向谷歌（服务器端）请求了一次数据，而我们看到的网页就是谷歌给浏览器返回的内容。

这里面就有几个细节要考虑了。

+ 第一，既然是请求，那么我请求的内容是什么，为什么通过网址就可以确定请求的内容？
+ 第二，上面的步骤说白了就是交流，而且是面向所有人的交流，那么是不是应该像写报告或者书信一样定义一种格式？不然，每个人用自己的风格，那么就完全乱了套。

第一个问题的答案就是我们采用了一种叫做统一资源定位符（URIs）的技术，技术细节这里不展开，总而言之，就是通过该技术，我们可以直接定位到我们要请求的资源的具体位置，相当于告诉浏览器去把xx地方的xx东西拿给我。当然，这里还涉及到域名，与 IP 的知识点，可以参考本人其他博客。

第二个问题可以说是回答了 HTTP 的来源吧，或者说是 HTTP 的本质——它不过是对请求数据的格式，与响应数据的格式的一种规范。协议，即表示大家都同意使用的规范。（但显然，并没有征询过本人的意见。。。本人意见很大。。）HTTP（HTTP，HyperText Transfer Protocol） 中文名为超文本传输协议，顾名思义就是可以传输不止是文本的内容。

另外考虑到在互联网中，一个服务器往往需要处理大量的客户端请求，HTTP 协议应该尽可能少的占用系统资源。为了达到这个目的，http 协议被设计成无状态的。另外，Http协议是在TCP/IP之上的一种协议（Http属于应用层，TCP/IP属于运输层），默认端口采用的是80。这一句话怎么理解，得稍微了解下网络的七层模型。部分内容如下：

<p><img src="{{ '/styles/images/2017-08-13_http_position_in_network.png' | prepend: site.baseurl }}" /></p>


## HTTP 基础

现在来谈谈 HTTP 的具体内容，大体包含如下：
1.URL
2.请求方式
3.状态码
4. request
5. response


### 版本
我们现在用到的版本是HTTP/1.1，它比1.0版本添加了更多特性。其中比较重要的特性有（注意：HTTP1.1 默认连接是一直保持的）：
~~~ Textile
>	* 支持持久连接
>	* 支持消息切分成块传输
>	* 更加丰富的cache特性
>	* 带宽优化及网络连接的使用
>	* 错误通知的管理
>	* 互联网地址的维护安全性及完整性
~~~
这里值得注意的一点是：`HTTP1.1 默认连接是一直保持的`。这里容易矛盾的一点是我们前面说 HTTP 是无状态的，现在又说是默认长连接。首先，我们要明白这是两个不同的概念：

无状态；HTTP协议是无状态的，指的是协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。也就是说，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（无连接）。举个例子，就像两个人开展一段对话，这段对话就是一次 TCP 连接，但是我对你说话的内容，可以不用上下关联，我说的第一句和第二句没有任何关系，这就是无状态。

长连接：当我们访问一个网页的时候，获取的内容往往都不止一个 HTML，而浏览器对 HTML 的解析过程中，如果发现需要获取的内容（比如说css,images)等，会再次发起 HTTP 请求去服务器获取。实际上会有多个 HTTP 请求，但是这些请求只依靠一个 TCP 连接就够了。这就是所谓的持久连接。

综上，所以我们可以说：

`HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接`

### 具体内容

#### 结构体系

总的来分，HTTP 消息只有两种 request 请求和 response 反应消息。它们都是由三部分组成：
~~~ Textile
	1. start-line 开始行 
	2. header 消息头 
	3. body 消息体
~~~
对于 start-line，又分为：
~~~ Textile
	1. Request-Line : 'METHOD/path-to-resource http-version'
	2. Response-Line : 'http-version status-code message'
~~~

对于 Headers 则有如下几种：
~~~ Textile
	1. general headers
	2. entity headers
	3. request or response headers
		a. request specific headers.
		b. response specific headers.
~~~
下面我们再用两幅图具体看一下Request和Response的消息格式：

<p>
	<img style="width:45%" src="{{ '/styles/images/2017-08-13_http-request_structure.png' | prepend: site.baseurl }}" />
	<img style="width:45%"  src="{{ '/styles/images/2017-08-13_http-response_structure.png' | prepend: site.baseurl }}" />
</p>

####  HTTP headers

+ General Headers

通用头即可以包含在HTTP请求中，也可以包含在HTTP响应中。通用头的作用是描述HTTP协议本身。比如描述HTTP是否持久连接的Connection头，HTTP发送日期的Date头，描述HTTP所在TCP连接时间的Keep-Alive头,用于缓存控制的Cache-Control头等，具体有：
~~~ Textile
	general-header = Cache-Control           
				   | Connection       
				   | Date             
				   | Pragma           
				   | Trailer          
				   | Transfer-Encoding
				   | Upgrade          
				   | Via              
				   | Warning
			   
	
    Cache -Control指定请求和响应遵循的缓存机制。
    Connection 允许客户端和服务器指定与请求/响应连接有关的选项
    Date 提供日期和时间标志,说明报文是什么时间创建的
    Pragma头域用来包含实现特定的指令，最常用的是Pragma:no-cache
    Trailer 如果报文采用了分块传输编码(chunked transfer encoding) 方式,就可以用这个首部列出位于报文拖挂(trailer)部分的首部集合
    Transfer-Encoding 告知接收端为了保证报文的可靠传输,对报文采用了什么编码方式
    Upgrade 给出了发送端可能想要”升级”使用的新版本和协议
    Via 显示了报文经过的中间节点(代理,网嘎un)
~~~
+ Entity Headers

实体头是那些描述HTTP信息的头。既可以出现在HTTP POST方法的请求中，也可以出现在HTTP响应中。比如图5和图6（自行想象）中的Content-Type和Content-length都是描述实体的类型和大小的头都属于实体头。其它还有用于描述实体的Content-Language,Content-MD5,Content-Encoding以及控制实体缓存的Expires和Last-Modifies头等，具体有：
~~~ Textile
	entity-header  = Allow                   
				   | Content-Encoding 
				   | Content-Language 
				   | Content-Length   
				   | Content-Location 
				   | Content-MD5      
				   | Content-Range    
				   | Content-Type     
				   | Expires          
				   | Last-Modified
~~~
+ Request/Response Headers

到这里为止，我们是不区分请求和响应来讲的，也可以说是讲请求和响应的相同部分。但请求和响应毕竟不同，下面的话就必须分开来讲了。

### Request 消息体

由上面的内容我们可知，request 消息体应该包含三部分内容，request-line, request_header, request body。

+ request-line
~~~ Textile
	Request-Line = Method SP URI SP HTTP-Version CRLF
	Method = "OPTIONS"
		   | "HEAD"  
		   | "GET"  
		   | "POST"  
		   | "PUT"  
		   | "DELETE"  
		   | "TRACE"
~~~
	其中sp代码字段的分隔符，HTTP-Version一般就是"http/1.1"，后面紧接着是一个换行。
	
+ request header

requst-line 后面紧跟的就是 header。

请求头是那些由客户端发往服务端以便帮助服务端更好的满足客户端请求的头。请求头只能出现在HTTP请求中。比如告诉服务器只接收某种响应内容的Accept头，发送Cookies的Cookie头，显示请求主机域的HOST头,用于缓存的If-Match，If-Match-Since,If-None-Match头，用于只取HTTP响应信息中部分信息的Range头，用于附属HTML相关请求引用的Referer头等。
~~~ Textile
	request-header = Accept                   
				   | Accept-Charset    
				   | Accept-Encoding   
				   | Accept-Language   
				   | Authorization     
				   | Expect            
				   | From              
				   | Host              
				   | If-Match          
				   | If-Modified-Since 
				   | If-None-Match     
				   | If-Range          
				   | If-Unmodified-Since
				   | Max-Forwards       
				   | Proxy-Authorization
				   | Range              
				   | Referer            
				   | TE                 
				   | User-Agent
~~~
Request Headers扮演的角色其实就是一个Request消息的调节器。需要注意的是若一个headers名称不在上面列表中，则默认当做Entity Headers的字段
				   
前缀为Accept 的headers定义了客户端可以接受的媒介类型、语言和字符集等。From, Host, Referer 和User-Agent 详细定义了客户端如何初始化Request。前缀为If 的headers规定了服务器只能返回符合这些描述的资源，若不符合, 则会返回304 Not Modified。

+ request body

若Request-Line中的Method为GET，请求中不包含消息体，若为POST，则会包含消息体。

+ 最后我们看一个具体的 request 实例
~~~ Textile
	GET /articles/http-basics HTTP/1.1
	Host: www.articles.com
	Connection: keep-alive
	Cache-Control: no-cache
	Pragma: no-cache
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
~~~	
### Reponse 消息体

+ response-line
~~~ Textile
	Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
	# 例子
	HTTP/1.1 200 OK
~~~	
+ response-header

HTTP响应头是那些描述HTTP响应本身的头，这里面并不包含描述HTTP响应中第三部分也就是HTTP信息的头（这部分由实体头负责）。比如说定时刷新的Refresh头，当遇到503错误时自动重试的Retry-After头，显示服务器信息的Server头，设置COOKIE的Set-Cookie头，告诉客户端可以部分请求的Accept-Ranges头等。
~~~ Textile
	response-header = Accept-Ranges
					| Age
					| ETag              
					| Location          
					| Proxy-Authenticate
					| Retry-After       
					| Server            
					| Vary              
					| WWW-Authenticate
~~~					

Age 表示消息自server生成到现在的时长，单位是秒
ETag 是对Entity进行MD5 hash运算的值，用来检测更改
Location 被重定向的URL
Server 服务器标识

+ response body

就是我们看到的页面。
