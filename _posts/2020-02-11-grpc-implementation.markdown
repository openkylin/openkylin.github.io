---
layout: post
title:  gRPC调研： 基于HTTP2的gRPC框架实现
author: Sindweller-ch <jinghan.chen@cs2c.com.cn>
tags: [grpc]
---
## gRPC简介
gRPC是由Google主导开发、可以在任何环境中运行的开源高性能RPC（远程过程调用）框架，基于HTTP2和Protobuf实现。gRPC是跨语言的RPC框架，采用了IDL来描述数据类型和接口，使用编译器编译出特定语言的代码从而实现跨语言的RPC。
- 简单的服务定义  
使用Protobuf，功能强大的二进制序列化工具集和语言定义服务  
- 快速启动并扩展  
只需一行即可安装运行和开发环境，并通过该框架每秒可扩展至数百万个RPC  
- 跨语言和平台工作  
自动以多语言多平台为服务生成惯用的客户端和服务器stubs  
- 双向流和集成身份验证  
基于HTTP2实现双向流传输和完全集成的身份验证  
> gRPC是Cloud Native Computing基金会的孵化项目  
### 什么是RPC？
假设有两台服务器A、B，应用a部署在A服务器上，想要调用B服务器上应用b提供的函数/方法，由于不在同一个内存空间，不能直接调用，此时需要网络来表达调用的语义和传达调用的数据。可用于多台机器组成的集群上部署应用。  

RPC的解决问题的过程如下：  
- 在客户端和服务器之间建立TCP连接，所有调用时交换的数据都在连接里传输。多个远程过程调用共享同一个连接。  
- 解决寻址问题。服务器A上的应用a如何告知底层的RPC框架连接到服务器B（通过主机或IP地址）以及特定的端口，方法的名称等，这样才能完成调用。比如，基于Web服务协议栈的RPC，需要提供endpoint URI；如果RMI（Remote Method Invocation，远程方法调用）的话，还需要一个RMI Registry来注册服务的地址。  
- 当服务器A上的应用a发起RPC时，方法的参数值序列化为二进制的形式，基于底层网络协议，如TCP，通过寻址和传输传递到服务器B。  
- 服务器B收到请求后，对参数进行反序列化，恢复为内存中的表达方式，找到对应的方法（寻址的一部分）进行本地调用，得到返回值。  
- 返回值序列化后发送回服务器A上的应用a，接收后，经历反序列化+恢复，传递给服务器A上的应用。 

可以理解为，RPC就是从一台机器（Client）上通过传参的方式调用另一台机器（Server）上的一个方法（统称为服务）并得到返回的结果。RPC会隐藏底层的通讯细节，即不需要直接处理Socket或HTTP通讯，它是一个请求响应模型：客户端发起请求，服务器返回响应，这是一种类似HTTP的工作方式，在形式上类似调用本地方法，只不过依靠网络可以调用远程的方法。  
**RPC框架的出现让调用者不必关心底层细节，处理了数据序列化、反序列化、连接管理、收发线程、超时处理等问题，是一种优秀的微服务架构。**  
## gRPC如何在HTTP2上传输
### gRPC请求&响应流程  
gRPC在HTTP2的基础上定义了request和response的规范。  
以下是gRPC请求&响应消息流中消息原子的一般顺序：  
- 请求 -> 请求报头*界定的消息（Length-Prefixed-Message）及EOS  
> 长度前缀法适用于二进制协议，HTTP协议的Content-Length头信息用来标记消息体的长度，也可以视为是长度前缀法的一种应用。  

