---
layout: post
title:  "几种跨域问题方案(上传文件/IE89 CORS兼容)"
date:   2016-06-03 18:14
categories: cross-domain 跨域 jsonp cors proxy
---

这篇文章中会讲到

* 后端代理
* JSONP
* CORS
* IE8/9 CORS兼容 文件上传


## 跨域问题

正常情况下浏览器端的ajax请求需要遵循[同源策略], 实际应用中难免会遇到需要违返同源策略去做跨域请求的用例。



## 后端代理

我觉得最推荐的方式就是在后端做转发代理。
这样处理对于浏览器来说，没有违反同源策略，所以前端代码完全不需要特殊处理。
转发代理后端配置也极其简单，所以该方法是最为简便的方法。

缺点则是需要后端的配合，而且所有流量都要先经过后端的代理服务器。


## JSONP
JSONP 更多上来讲应该算是`hack`。
前端通过`<script src="http://非同源地址?callback=handle_data">`来请求，
后端根据前代的callbcak生成javascript代码片段, 将返结果作为callback函数的参数传送给请求处。

{% highlight js %}
handle_data({"data_1": "hello world", "data_2": ["the","sun","is","shining"]});
{% endhighlight %}

该方法优点是兼容性很好，几乎所有浏览器都可以支持。
缺点在于只能发送GET请求, 并且请求无法带上自定义HEADER/ withCredentials(cookie)等信息。而且本身来讲，应该算是一种`hack`技术。


## CORS

[CORS]则是属于正牌的有标准协议的处理方法。

前端在请求时，会先进行一个`预请求`, 询问服务端是否允许该跨域请求。

{% highlight text %}
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com   # 这条消息是从bob这里发出的请求，你要不要允许它跨域
Access-Control-Request-Method: PUT  # 这条消息想用PUT方法
Access-Control-Request-Headers: X-Custom-Header  # 这条消息还想带上这些信息头
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
{% endhighlight %}

在服务端则要实现对`预请求`的回复, 如

{% highlight text %}
Access-Control-Allow-Origin: http://api.bob.com # 允许bob站点的跨域
Access-Control-Allow-Methods: GET, POST, PUT  # 允许这些方法
Access-Control-Allow-Headers: X-Custom-Header  # 允许这些header
Access-Control-Allow-Credentials: true # 允许credentials
Content-Type: text/html; charset=utf-8
{% endhighlight %}

浏览器在收到预请求的回复后，如果服务端允许访问则会发出实际的请求，并会保存此次预请求结果，下一次同样的请求时则会直接发送实际请求，不会再次进行预请求。
如果服务器端不允许，则不会发出实际的请求。


该方法优点是属于有协议标准支持的，并非`hack`技术，可以支持header/各类method等
缺点在于一些老版本的浏览器不支持。


## CORS IE8 / 9

IE8 和 IE9 算是部分支持CORS, 这两个版本中所用的是`XDomainRequest`。
(记得看过IE团队的人说，IE9的比较新的几个版本也不用XDomainRequest, 直接全部支持, IE这个些小版本真是没法吐槽)

`XDomainRequest`的主要的几点限制:
* 只支持`GET`/`POST`
* 不能自定义`HEADER`
* 不能带`cookie`
* content-type必须是`text/plain`


之前做的例子本来是在header/cookie中带着token来做身份验证的，在这两个版本下只好放到了body和query里面去了。
因为自定义Header不支持的原因, `X-HTTP-Method-Override`也是不支持的，所以也就牺牲了`PATCH`, `DELETE`这些语义。

`jQuery`在IE8/9下要使用，可以直接加入该插件[jQuery-ajaxTransport-XDomainRequest].


## 文件上传

这块其实只是勉强和跨域擦边。因为IE9以下并不支持`FormData`，所以文件上传也无法通过正常的ajax发出来。 
不过还好[jQuery-File-Upload]对此支持做得很好，有[jQuery-iframe-transport]插件可以对其进行支持。

原理是通过插入`iframe`,并在在其中插入`form`, action直接指到跨域的url。
如果需要返回结果的话，需要后端API实现返回结果类似如下的HTML, js这边通过`iframe.load`事件获取获取结果传回主document.

{% highlight html %}
<textarea data-type="application/json">
  {"ok": true, "message": "Thanks so much"}
</textarea>
{% endhighlight %}


## 参考文章 / 深入阅读

* [Using CORS]
* [JQUERY.IFRAME-TRANSPORT.JS annotation]
* [XDomainRequest – Restrictions, Limitations and Workarounds] 




[Using CORS]: http://www.html5rocks.com/en/tutorials/cors/
[CORS]: http://www.html5rocks.com/en/tutorials/cors/
[XDomainRequest – Restrictions, Limitations and Workarounds]: https://blogs.msdn.microsoft.com/ieinternals/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds/
[jQuery-ajaxTransport-XDomainRequest]: https://github.com/MoonScript/jQuery-ajaxTransport-XDomainRequest
[jQuery-File-Upload]: https://github.com/blueimp/jQuery-File-Upload
[jQuery-iframe-transport]: https://github.com/cmlenz/jquery-iframe-transport
[JQUERY.IFRAME-TRANSPORT.JS annotation]: https://cmlenz.github.io/jquery-iframe-transport/
