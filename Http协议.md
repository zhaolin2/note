# Http

 **HyperText Transfer Protocol**  通过URI来表示

 **HTTP 协议是以 ASCII 码传输，基于请求与响应模式的、无状态的，建立在 TCP/IP 协议之上的应用层规范** 

# Http/1.0

1.0规定浏览器和服务器只保持短暂的连接，每次请求都要建立tcp连接。

因为只保持短暂的连接，连接无法复用。

# Http/1.1

引入了持久化的连接，在一个TCP连接中可以传送多个HTTP请求，减少了关闭连接的消耗和延迟。

 ![img](https://mmbiz.qpic.cn/mmbiz_png/6fuT3emWI5JoqN8DvPFLUkbVYEWaxoLeWEYNddMncqKC771lysMljAEFQZ43QnrAKK7bjC11ls4veKHrvZsN6Q/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

# SPDY

 **SPDY位于HTTP之下，TCP和SSL之上，这样可以轻松兼容老版本的HTTP协议。** google研究的

- 多路复用
- header压缩。删除或者删除HTTP头
- 服务端推送。提供服务方发起通信，并向客户段推送数据的机制。

在HTTP/2之前，主流浏览器都支持SPDY，HTTP最后以SPDY2为基础开发HTTP/2

# HTTP/2

- 二进制分帧
  - 在应用层和传输层添加了一层 二进制分帧
  - 所有的传输信息都会被分割为更小的消息和帧，并进行二进制消息的编码
- 多路复用
  - 在1.1中，针对同一域名的请求有一定数量的限制，多了会被阻塞
  - HTTP2的TCP一旦建立，后续通过stream进行发送，每个stream的基本组成都是frame(二进制帧)，分解为乱序的帧，然后在另一端组合起来。
  - 采用了HPACK头部压缩算法来对Header进行压缩
  -  用户的浏览器和服务器在建立连接后，服务器主动将一些资源推送给浏览器并缓存起来的机制

 ![img](https://mmbiz.qpic.cn/mmbiz_png/6fuT3emWI5JoqN8DvPFLUkbVYEWaxoLedztM8v2mibWpsZ97ptE393S62XjyWE3MLy1MLyDy19Ek8l1QT5un23w/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

# Patch和Put

Put一般用来进行整体的更新，Put一个完整的对象来进行更新

Patch一般用来进行部分字段的更新，比如我只需要更新其中两个字段，那么可以使用Patch来进行局部的更新

## POST和PUT

- POST一般用来创建或者更新资源
- Put一般只拿来更新资源

# 跨域

## JSONP

jsonp是通过动态的插入一个script的标签，由于浏览器对script的资源引用没有限制，并且还会立刻执行。

同时前后端还会约定一个callback来约定处理返回数据的函数名称。

- 无法发送**POST**请求
- 确定失败与否并不容易，大部分框架结合超时来进行判定

## 代理

使用代理服务器来进行资源的请求

这种方式首先将请求发送给后台服务器，通过服务器来发送请求，然后将请求的结果传递给前端。 

在**https**的请求下，只能使用**代理**来进行请求

## CORS

使用XMLHttpRequest发送请求的时候，发现请求不符合同源策略(域名和端口号相同)，会添加请求头:Origin,后台在接受处理之后，需要在返回的结果中加入 Access-Control-Allow-Origin ,浏览器会判断对应的头中是否包含Origin的值，有的话，我们就可以拿到响应的值，不包含的话，无法拿到。

```java
 if (req.headers.origin) {
 
             res.writeHead(200, {
                 "Content-Type": "text/html; charset=UTF-8",
                 "Access-Control-Allow-Origin":'http://localhost'/*,
                 'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
                 'Access-Control-Allow-Headers': 'X-Requested-With, Content-Type'*/
             });
             res.write('cors');
            res.end();
}

```

### OPTIONS

一般用来检测服务器所支持的请求方法

 OPTIONS请求头部中会包含以下头部：

Origin、

Access-Control-Request-Method、

Access-Control-Request-Headers，

发送这个请求后，服务器可以设置如下头部与浏览器沟通来判断是否允许这个请求 

# URL

URL=protocol+domain+port+URI+Paramters

 RFC 1738(Uniform Resource Locators (URL))规定 URL 只能包含英文字母、阿拉伯数字和某些标点符号。这要求在 URL 中使用非英文字符的话必须编码后使用，不过 RFC 1738 并没有规定如何进行编码，而是交给应用程序（浏览器）自己决定。 

 用户从浏览器端发起一个 HTTP 请求，需要存在编码的地方是 **URL**、**Cookie**、**Parameter**。服务器端接受到 HTTP 请求后要解析 HTTP 协议，其中 URI、Cookie 和 POST 表单参数需要解码，服务器端可能还需要读取数据库中的数据，本地或网络中其它地方的文本文件，这些数据都可能存在编码问题，当 Servlet 处理完所有请求的数据后，需要将这些数据再编码通过 Socket 发送到用户请求的浏览器里，再经过浏览器解码成为文本。 