**gRPC的请求报头直接使用HTTP2头部，在HEADERS+CONTINUATION帧中传递。**  
举例说明：单独调用HTTP2帧序列    
```
HEADERS (flags = END_HEADERS)
:method = POST
:scheme = http
:path = /google.pubsub.v2.PublisherService/CreateTopic
:authority = pubsub.googleapis.com
grpc-timeout = 1S
content-type = application/grpc+proto
grpc-encoding = gzip
authorization = Bearer y235.wef315yfh138vh31hv93hv8h3v

DATA (flags = END_STREAM)
<Length-Prefixed Message>
```  
- 响应 -> (响应报头*界定消息尾部)/仅尾部  
**每个响应报头&尾部在单个HTTP2 HEADERS帧块（frame block)中传递。**  
对于响应，流的结束是通过最后接收到的带有尾部的HEADERS帧上存在的END_STREAM标志来指示。  
举例说明：单独调用HTTP2帧序列  
```
HEADERS (flags = END_HEADERS)
:status = 200
grpc-encoding = gzip
content-type = application/grpc+proto

DATA
<Length-Prefixed Message>

HEADERS (flags = END_STREAM, END_HEADERS)
grpc-status = 0 # OK
trace-proto-bin = jher831yy13JHy3hc
```  
### HTTP2传输映射
- 流识别  
所有GRPC调用都需要指定一个内部ID。 在此方案中，我们将使用HTTP2流ID作为调用标识符。注意：这些ID与打开的HTTP2会话相关，并且在处理给定的多个HTTP2会话进程中不是唯一的，也不能用作GUID。  
- 数据帧  
数据帧边界与长度前缀消息边界没有任何关系，实现中不应对数据对齐方式进行任何假设。  
- 错误  
在RPC应用程序或运行时错误时，状态和状态消息将在尾部中传递。  
- 安全  
HTTP2要求使用TLS 1.2或更高版本，对部署中允许的密码有额外约束，以避免已知安全问题，还有需要SNI支持的情况。  
- GOAWAY帧  
由于Stream ID不可被重复使用，如果一条连接上的ID分配完毕，客户端会新建一条连接，服务器则发送给客户端一个GOAWAY帧以指示它们将不再接受关联连接上的任何新流。该帧包含服务器上次成功接受的流的ID。客户应将在最后一个成功接受的流之后发起的任何流视为“不可用”，然后在其他地方重试该调用。客户端可以自由地继续使用已经接受的流，直到它们完成或连接终止。服务器应在终止连接之前发送GOAWAY，通知客户端服务器已接受并正在执行哪些工作。  
- PING帧  
客户端和服务器都可以发送PING帧，对方必须通过准确地回显它们收到的内容来响应。这用于断言该连接仍然有效，并提供一种估算端到端延迟的方法。 如果服务器启动的PING在运行时预期的期限内未收到响应，则服务器上所有未完成的呼叫将以CANCELED状态关闭。 客户端启动的PING过期将导致所有呼叫以UNAVAILABLE状态关闭。  
- 连接失败  
如果客户端上发生可检测到的连接失败，则所有调用将以UNAVAILABLE状态关闭。对于服务器，已开放的调用将以“已取消”状态关闭。  
## 总结：HTTP2是如何在gRPC中发挥优势的？
我们可以通过令HTTP2的优势与其在gRPC中的实现一一对应来总结这个问题：
- 二进制成帧和压缩  
RPC框架中，传参的值是二进制格式的，HTTP2采用二进制格式传输数据，对机器更为友好，使得HTTP2协议在发送和接收方面都是紧凑而高效的。  
- 单个TCP连接和多路复用   
根据帧首部的流标识，可以重新组装多个乱序发送的帧，使得在单个连接可以承载任意数量的双向数据流。HTTP2是标准协议，其流式传输特性完全适合gRPC的要求，因此无需重新设计即可利用。在单个TCP连接上多路复用同时消除了行头阻塞。  
- 服务器推送  
HTTP2可以让服务器预测并提前推送给客户端需要的资源。此外，gRPC可以智能地选择向哪个后端发送流量。（库功能，而不是有线协议功能）。这意味着可以将请求发送到负载最少的后端服务器，而无需使用代理。  
- Protobuf  
> 序列化用Protobuf，通信用HTTP2  

gRPC 的 service 接口是基于 protobuf 定义的，我们可以非常方便的将 service 与 HTTP/2 关联起来，在HTTP2之上带来额外的优势。格式如下：  

```
Path : /Service-Name/{method name}  
Service-Name : ?( {proto package name} "." ) {service name}  
Message-Type : {fully qualified proto message name}  
Content-Type : "application/grpc+proto"  
```    

