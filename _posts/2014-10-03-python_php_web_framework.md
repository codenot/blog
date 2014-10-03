---
layout: post
category: 学习笔记
title: python php web框架 说明
tags: [php, python, web]
---


`PHP` 本质上是四部分组成.

1. `语言和语言运行时环境（PHP Script）` 写一个 *.php 的脚本然后在终端运行，往往可以和 sh 脚本或者 python 脚本等价，这里就只用到了 PHP 的语言运行时。
2. `标准库` 也就是除了上述的最小子集外，PHP 中可以用到的一系列内置函数、内置类。
3. `Web 运行时` 这部分负责的就是将 CGI 模式的文本输入流封装，产生 $_GET、$_POST、$_COOKIE 和一系列和 Web 相关的支持，同时在脚本执行时候将标准输出（echo打印出来的内容）也添加协议中的附加内容（Content-Type、状态码、响应头等）输出。
4. `模版引擎` PHP 比较不同的是还内置模版引擎，这个模版引擎和其他语言中作为工具的模版有点不同，PHP 在语法解析的层面上是原生支持和模版混写的，所有没有包在 <?php?>中的文本都不作为语法解析，而是直接作为输出，PHP 的模版也籍此实现。


<!--more-->

`Python` 的目标是作为通用工具语言,一个新安装的python包含以下两个部分:

1. 语言和语言运行时环境（Python 虚拟机）
2. `标准库`（Python 的标准库是出名的给力，覆盖面极广）

不同于PHP, Python的 Web 并不作为语言的一部分实现.

{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}


![Alt text](./0_12839166559zGP.gif)


> WSGI 定义的标准将 Web 应用划分为 `WSGI Application` 和 `WSGI Server`。
> 
>   * `WSGI Server`类似 PHP 的 `Web 运行时`，提供对标准输入输出流的封装，
>   * `WSGI Application`则类似自己写的 PHP 应用，在封装后的环境中对具体应用进行 Web 开发。



Python 开发一定要用 `Web 框架`吗？不一定，但是用一个良好封装了的 `WSGI Application 框架`可以让你不用自己去解析请求头、拼接响应头，如果不用 Web 框架相当于把 PHP 剥离到只能写脚本。而用一个框架，尤其是 `WSGI Application 框架`，不仅为低层协议提供了良好的封装，还能在 `WSGI` 公约下和广大 `WSGI Server` 协作。


至于 Python 能不用像 PHP 一样内嵌到模版中，历史上到曾经有过一个叫 `PSP（ Python Server Page）`的东西，现在应该记得的人不多了吧。用一个领域逻辑、持久化、控制流、视图逻辑层层分的清清楚楚的开发方式，不是比什么都堆在一个页面里面要优雅的多吗？

---

## 术语说明

`CGI` : CGI是为了保证web server传递过来的数据是标准格式的，方便CGI程序的编写者

> `web server（比如说nginx）`只是内容的分发者。比如，如果请求/index.html，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。好了，如果现在请求的是/index.php，根据配置文件，nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。

> Nginx会传哪些数据给PHP解析器呢？url要有吧，查询字符串也得有吧，POST数据也要有，HTTP header不能少吧，好的，`CGI`就是规定要传哪些数据、以什么样的格式传递给后方处理这个请求的协议。仔细想想，你在PHP代码中使用的用户从哪里来的。

> 当web server收到/index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

`FastCGI`: 提高性能

> 提高性能，那么CGI程序的性能问题在哪呢？"`PHP`解析器会解析`php.ini`文件，初始化执行环境"，就是这里了。标准的`CGI`对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！那么`Fastcgi`是怎么做的呢？首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

`PHP-FPM` : 是一个实现了`Fastcgi`的程序，被PHP官方收了。

> 大家都知道，PHP的解释器是`php-cgi`。`php-cgi`只是个`CGI`程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的`spawn-fcgi`。好了`PHP-FPM`也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨`PHP-FPM`稳定性太差的），也越来越流行。

`spawn-fcgi` : 同`php-fpm`

`Flup`: 使用 Python 语言对 `WSGI Server( == PHP Web运行时)`的一种实现

> `CGI`中所谓的`Server/Gateway`，它负责接受apache/lighttpd转发的请求，并调用你写的程序(application)，并将application处理的结果返回到apache/lighttpd
PS. `PHP解释器(php-cgi)`自带了此功能,所以PHP不需要这个东西.


`WSGI`: 除了`flup Server/Gateway`外还有很多其他人的写的`Server/Gateway`, 这个时候就会出问题了，如果你在flup上写了一个程序，现在由于各种原因你要使用xdly了，这个时候你的程序也许就要做很多痛苦的修改才能使用 xdly server了，`WSGI`就是一个规范，他规范了`flup`这个服务应该怎么写，应该使用什么方式什么参数调用你写的程序(application)等，当然同时也规范你的程序应该怎么写了，这样的话，只要flup跟xdly都遵守WSGI的话，你的程序在两个上面都可以使用了，flup就是一个`WSGI server`

`uWSGI` : uwsgi同WSGI一样是一种通信协议，而uWSGI是实现了uwsgi和WSGI两种协议的Web服务器。
