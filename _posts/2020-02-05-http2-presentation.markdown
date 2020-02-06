---
layout: post
title: grpc学习笔记（一） HTTP2 简介
author: Sindweller-ch <jinghan.chen@cs2c.com.cn>
tags: [grpc]
---
## HTTP2 简介
基于Google引入的支持多路复用的SPDY（speedy)协议，IETF的HTTP工作组于2015年提出了HTTP2。
1. 引入了**服务器推送**概念，其中服务器预计客户端将需要的资源，并在客户端发出请求之前将其推送，内置的安全机制必须授权服务器预先推送资源。 客户端保留拒绝服务器推送的权限；客户端将推送的资源保存在缓存中，可以跨不同页面重用这些缓存的资源，还可以限制同时多路复用的推送流的数量。
![服务器推送](https://kinsta.com/wp-content/uploads/2016/04/http2-push.png)
![HTTP2授权](https://kinsta.com/wp-content/uploads/2016/04/http2-authorized.png)
2. 引入了**多路复用**概念，通过单个TCP连接完成交错请求和响应，且没有队首阻塞。功能的实现基于“二进制分帧”的特性。
3. 是一种**二进制协议**，二进制成帧层将消息划分为多个帧，这些帧根据其类型（数据或报头）进行分离。 此功能在安全性，压缩和多路复用方面大大提高了效率。
![](https://cheapsslsecurity.com/p/wp-content/uploads/2019/07/binary-framing-layer-500x258.png)
4. **安全性能**方面，HTTP2使用HPACK标头压缩算法，该算法可抵御像CRIME这样的攻击，并使用静态霍夫曼编码。
## HTTP2与HTTP1的区别
1. 线端阻塞：HTTP1限制为每个TCP连接仅处理一个未完成的请求，如果有多个请求，则浏览器需要并行使用多个TCP连接，将导致拥塞，同时垄断网络资源，降低其他用户的网络性能。**HTTP2采用多路复用，所有的通信都在一个TCP连接上完成，实现了真正的并行传输。**
2. 流优先级：http1底层TCP连接的无效使用导致资源优先级低下，随着Web应用程序在复杂性、功能和范围方面的增长而导致性能指数级下降。**http2每个数据流都可以设置优先级和依赖，优先级高的数据流会被服务器优先处理和返回客户端。无需应用不必要的优化。**
![流优先级](https://assets.digitalocean.com/articles/cart_63893/Stream_Priority2.png)
3. 安全：HTTP1.1中使用了摘要身份验证。**http2中改进了[新TLS功能](http://http2.github.io/http2-spec/#TLSUsage)的实现，使得新应用协议具有更好的安全性能设计。**
4. 报头压缩：在HTTP1.x中，头部元数据都是以纯文本的形式发送的，通常会给每个请求增加500~800字节的负荷。**HTTP2默认使用UPACK压缩，既避免了重复报头的传输，又减小了需要传输的大小。高效的压缩算法可以很大的压缩报头，减少发送包的数量从而降低延迟。**
![HTTP2 HPACK 压缩](https://kinsta.com/wp-content/uploads/2016/04/http2-hpack-compression.png)
5. 协议格式： HTTP1采用文本格式。**HTTP2则是二进制协议。使用HTTP2实现的浏览器会将相同的文本命令转换为二进制命令，然后再通过网络进行传输。**
![二进制协议](https://kinsta.com/wp-content/uploads/2016/04/binary-protocols-2-1024x201.png)
[HTTP/1.x vs. HTTP/2 – The Difference Between the Two Protocols Explained](https://cheapsslsecurity.com/p/http2-vs-http1/)提供了一份更为详细的表格来对比HTTP1和HTTP2。
## 总结
HTTP2是二进制协议，支持多路复用，报头压缩，资源优先级和更智能的数据包流管理。可以减少延迟，并加快网页上的内容下载。
### 备注
- 多路复用
1. 所有的http2.0通信都在一个TCP连接上完成，这个连接可以承载任意数量的双向数据流。
2. 每个数据流以消息的形式发送，消息由多帧组成，帧可以乱序发送，接受方根据帧头部的流标识符（stream id）重新组装，归属到各自的请求中。
3. 多路复用可能导致关键请求阻塞。但在HTTP2中，每个数据流都可以设置优先级和依赖，优先级高的数据流会被服务器优先处理和返回客户端，数据流还可以依赖其他的子数据流。
- TLS(Transport Layer Security,安全传输层协议)
位于会话层，主要目标是使SSL更安全，并使协议的规范更精确和完善。TLS 在SSL v3.0 的基础上，主要有以下增强内容：
1. TLS 使用“消息认证代码的密钥散列法”（HMAC）更安全的MAC算法。
2. TLS提供更多的特定和附加警报，还对何时应该发送某些警报进行记录。
3. 增强的伪随机功能，TLS对于安全性的改进。
- 服务器推送
当浏览器请求一个网页时，服务器将会发回HTML，在服务器开始发送JavaScript、图片和CSS前，服务器需要等待浏览器解析HTML和发送所有内嵌资源的请求。服务器推送服务通过“推送”那些它认为客户端将会需要的内容到客户端的缓存中，以此来避免往返的延迟。
### ref
[What's wrong with http1?](https://kinsta.com/learn/what-is-http2/#what_was_wrong_with_http1)
[http/1.x,http/2,https,SSL,TLS](https://www.cnblogs.com/smallJunJun/p/10546975.html)
[new TLS features](http://http2.github.io/http2-spec/#TLSUsage) Section 9.2
[HTTP/1.x vs. HTTP/2 – The Difference Between the Two Protocols Explained](https://cheapsslsecurity.com/p/http2-vs-http1/)
[The first answer](https://stackoverflow.com/questions/28592077/what-is-the-difference-between-http-1-1-and-http-2-0)
[HTTP/1.1 vs HTTP/2: What's the Difference?](https://www.digitalocean.com/community/tutorials/http-1-1-vs-http-2-what-s-the-difference)