HTTP2的这些优点为gRPC的**高性能**提供了保障，而gRPC也为通过HTTP2进行流传输提供了广泛的支持（无串流、服务器到客户端流、客户端到服务器流、双向流），在延迟和网络吞吐量方面的表现都很优秀（特别是比起传统的REST+HTTP1），更适用于分布式部署。这些特性使得gRPC在移动设备上表现尤为出色，更省电、节省空间占用。  
当然，gRPC也有其劣势：由于gRPC大量使用HTTP2功能，没有浏览器能提供Web请求所需的控制级别来支持gRPC客户端，比如不允许调用者使用HTTP的要求，或提供对基础HTTP帧的访问（Chrome会崩溃的）。gRPC-Web是gRPC团队的另一项技术，可以在浏览器中提供优先的gRPC。gRPC-Web由两部分组成，一个支持所有现代浏览器的JavaScript客户端，以及服务器上的一个gRPC-Web代理。gRPC-Web客户端调用代理，代理将根据gRPC请求转发到gRPC服务器。gRPC-Web并非支持所有gRPC的功能。不支持客户端和双向流，并且对服务器流的支持有限。  
由于gRPC消息使用Protobuf编码。尽管Protobuf可以高效地发送和接收，但其二进制格式不是人类可读的。Protobuf要求在.proto文件中指定的消息接口描述才能正确反序列化。需要额外的工具来分析网上的Protobuf有效载荷并手动撰写请求。为了解决这个问题，存在诸如服务器反射和gRPC命令行工具之类的功能来辅助二进制Protobuf消息。此外，protobuf消息支持与JSON之间的转换。内置的JSON转换提供了一种在调试时将Protobuf消息与人类可读格式之间相互转换的有效方法。  
## gRPC相关项目
[gRPC](https://github.com/grpc/grpc)该存储库包含在共享C核心库src / core之上编写的以多种语言实现的gRPC库的源代码，并欢迎对其代码库做出贡献。
在大多数语言中，gRPC运行时是作为软件包提供的。不同语言的gRPC库处于发展状态，正在寻求贡献：
|   Language   |    Source Repo   |
|:------------:|:----------------:|
|     Java     | [grpc-java](https://github.com/grpc/grpc-java) |
|      Go      |   [grpc-go](https://github.com/grpc/grpc-go)   |  

|   Language   |    Source   |
|:----------:|:---------:|
|Shared C [core library]|[src/core](https://github.com/grpc/grpc/blob/master/src/core)|
|C++|[src/cpp](https://github.com/grpc/grpc/blob/master/src/cpp)|
|Ruby|[src/ruby](https://github.com/grpc/grpc/blob/master/src/ruby)|
|Python|[src/python](https://github.com/grpc/grpc/blob/master/src/python)|
|PHP|[src/php](https://github.com/grpc/grpc/blob/master/src/php)|
|C#(core library based)|[src/csharp](https://github.com/grpc/grpc/blob/master/src/csharp)|  

### ref
[谁能用通俗的语言解释一下什么是 RPC 框架？](https://www.zhihu.com/question/25536695)  
[RPC简介及框架选择](https://www.jianshu.com/p/b0343bfd216e)  
[gRPC学习笔记](https://zhuanlan.zhihu.com/p/28927834)  
[grpc.io](https://grpc.io/)  
[gRPC over HTTP2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)  
[无所不能的RPC消息协议是如何设计的？](https://yq.aliyun.com/articles/623342)  
[深入了解 gRPC：协议](https://zhuanlan.zhihu.com/p/27595419)  
[Is gRPC(HTTP/2) faster than REST with HTTP/2?](https://stackoverflow.com/questions/44877606/is-grpchttp-2-faster-than-rest-with-http-2)  
[开源中国翻译：gRPC官方文档中文版](http://doc.oschina.net/grpc?t=56831)    
[Compare gRPC services with HTTP APIs](https://docs.microsoft.com/en-us/aspnet/core/grpc/comparison?view=aspnetcore-3.1#grpc-strengths)