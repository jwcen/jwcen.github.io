---
layout: post
title:  "[计网] HTTP和HTTPS的区别"
categories: [Computer-Network]
tags: [http/https]
permalink: /posts/:categories/:title:output_ext
description: HTTP和HTTPS的区别，详解 HTTP 和 https
math: true
---

> 如需转载，请附上链接：[https://jwcen.github.io/](https://jwcen.github.io/)
{: .prompt-tip}

* This will become a table of contents (this text will be scrapped).
{:toc}

## HTTP 
HTTP 是超文本传输协议，也就是HyperText Transfer Protocol。  
HTTP 是一个在计算机世界里专门在「两点」之间「传输」文字、图片、音频、视频等「超文本」数据的「约定和规范」。
拆成三个部分：
- 超文本：HTTP 传输的内容是「超文本」。
- 传输：A->B, B-> A；允许中间有中转或接力。
- 协议：一种计算机之间交流通信的规范。

### HTTP 报文 
http 报文大概分三部分：**起始行、首部、实体。**

![](../../assets/img/note_img/20230225170958.png)_http请求报文格式_

### HTTP 常见首部字段

| Header         | 作用                                            |
| -------------- | ----------------------------------------------- |
| Host           | 客户端发送请求时，用来指定服务器的域名          |
| Content-Length | 服务器回应时，告诉客户端，本次数据是什么格式    |
| Connection     | 最常用于客户端要求服务器使用「HTTP 长连接」机制 |


### HTTP 状态码
{% details 1xx 表示服务器已收到请求，处理将继续 %}
- 100 继续 – 现在一切正常，继续。
- **101 切换协议** – 有消息，例如升级请求、正在将事物更改为不同的协议。
- 102 处理 – 正在发生但尚未完成。
{% enddetails %}


{% details 2xx 表示客户端请求已被服务器接收、理解、和处理 %}
- 200 请求成功，有响应体
- 201 已创建 — 与 200 类似，但衡量成功的标准是创建了新资源。
- 204 无内容 — 请求已发送，但正文中没有内容。
- 205 重置内容 — 将文档重置为原始状态，例如，清除表单。
- 206 部分内容 — 只发送了部分内容。
{% enddetails %}


{% details 3xx 重定向相关，表示资源位置发生变动，需客户端重新发送请求 %}
- **301 永久跳转** – 旧资源现在重定向到新的资源上。（会缓存）
- **302 临时跳转** – 旧资源现在临时重定向到新资源。（不缓存）
- **304 无修改** – 表示页面没有被修改。通常用于缓存。
{% enddetails %}

{% details 4xx 表示客户端有错误，请求报文有误，服务器无法处理。 %}
- 400 客户端请求的语法错误
- **401 未认证响应**：是由于用户没有进行身份认证或者身份认证不对。
- **403 拒绝响应**：是当用户通过了身份验证，但无权对给定资源执行请求的操作（比如没有读写权限）。
{% enddetails %}

{% details 5xx 表示服务器有错误，在服务器在处理请求时内部发生了错误。 %}
- **500 内部服务器错误** – 服务器遇到某种问题、并且没有更好或更具体的错误代码。
- 501 无法实现 – 服务器不支持请求方法。
- **502 网关错误** – 服务器处于请求中间状态。但是它从它路由到的服务器收到了错误的响应。
- 503 暂停服务 – 服务器因维护而过载或停机，现在无法处理请求。它可能很快就会恢复。
- **504 网关超时** – 服务器处于请求中间状态。但是没有收到来自它路由到的服务器的及时响应。
{% enddetails %}


### HTTP请求方法

| 方法    | 说明                                                                                                                    |
| ------- | ----------------------------------------------------------------------------------------------------------------------- |
| GET     | **请求指定的资源**，一般用于数据的读取                                                                                  |
| HEAD    | 与GET方法一样，服务器不会回传响应主体。  场景：客户端查看服务器的性能；检查文件是否存在/最新版本                        |
| POST    | **向指定资源提交数据，请求服务器进行处理**，  如：表单数据提交、文件上传等，请求数据会被包含在请求体中。                |
| PUT     | 向指定资源位置上传其最新内容                                                                                            |
| PATCH   | 与PUT请求类似，同样用于资源的更新。                                                                                     |
| DELETE  | 用于请求服务器删除所请求URI所标识的资源                                                                                 |
| CONNECT | HTTP/1.1协议预留的，能够将连接改为管道方式的代理服务器。  用于SSL加密服务器的链接 与 非加密的HTTP代理服务器之间的通信。 |
| OPTIONS | 与HEAD类似，一般也是用于客户端查看服务器的性能。                                                                        |
| TRACE   | 服务器会将通信路径返回给客户端。用于HTTP请求的测试或诊断。                                                              |

> 安全和幂等的概念：
> - 在 HTTP 协议里，所谓的「安全」是指**请求方法不会「破坏」服务器上的资源。**
> - 所谓的「幂等」，意思是**多次执行相同的操作，结果都是「相同」的。**
{: .prompt-tip}


{% details GET 和 POST 区别 %}
-  **语义**。
   -  GET是请求获取指定的资源
   -  POST 是向指定资源提交数据，请求服务器进行处理
- **传参方式**。
  - GET 通过 query URL或Cookie传参
  - POST将数据放在body中（HTTP协议用法的约定）。
- **提交的数据大小**。
  - GET 可提交的数据量受到URL长度的限制（特定的浏览器及服务器对它的限制，HTTP 没有对其限制）
  - POST 是没有大小限制的（安全考虑，服务器软件实现时会做限制）。
- 安全和幂等性。
  - GET 请求是安全和幂等的。POST请求不是。
{% enddetails %}


{% details POST、PUT、PATCH 区别 %}
都是向服务器端发送数据的。  
区别：
- PUT、PATCH 请求是幂等的，POST 不是
- PUT 更新数据时是全部更新，PATCH 只进行部分更新，POST 的话会创建新的内容
{% enddetails %}


### HTTP 缓存技术
浏览器缓存是浏览器对之前请求过的文件进行缓存，以便下一次访问时重复使用，减少带宽、服务器压力，加快网页响应速度。

HTTP/1.0 提出缓存概念，即强缓存 `Expires` 和协商缓存 `Last-Modified`。后 HTTP/1.1 又有了更好的方案，即**强缓存 `Cache-Control` 和协商缓存 `ETag`**。

缓存相关的请求/响应头  

|       Header             |           作用         |
| ------ | ------ |
| Expires | 响应头，代表该资源的过期时间。  |
| Cache-Control | 请求/响应头，缓存控制字段，精确控制缓存策略。  |
| If-Modified-Since | 请求头，资源最近修改时间，由浏览器告诉服务器。  |
| Last-Modified | 响应头，资源最近修改时间，由服务器告诉浏览器。  |
| Etag | 响应头，资源标识，由服务器告诉浏览器。  |
| If-None-Match | 请求头，缓存资源标识，由浏览器告诉服务器。  |


#### 强制缓存和协商缓存

- 强制缓存
  - 只要浏览器判断**缓存没有过期，则直接使用本地缓存**，决定是否使用缓存的主动性在于浏览器这边。
  - 通过设置`Expires`和`Cache-Control`(优先级更高) 两种响应头实现
    - Expires: 绝对时间, 客户端与服务端的时间时差或误差等因素可能造成客户端与服务端的时间不一致。
    - Cache-Control: 相对时间，它解决了绝对时间的带来的问题。`Cache-Control: max-age=315360000`
- 协商缓存就得和服务器协商确认下这个缓存能不能用。
  - 由 `Last-Modified` / `IfModified-Since`， `Etag` /`If-None-Match`实现
  - 每次请求需要让服务器判断一下资源是否更新过，从而决定浏览器是否使用缓存，如果是，则返回 304，否则重新完整响应。

> 强缓存作用于那些不怎么变化的资源(如js，css等)，协商缓存适用常更新的文件(如 html)。



## HTTP/1.0 和 HTTP/1.1 区别
### 1. 改进持久连接
HTTP/1.0 默认使用短连接。每进行一次 HTTP 通信，都要经历**建立 TCP 连接、传输数据、断开 TCP 连接**三个阶段。
- 单个页面的资源文件越来越多，每次通信都要建连、传输、断开，会增加大量无谓的开销

HTTP/1.1 增加了**持久连接的方法，并支持请求管道化**--在一个TCP连接上可以传输多个HTTP 请求，只要浏览器或者服务器没有明确断开连接，那么该TCP连接会一直保持。  
- 有效减少 TCP 建连和断连的次数，这降低了通信的延迟
- 默认开启长连接，`Connection: keep-alive`
- 目前浏览器对同一个域名，默认允许同时建立 6 个 TCP 持久连接

### 2. 不成熟的 HTTP 管线化
持久连接虽然能减少 TCP 的建立和断开次数，但 HTTP1.x 由于采用了序列化的请求/响应模型，服务器只能处理一个请求/响应, 它需要等待前面的请求返回之后，才能进行下一次请求。如果 TCP 通道中的某个请求因为某些原因没有及时返回，那么就会阻塞后面的所有请求，这就是著名的**队头阻塞**的问题。  

HTTP/1.1 中试图通过管线化的技术来解决队头阻塞的问题。
- 虽然可以整批发送请求，不过服务器依然需要根据请求顺序来回复浏览器的请求。

### 3. 提供虚拟主机的支持
HTTP/1.0中，每个域名绑定了一个唯一的IP地址，因此一个服务器只能支持一个域名。

> 现实需求：可能一台物理机绑定了多个虚拟主机，各自有单独域名，共享同一ip地址。

HTTP/1.1的请求头中增加了`Host`字段，用来表示当前的域名地址，这样服务器就可以根据不同的`Host`值做不同的处理。

### 4. 提供对动态生成的内容的支持
HTTP/1.0，需要在响应头中设置完整的数据大小，如`Content-Length: 901`，浏览器根据设置的数据大小来接收数据。

缺点：⻚面是动态生成的，在传输数据之前并不知道最终数据大小，导致浏览器不知道何时会接收完所有的文件数据。

HTTP/1.1 通过引入**分块传输编码** `Chunk transfer` 机制来解决这个问题，服务器会将数据分割成若干个任意大小的数据块，每个数据块发送时会**附上上一个数据块的⻓度**，最后使用一个**零⻓度的块**作为发送数据完成的标志。

### 5. 更多的缓存机制

## HTTP/1.x 和 HTTP/2.0 区别


<!-- HTTP/1.0 是比较早在网页上使用的应用层协议，用于一些较简单的网页上和网络请求上。它的特点：
- 无状态、无连接的应用层协议
- 默认短连接。
- 在每个请求中都会发送版本控制信息 `GET / HTTP/1.0`
- 在响应开始时还会发送状态代码行。`200 OK`
- 引入了首部字段，如 `Content-Type`，可以传输纯 HTML 文件以外的文档。
- 提供了简单的缓存机制。通过首部的`If-Modified-Since`,`Expires` 来做为缓存判断的标准。

**缺点:**
- **默认使用短连接。** 每次请求建立 TCP 连接，服务完成立即断开，会增加大量无谓的开销（`Connection: keep-alive` 强制开启长连接）。
- 错误状态响应码少。16种。
- 缓存机制简单。
- HTTP层队头阻塞。下一个请求必须在前一个请求响应到达之后才能发送。 -->

### HTTP/1.x 的缺陷
#### 1. HTTP/1.x 对带宽的利用率却并不理想（核心问题）
> 带宽是指每秒最大能发送或者接收的字节数。
> - 每秒能发送的最大字节数称为上行带宽
> - 每秒能够接收的最大字节数称为下行带宽
{: .prompt-tip}

之所以说HTTP/1.1对带宽的利用率不理想，是因为HTTP/1.1很难将带宽用满。比如我们常说的100M带宽， 实际的下载速度能达到12.5M/S，而采用HTTP/1.1时，也许在加载⻚面资源时最大只能使用到2.5M/S，很难 将12.5M全部用满。原因：
- HTTP/1.1 队头阻塞的问题
  - 虽然能公用一个TCP管道，但是在一个管道中同 一时刻只能处理一个请求，在当前的请求没有结束之前，其他的请求只能处于阻塞状态。
  - 在这个等待的过程中，带宽、CPU都被白白浪费了
- TCP的慢启动
  - TPC 建立后进入发送数据状态，需要进行慢启动，逐渐增加发送窗口的大小。
  - 这个过程可能会导致初始数据传输速度较慢，影响页面加载速度。
  - 慢启动是 TCP 为了减少网络拥塞的一种策略，无法改变。

- 同时开启了多条TCP连接，这些连接会竞争固定的带宽。
  - 当带宽充足时，每条连接发送或者接收速度会慢慢向上增加; 而一旦带宽不足时，这些TCP连接又会减慢发送或者接收的速度。
  - 多条TCP连接之间又不能协商让哪些关键资源优先下载，这样就有可能影响那些关键资源的下载速度了。

#### 2. 无状态特性--带来的巨大HTTP头部
由于 HTTP/1.x 的无状态性，Header会携带比较多的固定首部字段，有时多达几百甚至上千字节，而 Body 只有几十字节（如GET请求，204/301/304响应）。

Header里携带的内容过大，在一定程度上增加了传输的成本。更要命的是，成千上万的请求响应报文里有很多字段值都是重复的，非常浪费。

#### 3. 明文传输--带来的不安全性

#### 4. 不支持服务器推送消息

<!-- HTTP1.1 是 HTTP 的第一个标准化版本，继承了 1.0 的简单，并引入了许多改进：
- **默认开启长连接**。连接可以重复使用，从而节省了时间。
- **添加了管道机制**。这允许在第一个请求的响应完成传输之前发送第二个请求。这降低了通信的延迟。
- 支持分块传输编码。服务端每产生一块数据，就发送一块，用” 流模式” 取代” 缓存模式”。`transfer-encoding:chunked` ，最后一块字节数为 0.
- 更多的[缓存控制机制](https://jwcen.github.io/posts/computer-network/HTTP-HTTPS.html#http-缓存技术)。
- `Host` 首部字段，使得一个服务器能够用来创建多个Web站点。（可能多个虚拟主机共享同一ip地址）

**HTTP/1.1 瓶颈**：
- 头部（Header）未经压缩就发送，首部信息越多延迟越大，只能压缩 Body 的部分；
- 发送冗长的首部。每次互相发送相同的首部造成的浪费较多；
- 没有解决响应的 **HTTP 层队头阻塞**问题。（服务器是按请求的顺序响应的）
- 没有请求优先级控制；
- 请求只能从客户端开始，服务器只能被动响应。 -->

### HTTP/2.0 改进 
HTTP/2的思路就是**一个域名只使用一个TCP⻓连接来传输数据**，这样整个页面资源的下载过程只需要一次慢启动，同时也避免了多个TCP连接竞争带宽所带来的问题。

HTTP/2 通过完全多路复用机制，解决HTTP/1.x 队头阻塞的问题。

> HTTP/2 是 HTTP 协议自 1999 年 HTTP 1.1 发布后的首个更新，主要基于 [SPDY 协议](https://juejin.cn/post/6844903968380813325#heading-8)。


HTTP/2 采用**多路复用机制**解决影响HTTP/1.1效率的三个主要因素: TCP的慢启动、多条TCP连接竞争带宽和队头阻塞。
- HTTP/2 通过引入二进制分帧层，实现了HTTP的多路复用技术
 ![](../../assets/img/note_img/20230304215832.png){: width="300", height="600"}
-  支持在一个 TCP 连接上同时发送多个请求或响应，这些请求或响应可以并行传输且不用按照顺序一一对应 
- 因为在传输过程中，HTTP/2使用二进制帧将请求和响应分成多个小的帧，这些帧可以交错传输并且不需要按照请求的顺序进行传输，浏览器和客户端会根据相同的编号的帧ID有序组装成 HTTP 消息。

其他新特性：
- **二进制格式**。
  - 1.x 版本的头信息是纯文本（ASCII 编码），数据体可以是文本或者二进制；
  - 2.0 中，将请求和响应数据分割为更小的帧，并且它们采用二进制编码。这增加了数据传输的效率。
- **header压缩**。
  - 1.x 的 header 带有大量信息，而且每次都要重复发送； 
  - 2.0使用 encoder 来减少需要传输的header 大小，通信双方维护一张头信息表，既避免了重复 header 的传输，又减小了需要传输的大小。
- **服务器主动推送资源。**


<!-- HTTP/2 为了解决 HTTP/1.1 中仍然存在的效率问题。相比 HTTP/1.x 多的新特性：

- **多路复用**。引出了数据流 Stream 概念，多个 Stream 复用一条 TCP 连接，达到并发的效果。
  - 关键之一就是在应用层和传输层之间增加一个**二进制分帧层**。
  - 不同的请求通过 Stream ID 来区分，接收端可以通过 Stream ID 有序组装成 HTTP 消息，所以不同 Stream 的帧是可以乱序发送的，即 HTTP/2 可以并行交错地发送请求和响应。
- **服务器主动推送资源。** -->

**HTTP/2.0 缺陷：** TCP层队头阻塞。
- HTTP/2 是基于 TCP 协议来传输数据的，TCP 是字节流协议，可能发生丢包
- 一旦发生了丢包现象，就会触发 **TCP 的重传机制**，这样在一个 TCP 连接中的所有的 HTTP 请求都必须等待这个丢了的包被重传回来。

所以 HTTP/3 把 HTTP 下层的 TCP 协议改成了 UDP！

> HTTP/2.0 和 SPDY 区别                    
> - HTTP/2.0 支持明文 HTTP 传输，不强制 HTTPS，而 SPDY 强制使用 HTTPS
> - HTTP/2.0 消息头的压缩算法采用 [HPACK](http://http2.github.io/http2-spec/compression.html)，而非 SPDY 采用的 [DEFLATE](http://zh.wikipedia.org/wiki/DEFLATE)
{: .prompt-tip}

#### HTTP/2.0 的多路复用和 HTTP/1.X 中的长连接复用有什么区别？
- HTTP/1.* **一次请求-响应。** 建立一个连接，用完关闭；每一个请求都要建立一个连接；开销大。
- HTTP/1.1 **管道方式。** 若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，即 HTTP 队头阻塞；
- HTTP/2 **并发传输。** 多个请求可同时在一个连接上并行执行。某个请求任务耗时严重，不会影响到其它连接的正常执行。但下层基于TCP，存在 TCP层队头阻塞问题。

#### 服务器推送到底是什么？
- 服务端推送能把客户端所需要的资源伴随着index.html一起发送到客户端，**省去了客户端重复请求的步骤。**
- 正因为没有发起请求，建立连接等操作，所以**静态资源通过服务端推送的方式可以极大地提升速度。**

### HTTP/3.0 - QUIC
QUIC (Quick UDP Internet Connections), 快速 UDP 互联网连接。被寄予厚望的下一代互联网传输协议，优点：
- 多路复用。QUIC 升华了 HTTP/2 的多路复用技术，实现基于互相独立的多流（多通道）数据传输，根本上解决了 TCP 存在的队头阻塞问题。
- 快速握手。
- 连接迁移。
- 

#### 缺点


## HTTPS 




----
参考
> - https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP
> - HTTP/2.0 相比1.0有哪些重大改进？深入研究：HTTP2 的真正性能到底如何HTTP/2 头部压缩技术介绍
> - https://juejin.cn/post/6855470356657307662

> 如需转载，请附上链接：[https://jwcen.github.io/](https://jwcen.github.io/)
{: .prompt-tip}