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