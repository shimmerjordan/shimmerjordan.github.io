---
title: 浏览器输入URL访问网页的执行流程
date: 2022-07-08 05:38:12
tags: 
    - Web
categories: web
id: url-visit
---

# 流程概览

对于这一个过程应该有一个大概的骨架，然后再是回忆里面的具体细节。总体流程如下，每一部分细节后面补充：

- 查找浏览器缓存：如果查找到缓存中有我们URL对应的文件，则判断是否命中强缓存，如果命中直接读取使用即可，如果强缓存没有命中，判断协商缓存是否命中，但协商缓存不论是否命中都会发送请求，所以都会走下面的步骤

- DNS域名解析：将输入的URL解析成对应的IP地址

- 生成HTTP请求报文：请求报文包括起始行，首部，主体

- TCP连接：客户端与服务端进行TCP三次握手，建立连接

- 发送HTTP请求：握手成功后，客户端向服务端发送http请求，请求数据

- 服务器收到请求并返回数据：客户端根据返回的结果进行渲染展示，同时判断是否需要将文件存入缓存

- TCP断开连接：客户端与服务端进行TCP四次挥手，断开连接

<!-- more -->

# 一、查找浏览器缓存：

如果是第一次请求肯定在缓存找不到文件啦（先在memory cache也就是内存中找，再在disk cache也就是磁盘中找），那之后的请求为什么能找到呢？
因为第一次请求时如果服务器端在对应的**请求接口中设置了响应头里**的`Cache-Control:max-age`或者`Expires`，`Last-Modified`（最后修改时间） 或者`etag`（其实就是一个比文件最后修改时间判断是否修改更精确的一个标识，通常会用文件的md5值），浏览器第一次请求到之后就会将文件及这些信息缓存下来，再次请求的时候就会先判断`max-age`有没有过期（没有`max-age`就找`Expires`，两者同时存在前者优先级更高），如果没有过期，则强缓存命中，返回200（from disk cache或者from memory cache）;

如果已经过期，则会发起一个请求，请求头中带上`If-None-Match`（值就是上一次服务器返回的`etag`）或者`If-Modified-Since`（值就是上一次服务器返回的`Last-Modified` ），去比对服务器中要请求的文件的`etag`或者`last-modified`（注意如果两者同时存在，前者的优先级会更高），判断文件到底有没有修改，如果没有修改，则使用本地缓存的就好了，返回304（from disk cache或者from memory cache），如果修改过了，再从服务器中重新请求，并修改`etag`或者`last-modified`。返回200

## 看两张图，走一遍流程~

### 第一次请求某个URL资源文件

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/URL-visit/1.png" width="70%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">第一次请求URL</div>
</center>

注意这里的`Expires`是http1.0的属性，已经被http1.1的`max-age`所替代了，只有没有设置`max-age`的时候才回去找`Expires`

### 第一次请求某个URL资源文件

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/URL-visit/2.png" width="70%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">再次请求URL</div>
</center>

补充：上文说了缓存会存在两个地方，memory cache或者disk cache，一般来说需要经常读取的文件，比如图片，JS会存在memory cache，不会经常读取的比如css文件会存在disk cache中

# 二、DNS域名解析

DNS虽然也是应用层协议，事实上他是为其他应用层协议工作的，包括不限于HTTP和SMTP以及FTP，用于将用户提供的主机名解析为ip地址

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/URL-visit/3.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">DNS解析流程</div>
</center>

- 查找主机中的浏览器中DNS缓存，缓存中维护了一张域名和IP地址的对应表  

- 查找主机中的操作系统中的DNS缓存，就存在C:\Windows\System32\drivers\etc目录下的hosts文件

- 查询本地域名服务器，注意主机和本地域名服务器为递归查询，也就是主机需要的Ip地址**一定要从本地域名服务器**查询获得，不论他是自身查到的还是继续向上从哪一级获取的

- 如果本地域名服务器没有查询到：
  
  1. 向根域名服务器发起迭代查询，一般DNS服务器的缓存中会有.com域名服务器中的域名，所以到顶级服务器的匹配过程不是那么必要了
  
  2. 根域名服务器会返回他一个权限域名服务器的地址
  
  3. 然后再去向权限域名服务器迭代查询，得到最终的IP地址

- 本地域名服务器得到IP地址后返回给主机的操作系统，操作系统缓存后，交给浏览器，浏览器同时也存在维护`域名-IP地址`的表中

## 例如`www.wikipedia.org`：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/URL-visit/6.png" width="80%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">DNS解析流程</div>
</center>

