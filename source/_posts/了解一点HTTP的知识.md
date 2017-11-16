title: 了解一点HTTP的知识
author: liompei
tags:
  - 网络
categories:
  - 基础知识
date: 2017-11-16 12:53:00
---
超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。

HTTP 是应用层协议，当你上网浏览网页的时候，浏览器和 web 服务器之间就会通过 HTTP 在 Internet 上进行数据的发送和接收。

HTTP 是一个基于请求/响应模式的、无状态的协议。即我们通常所说的 Request/Response

### Http协议的主要特点
- 默认端口:80


1. 支持客户端/服务器模式
2. 简单快速: 客户向服务器请求服务时,只需传送请求方式和路径
3. 灵活: 允许传输任意类型的数据对象.由Content-Type加以标记
4. 无连接: 每次相应一个请求,响应完成以后就断开连接
5. 无状态: 服务器不保存浏览器的任何信息,每次提交的请求之间没有关联

### Http协议版本

- 0.9 :已过时。只接受 GET 一种请求方法，没有在通讯中指定版本号，且不支持请求头。由于该版本不支持 POST 方法，所以客户端无法向服务器传递太多信息。
- HTTP/1.0 :这是第一个在通讯中指定版本号的HTTP 协议版本，至今仍被广泛采用，特别是在代理服务器中。
- HTTP/1.1 :持久连接被默认采用，并能很好地配合代理服务器工作。还支持以管道方式同时发送多个请求，以便降低线路负载，提高传输速度。
- HTTP 2.0 :大幅度的提升了 web 性能。在与 HTTP/1.1 完全语义兼容的基础上，进一步减少了网络延迟。

#### HTTP1.1和HTTP1.0的区别
1. 缓存处理
2. 带宽优化及网络连接的使用
3. 错误通知的管理
4. 消息在网络中的发送
5. 互联网地址的维护
6. 安全性及完整性

### URL
URL（Uniform Resource Locator）是统一资源定位符的简称，有时候也被俗称为网页地址（网址），如同是网络上的门牌，是因特网上标准的资源的地址

#### 基本组成
通用的格式：schema://host[:port#]/path/…/[?query-string][#anchor]<br>

-----
| 名称 | 功能 |
| :--- | :--- |
| schema | 访问服务器以获取资源时要使用哪种协议,比如http,https,ftp等 |
| host | 服务器的IP地址或域名,如www.baidu.com |
| port | HTTP服务器的默认端口是80,可以省略,如果使用了别的端口,必须指明 |
| path | 访问资源的路径 |
| query-string | 发给http服务器的数据 |
| anchor | 锚 |

例如:
![upload successful](\images\pasted-4.png)

### HTTP请求
http的请求分为三个部分: 请求行,请求头,请求体

![upload successful](\images\pasted-5.png)

#### 1.请求行
请求行（Request line）分为三个部分：请求方法、请求地址和协议版本

##### 请求方法
HTTP/1.1 协议中共定义了八种方法（也叫“动作”）来以不同的方式操作指定的资源

|方法名|功能|
|:--|:--|
|GET|向指定的资源发出“显示”请求，使用 GET 方法应该只用在读取数据上，而不应该用于产生“副作用”的操作中|
|POST|指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求文本中。这个请求可能会创建新的资源或者修改现有资源，或两者皆有。|
|PUT|向指定资源位置上传其最新内容|
|DELETE|请求服务器删除 Request-URI 所标识的资源|
|OPTIONS|使服务器传回该资源所支持的所有HTTP请求方法。用`*`来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作|
|HEAD|与 GET 方法一样，都是向服务器发出指定资源的请求，只不过服务器将不传回资源的本文部分，它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中关于该资源的信息（原信息或称元数据）|
|TRACE|	显示服务器收到的请求，主要用于测试或诊断|
|CONNECT|HTTP/1.1 中预留给能够将连接改为通道方式的代理服务器。通常用于 SSL 加密服务器的链接（经由非加密的 HTTP 代理服务器）|

- 其中，最常见的是 GET 和 POST 方法，如果是 RESful 接口的话一般会用到 PUT、DELETE、GET、POST（分别对应增删查改）

##### POST和GET的区别
|Post|Get|
|--|--|
|Post一般用于更新或者添加资源信息|Get一般用于查询操作，而且应该是安全和幂等的|
|Post更加安全|Get会把请求的信息放到URL的后面|
|Post传输量一般无大小限制|Get不能大于2KB|
|Post执行效率低|Get执行效率略高|

#### 2.请求头Header
请求头可用于传递一些附加信息，格式为：键: 值，注意冒号后面有一个空格

请求和响应常见通用的 Header

|名称|作用|
|:--|:--|
|Content-Type|请求体/响应体的类型，如：text/plain、application/json|
|Accept|说明接收的类型，可以多个值，用,(英文逗号)分开|
|Content-length|请求体/响应体的长度，单位字节|
|Content-Encoding|请求体/响应体的编码格式，如 gzip、deflate|
|Accept-Encoding|告知对方我方接受的 Content-Encoding|
|ETag|给当前资源的标识，和Last-Modified、If-None-Match、If-Modified-Since配合，用于缓存控制|
|Cache-Control|取值一般为no-cache、max-age=xx，xx为整数，表示资源缓存有效期（秒）|

**常见的请求Request Header**

|名称|作用|
|--|--|
|Authorization|用于设置身份认证信息|
|User-Agent|用户标识，如：OS 和浏览器的类型和版本|
|If-Modified-Since|值为上一次服务器返回的`Last-Modified`值，用于确定某个资源是否被更改过，没有更改过就从缓存中读取|
|If-None-Match|值为上一次服务器返回的 ETag 值，一般会和`If-Modified-Since`|
|Cookie|已有的Cookie|
|Referer|标识请求引用自哪个地址，比如你从页面 A 跳转到页面 B 时，值为页面 A 的地址|
|Host|请求的主机和端口号|

**常见的响应头Response Header**

|名称|作用|
|--|--|
|Date|服务器的日期|
|Last-Modified|该资源最后被修改的时间|
|Transfer-Encoding|取值一般为 chunked，出现在 Content-Length 不能确定的情况下，表示服务器不知道响应板体的数据大小，一般同时出现`Content-Encoding`响应头|
|Set-Cookie|设置 Cookie|
|Location|重定向到另一个 URL，如输入浏览器就输入 baidu.com 回车，会自动跳转到www.baidu.com 就是通过这个响应头控制的|
|Server|后台服务器|

### 为什么POST效率低，Get效率高
- Get将参数拼成URL,放到header消息头里传递
- Post直接以键值对的形式放到消息体中传递。
- 但两者的效率差距很小很小

### Https
- 端口号是443
- 是由SSL+Http协议构建的可进行加密传输、身份认证的网络协议。










