---
title: 综合-HTTP协议入门
date: 2018-04-17 15:10:58
tags: ["基础知识",http]
category: "综合"
---
## HTTP 协议入门

HTTP 协议是互联网的基础协议，也是网页开发的必备知识，最新版本 HTTP/2 更是让它成为技术热点。
本文介绍 HTTP 协议的历史演变和设计思路。

HTTP协议是`Hyper Text Transfer Protocol`（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

HTTP是一个基于`TCP/IP`通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。

![](http://ou3l58em5.bkt.clouddn.com/http1.jpg)

### 一、HTTP/0.9

HTTP 是基于`TCP/IP`协议的应用层协议。它不涉及数据包（packet）传输，主要规定了客户端和服务器之间的通信格式，默认使用`80`端口。

最早版本是1991年发布的0.9版。该版本极其简单，只有一个命令GET。

```bath
GET /index.html
```
上面命令表示，`TCP` 连接（connection）建立后，客户端向服务器请求（request）网页index.html。
协议规定，服务器只能回应HTML格式的字符串，不能回应别的格式。

```html
<html>
  <body>Hello World</body>
</html>
```

服务器发送完毕，就关闭TCP连接。

### 二、HTTP/1.0

#### 2.1 简介
1996年5月，`HTTP/1.0` 版本发布，内容大大增加。

首先，任何格式的内容都可以发送。这使得互联网不仅可以传输文字，还能传输图像、视频、二进制文件。这为互联网的大发展奠定了基础。

其次，除了`GET`命令，还引入了`POST`命令和`HEAD`命令，丰富了浏览器与服务器的互动手段。

再次，HTTP请求和回应的格式也变了。除了数据部分，每次通信都必须包括头信息（`HTTP header`），用来描述一些元数据。

其他的新增功能还包括状态码（`status code`）、多字符集支持、多部分发送（`multi-part type`）、权限（`authorization`）、缓存（`cache`）、内容编码（`content encoding`）等。

#### 2.2 请求以及相应格式

这个将在`HTTP/1.1`部分讲解

#### 2.3 缺点
`HTTP/1.0` 版的主要缺点是，每个`TCP`连接只能发送一个请求。发送数据完毕，连接就关闭，如果还要请求其他资源，就必须再新建一个连接。

`TCP`连接的新建成本很高，因为需要客户端和服务器三次握手，并且开始时发送速率较慢（`slow start`）。所以，`HTTP 1.0`版本的性能比较差。随着网页加载的外部资源越来越多，这个问题就愈发突出了。

为了解决这个问题，有些浏览器在请求时，用了一个非标准的`Connection`字段。

```bath
Connection: keep-alive
```
这个字段要求服务器不要关闭`TCP`连接，以便其他请求复用。服务器同样回应这个字段。

```bash
Connection: keep-alive
```
一个可以复用的`TCP`连接就建立了，直到客户端或服务器主动关闭连接。但是，这不是标准字段，不同实现的行为可能不一致，因此不是根本的解决办法。

### 三、HTTP/1.1

1997年1月，HTTP/1.1 版本发布，只比 1.0 版本晚了半年。它进一步完善了 HTTP 协议，一直用到了20年后的今天，直到现在还是最流行的版本。

#### 3.1 请求格式

`GET`的例子
```
GET /562f25980001b1b106000338.jpg HTTP/1.0
Host:    img.mukewang.com
User-Agent:    Mozilla/5.0 (Macintosh; Intel …) Gecko/20100101 Firefox/55.0
Accept:    */*
Accept-Language:   zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding:   gzip, deflate
Referer:   http://fragment.firefoxchina.c…/baidu_main_page_250x250.html
Cookie:    BAIDUID=F395366A0E9261C1F376BA…BAE2E839289B; PSTM=1494254264
Connection:    keep-alive
```
`POST`的例子

```
POST / HTTP1.1
Host:www.wrox.com
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive

name=Professional%20Ajax&publisher=Wiley
```

第一部分是请求命令，必须在尾部添加协议版本（HTTP/1.1）。第二部分后面就是多行头信息，描述客户端的情况。第三部分：空行，即使第四部分没有请求的数据体，空行也要有。第四部分：请求数据。

#### 3.2 回应格式

服务器的回应如下。
```
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
```
回应的格式是"头信息 + 一个空行（\r\n） + 数据"。其中，第一行是"协议版本 + 状态码（status code） + 状态描述"。
#### 3.3 Content-Type 字段

关于字符的编码，1.0版规定，头信息必须是 ASCII 码，后面的数据可以是任何格式。因此，服务器回应的时候，必须告诉客户端，数据是什么格式，这就是Content-Type字段的作用。
下面是一些常见的Content-Type字段的值。
```
text/plain
text/html
text/css
image/jpeg
image/png
image/svg+xml
audio/mp4
video/mp4
application/javascript
application/pdf
application/zip
application/atom+xml
```
这些数据类型总称为`MIME type`，每个值包括一级类型和二级类型，之间用斜杠分隔。
除了预定义的类型，厂商也可以自定义类型。[MIME type 说明](/note/specification/mime)

application/vnd.debian.binary-package
上面的类型表明，发送的是Debian系统的二进制数据包。

MIME type还可以在尾部使用分号，添加参数。
```
Content-Type: text/html; charset=utf-8
```

上面的类型表明，发送的是网页，而且编码是UTF-8。

客户端请求的时候，可以使用Accept字段声明自己可以接受哪些数据格式。
```
Accept: */*
```
上面代码中，客户端声明自己可以接受任何格式的数据。

#### 3.4 Content-Encoding 字段
由于发送的数据可以是任何格式，因此可以把数据压缩后再发送。Content-Encoding字段说明数据的压缩方法。
```
Content-Encoding: gzip
Content-Encoding: compress
Content-Encoding: deflate
```
客户端在请求时，用Accept-Encoding字段说明自己可以接受哪些压缩方法。
```
Accept-Encoding: gzip, deflate
```

#### 3.5 1.0与1.1差异

##### 3.5.1 版的最大变化，就是引入了持久连接（persistent connection），即`TCP`连接默认不关闭，可以被多个请求复用，不用声明`Connection: keep-alive`。
客户端和服务器发现对方一段时间没有活动，就可以主动关闭连接。不过，规范的做法是，客户端在最后一个请求时，发送`Connection: close`，明确要求服务器关闭`TCP`连接。
```
Connection: close
```
目前，对于同一个域名，大多数浏览器允许同时建立6个持久连接。

##### 3.5.2 管道机制
1.1 版还引入了管道机制（pipelining），即在同一个`TCP`连接里面，客户端可以同时发送多个请求。这样就进一步改进了HTTP协议的效率。

举例来说，客户端需要请求两个资源。以前的做法是，在同一个`TCP`连接里面，先发送A请求，然后等待服务器做出回应，收到后再发出B请求。管道机制则是允许浏览器同时发出A请求和B请求，但是服务器还是按照顺序，先回应A请求，完成后再回应B请求。

##### 3.5.3 `Content-Length` 字段
一个`TCP`连接现在可以传送多个回应，势必就要有一种机制，区分数据包是属于哪一个回应的。这就是`Content-length`字段的作用，声明本次回应的数据长度。
```
Content-Length: 3495
```
上面代码告诉浏览器，本次回应的长度是3495个字节，后面的字节就属于下一个回应了。

在1.0版中，`Content-Length`字段不是必需的，因为浏览器发现服务器关闭了`TCP`连接，就表明收到的数据包已经全了。

##### 3.5.4 分块传输编码

使用`Content-Length`字段的前提条件是，服务器发送回应之前，必须知道回应的数据长度。

对于一些很耗时的动态操作来说，这意味着，服务器要等到所有操作完成，才能发送数据，显然这样的效率不高。更好的处理方法是，产生一块数据，就发送一块，采用"流模式"（stream）取代"缓存模式"（buffer）。

因此，1.1版规定可以不使用`Content-Length`字段，而使用"分块传输编码"（chunked transfer encoding）。只要请求或回应的头信息有Transfer-Encoding字段，就表明回应将由数量未定的数据块组成。
```
Transfer-Encoding: chunked
```
每个非空的数据块之前，会有一个16进制的数值，表示这个块的长度。最后是一个大小为0的块，就表示本次回应的数据发送完了。下面是一个例子。

```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1C
and this is the second one

3
con

8
sequence

0
```
##### 3.5.5 其他功能
1.1版还新增了许多动词方法：`PUT`、`PATCH`、`HEAD`、`OPTIONS`、`DELETE`。

另外，客户端请求的头信息新增了`Host`字段，用来指定服务器的域名。
```
Host: www.example.com
```
有了Host字段，就可以将请求发往同一台服务器上的不同网站，为虚拟主机的兴起打下了基础。

#### 3.6 HTTP之状态码

状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别:
```
1xx：指示信息--表示请求已接收，继续处理

2xx：成功--表示请求已被成功接收、理解、接受

3xx：重定向--要完成请求必须进行更进一步的操作

4xx：客户端错误--请求有语法错误或请求无法实现

5xx：服务器端错误--服务器未能实现合法的请求
```
常见状态码：
```js
100 Continue                  // 继续。客户端应继续其请求
200 OK                        // 客户端请求成功
301 Moved Permanently         // 资源（网页等）被永久转移到其它URL
304 Not Modified              // 未修改，客户端使用缓存
400 Bad Request               // 客户端请求有语法错误，不能被服务器所理解
401 Unauthorized              // 请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用
403 Forbidden                 // 服务器收到请求，但是拒绝提供服务
404 Not Found                 // 请求资源不存在，eg：输入了错误的URL
500 Internal Server Error     // 服务器发生不可预期的错误
503 Server Unavailable        // 服务器当前不能处理客户端的请求，一段时间后可能恢复正常
```
[更多状态请查看](http://www.runoob.com/http/http-status-codes.html)

#### 3.7 HTTP请求方法
根据HTTP标准，HTTP请求可以使用多种请求方法。

HTTP1.0定义了三种请求方法： `GET`, `POST`和`HEAD`方法。

HTTP1.1新增了五种请求方法：`OPTIONS`, `PUT`, `DELETE`, `TRACE`和`CONNECT`方法。

| 方法        | 说明           |
| :------------- |:-------------|
| GET      | 请求指定的页面信息，并返回实体主体。 |
| HEAD      | 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头。      |
| POST | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。      |
| PUT | 从客户端向服务器传送的数据取代指定的文档的内容。      |
| DELETE | 请求服务器删除指定的页面。      |
| CONNETCT | HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。      |
| OPTIOINS | 允许客户端查看服务器的性能。     |
| TRACE | 回显服务器收到的请求，主要用于测试或诊断。      |


#### 3.8 HTTP/1.1 缺点
虽然1.1版允许复用`TCP`连接，但是同一个`TCP`连接里面，所有的数据通信是按次序进行的。服务器只有处理完一个回应，才会进行下一个回应。要是前面的回应特别慢，后面就会有许多请求排队等着。这称为"队头堵塞"（Head-of-line blocking）。

为了避免这个问题，只有两种方法：一是减少请求数，二是同时多开持久连接。这导致了很多的网页优化技巧，比如合并脚本和样式表、将图片嵌入CSS代码、域名分片（domain sharding）等等。如果HTTP协议设计得更好一些，这些额外的工作是可以避免的。

### 四、SPDY 协议
2009年，谷歌公开了自行研发的 SPDY 协议，主要解决 HTTP/1.1 效率不高的问题。

这个协议在Chrome浏览器上证明可行以后，就被当作 HTTP/2 的基础，主要特性都在 HTTP/2 之中得到继承。
### 五、HTTP/2
2015年，HTTP/2 发布。它不叫 HTTP/2.0，是因为标准委员会不打算再发布子版本了，下一个新版本将是 HTTP/3。

#### 5.1 二进制协议
HTTP/1.1 版的头信息肯定是文本（ASCII编码），数据体可以是文本，也可以是二进制。HTTP/2 则是一个彻底的二进制协议，头信息和数据体都是二进制，并且统称为"帧"（frame）：头信息帧和数据帧。

二进制协议的一个好处是，可以定义额外的帧。HTTP/2 定义了近十种帧，为将来的高级应用打好了基础。如果使用文本实现这种功能，解析数据将会变得非常麻烦，二进制解析则方便得多。

#### 5.2 多工
HTTP/2 复用`TCP`连接，在一个连接里，客户端和浏览器都可以同时发送多个请求或回应，而且不用按照顺序一一对应，这样就避免了"队头堵塞"。

举例来说，在一个`TCP`连接里面，服务器同时收到了A请求和B请求，于是先回应A请求，结果发现处理过程非常耗时，于是就发送A请求已经处理好的部分， 接着回应B请求，完成后，再发送A请求剩下的部分。

这样双向的、实时的通信，就叫做多工（Multiplexing）。

#### 5.3 数据流

因为 HTTP/2 的数据包是不按顺序发送的，同一个连接里面连续的数据包，可能属于不同的回应。因此，必须要对数据包做标记，指出它属于哪个回应。

HTTP/2 将每个请求或回应的所有数据包，称为一个数据流（stream）。每个数据流都有一个独一无二的编号。数据包发送的时候，都必须标记数据流ID，用来区分它属于哪个数据流。另外还规定，客户端发出的数据流，ID一律为奇数，服务器发出的，ID为偶数。

数据流发送到一半的时候，客户端和服务器都可以发送信号（RST_STREAM帧），取消这个数据流。1.1版取消数据流的唯一方法，就是关闭`TCP`连接。这就是说，HTTP/2 可以取消某一次请求，同时保证`TCP`连接还打开着，可以被其他请求使用。

客户端还可以指定数据流的优先级。优先级越高，服务器就会越早回应。

#### 5.4 头信息压缩

HTTP 协议不带有状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的，比如Cookie和User Agent，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。

HTTP/2 对这一点做了优化，引入了头信息压缩机制（header compression）。一方面，头信息使用gzip或compress压缩后再发送；另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就提高速度了。

#### 5.5 服务器推送

HTTP/2 允许服务器未经请求，主动向客户端发送资源，这叫做服务器推送（server push）。

常见场景是客户端请求一个网页，这个网页里面包含很多静态资源。正常情况下，客户端必须收到网页后，解析HTML源码，发现有静态资源，再发出静态资源请求。其实，服务器可以预期到客户端请求网页后，很可能会再请求静态资源，所以就主动把这些静态资源随着网页一起发给客户端了。