这样的整个域名看上去只是对应一个单独的IP地址，**不过事实上后面可能对应着多个ip地址**（**也是学习到了，一个ip地址可以对应多个域名[听说一个IP可以绑定多个域名，那么…? - 互联网](http://www.zhihu.com/question/29390934)，一个域名也可以对应多个ip地址[负载均衡实现，一个域名对应多个IP地址](https://blog.csdn.net/ywcpig/article/details/52525689)，cry！**）还好，有几种方法可以消除这个瓶颈：

- [循环 DNS](https://blog.csdn.net/ywcpig/article/details/52525689) 是DNS查找时返回多个IP时的解决方案。举例来说，
  
  ```text
  Facebook.com
  ```
  
  实际上就对应了四个IP地址。

- [负载平衡器](https://blog.csdn.net/ywcpig/article/details/52525689) 是以一个特定IP地址进行侦听并将网络请求转发到集群服务器上的硬件设备。 一些大型的站点一般都会使用这种昂贵的高性能负载平衡器。

- **地理 DNS** 根据用户所处的地理位置，通过把域名映射到多个不同的IP地址提高可扩展性。这样不同的服务器不能够更新同步状态，但映射静态内容的话非常好。

- [Anycast](https://blog.csdn.net/ywcpig/article/details/52525689)是一个IP地址映射多个物理主机的路由技术。 美中不足，Anycast与TCP协议适应的不是很好，所以很少应用在那些方案中

# 三、生成HTTP请求报文

因为像Facebook主页这样的动态页面，打开后在浏览器缓存中很快甚至马上就会过期，毫无疑问他们不能从中读取。

所以，浏览器将把以下请求发送到Facebook所在的服务器：

```yaml
GET http://facebook.com/ HTTP/1.1
 Accept: application/x-ms-application, image/jpeg, application/xaml+xml, [...]
 User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; [...]
 Accept-Encoding: gzip, deflate
 Connection: Keep-Alive
 Host: facebook.com
 Cookie: datr=1265876274-[...]; locale=en_US; lsd=WW[...]; c_user=2101[...]
```

GET 这个请求定义了要读取的**URL**：`http://facebook.com/`

以及浏览器自身定义 (**User-Agent** 头)， 和它希望接受什么类型的相应 (**Accept** and **Accept-Encoding** 头)

**Connection**头要求服务器为了后边的请求不要关闭TCP连接。

## Tip：

**URL**：`http://facebook.com/`中最后的斜杠是至关重要的。这种情况下，浏览器能安全的添加斜杠。而像`http://example.com/folderOrFile`这样的地址，因为浏览器不清楚`folderOrFile`到底是文件夹还是文件，所以不能自动添加斜杠。这时，浏览器就不加斜杠直接访问地址，服务器会响应一个重定向，结果造成一次不必要的握手。

# 四、TCP连接

进行TCP三次握手连接，这里附图复习一下三次握手四次挥手

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/URL-visit/4.jpg" width="80%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">三次握手</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/URL-visit/5.jpg" width="80%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">四次挥手</div>
</center>

# Option - 服务的永久重定向响应

例如 Facebook 服务器会发回浏览器如下响应：

```yaml
HTTP/1.1 301 Moved Permanently
 Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
 pre-check=0
 Expires: Sat, 01 Jan 2000 00:00:00 GMT
 Location: http://www.facebook.com/
 P3P: CP="DSP LAW"
 Pragma: no-cache
 Set-Cookie: made_write_conn=deleted; expires=Thu, 12-Feb-2009 05:09:50 GMT;
 path=/; domain=.facebook.com; httponly
 Content-Type: text/html; charset=utf-8
 X-Cnection: close
 Date: Fri, 12 Feb 2010 05:09:51 GMT
 Content-Length: 0
```

服务器给浏览器响应一个301永久重定向响应，这样浏览器就会访问`http://www.facebook.com/`而非`http://facebook.com/`

为什么服务器一定要重定向而不是直接发会用户想看的网页内容呢？这个问题有好多有意思的答案：

1. 其中一个原因跟**搜索引擎排名**有关。你看，如果一个页面有两个地址，就像`http://www.igoro.com/`和`http://igoro.com/`，搜索引擎会认为它们是两个网站，结果造成每一个的搜索链接都减少从而降低排名。而搜索引擎知道301永久重定向是什么意思，这样就会把访问带www的和不带www的地址归到同一个网站排名下。

2. 还有一个是用不同的地址会造成**缓存友好性**变差。当一个页面有好几个名字时，它可能会在缓存里出现好几次。

# 服务器收到请求并返回数据

服务器在处理完请求之后生成并返回的响应：

```yaml
HTTP/1.1 200 OK
 Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
 pre-check=0
 Expires: Sat, 01 Jan 2000 00:00:00 GMT
 P3P: CP="DSP LAW"
 Pragma: no-cache
 Content-Encoding: gzip
 Content-Type: text/html; charset=utf-8
 X-Cnection: close
 Transfer-Encoding: chunked
 Date: Fri, 12 Feb 2010 09:05:55 GMT

 2b3Tn@[...]
```

整个响应大小为35kB，其中大部分在整理后以blob类型传输。

**内容编码**头告诉浏览器整个响应体用gzip算法进行压缩。解压blob块后，你可以看到如下期望的HTML：

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"    
 "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
 <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en"
 lang="en" id="facebook">
 <head>
 <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
 <meta http-equiv="Content-language" content="en" />
 ...
```

关于压缩，头信息说明了是否缓存这个页面，如果缓存的话如何去做，有什么cookies要去设置（前面这个响应里没有这点）和隐私信息等等。

请注意报头中把**Content-type**设置为`text/html`。报头让浏览器将该响应内容以HTML形式呈现，而不是以文件形式下载它。浏览器会根据报头信息决定如何解释该响应，不过同时也会考虑像URL扩展内容等其他因素。

## 浏览器发送获取嵌入HTML的对象

在浏览器显示HTML时，它会注意到需要获取其他地址内容的标签。这时，浏览器会发送一个获取请求来重新获得这些文件。

下面是几个我们访问[http://facebook.com](https://blog.csdn.net/ywcpig/article/details/52525689)时需要重获取的几个URL：

```yaml
# pic
http://static.ak.fbcdn.net/rsrc.php/z12E0/hash/8q2anwu7.gif
http://static.ak.fbcdn.net/rsrc.php/zBS5C/hash/7hwy7at6.gif
# css
http://static.ak.fbcdn.net/rsrc.php/z448Z/hash/2plh8s4n.css
http://static.ak.fbcdn.net/rsrc.php/zANE1/hash/cvtutcee.css
# j
http://static.ak.fbcdn.net/rsrc.php/zEMOA/hash/c8yzb6ub.js
http://static.ak.fbcdn.net/rsrc.php/z6R9L/hash/cq2lgbs8.jss
```

这些地址都要经历一个和HTML读取类似的过程。所以浏览器会在DNS中查找这些域名，发送请求，重定向等。

但不像动态页面那样，静态文件会允许浏览器对其进行缓存。有的文件可能会不需要与服务器通讯，而从缓存中直接读取。服务器的响应中包含了静态文件保存的期限信息，所以浏览器知道要把它们缓存多长时间。还有，每个响应都可能包含像版本号一样工作的ETag头（被请求变量的实体值），如果浏览器观察到文件的版本 `ETag` 信息已经存在，就马上停止这个文件的传输。

试着猜猜看`http://fbcdn.net`在地址中代表什么？聪明的答案是”Facebook内容分发网络”。Facebook利用**内容分发网络**（CDN）分发像图片，CSS表和JavaScript文件这些静态文件。所以，这些文件会在全球很多CDN的数据中心中留下备份。

静态内容往往代表站点的带宽大小，也能通过CDN轻松的复制。通常网站会使用第三方的CDN。例如，Facebook的静态文件由最大的CDN提供商Akamai来托管。

举例来讲，当你试着ping `http://static.ak.fbcdn.net`的时候，可能会从某个`http://akamai.net`服务器上获得响应。有意思的是，当你同样再ping一次的时候，响应的服务器可能就不一样，这说明幕后的负载平衡开始起作用了。

# Option - 览器发送异步（AJAX）请求

在Web 2.0伟大精神的指引下，页面显示完成后客户端仍与服务器端保持着联系。

以 Facebook 聊天功能为例，它会持续与服务器保持联系来及时更新你那些亮亮灰灰的好友状态。为了更新这些头像亮着的好友状态，在浏览器中执行的 JavaScript 代码会给服务器发送异步请求。这个异步请求发送给特定的地址，它是一个按照程式构造的获取或发送请求。还是在Facebook这个例 子中，客户端发送给`http://www.facebook.com/ajax/chat/buddy_list.php`一个发布请求来获取你好友里哪个在线的状态信息。

提起这个模式，就必须要讲讲”AJAX”– “异步JavaScript 和 XML”，虽然服务器为什么用XML格式来进行响应也没有个一清二白的原因。再举个例子吧，对于异步请求，Facebook会返回一些JavaScript的代码片段。

除了其他，fiddler 这个工具能够让你看到浏览器发送的异步请求。事实上，你不仅可以被动的做为这些请求的看客，还能主动出击修改和重新发送它们。AJAX请求这么容易被蒙，可着实让那些计分的在线游戏开发者们郁闷的了。（当然，可别那样骗人家~）

Facebook聊天功能提供了关于AJAX一个有意思的问题案例：把数据从服务器端推送到客户端。因为HTTP是一个 请求-响应 协议，所以聊天服务器不能把新消息发给客户。取而代之的是客户端不得不隔几秒就轮询下服务器端看自己有没有新消息。

这些情况发生时长轮询是个减轻服务器负载挺有趣的技术。如果当被轮询时服务器没有新消息，它就不理这个客户端。而当尚未超时的情况下收到了该客户的新消息，服务器就会找到未完成的请求，把新消息做为响应返回给客户端。
