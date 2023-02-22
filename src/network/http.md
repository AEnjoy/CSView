---
title: HTTP面试常见题
---


### HTTP和HTTPS的区别？<Badge text="重要" type="danger" />

- HTTP 明文传输，数据都是未加密的，安全性较差，HTTPS(SSL+HTTP）数据传输过程是加密的，安全性较好。

- 使用 HTTPS 协议需要到 CA 申请证书。

- HTTP 页面响应速度比 HTTPS 快，主要是因为 HTTP 使用 TCP 三次握手建立连接，而 HTTPS除了 TCP 的三个包，还要加上SSL握手的消耗。

- 用的端口也不一样，前者是 80，后者是 443。

- HTTPS 其实就是建构在 SSL/TLS 之上的 HTTP 协议，所以，要比较 HTTPS 比 HTTP 要更耗费服务器资源。



### HTTPS的加密与认证过程？<Badge text="重要" type="danger" />

#### ClientHello

首先，由客户端向服务端发起加密通信请求。客户端主要向服务端发送：

- 客户端支持的 SSL/TLS协议版本
- 客户端产生的的随机数(Client Random）
- 客户端支持的密码套件列表

#### SeverHello

服务器收到客户端请求后，向客户端发出响应。服务端回应的内容有：

- 确认 SSL/ TLS 协议版本(如果浏览器不支持，则关闭加密通信）
- 服务端生产的随机数(Server Random）
- 确认的密码套件列表
- 服务端的数字证书

#### 客户端回应

客户端收到服务端的回应之后，首先通过浏览器或者操作系统中的 CA 公钥，确认服务端的数字证书的真实性。如果证书没有问题，客户端会从数字证书中取出服务端的公钥，然后使用它加密报文，向服务端发送如下信息：

- 一个随机数，该随机数会被服务端公钥加密
- 加密通信算法改变通知，表示随后的信息都将用会话秘钥加密通信
- 客户端握手结束通知，表示客户端的握手阶段已经结束
- 之前所有内容的发生的数据做个摘要，用来供服务端校验

服务端和客户端有了三个随机数，接着用双方协商的加密算法，各自生成本次通信的会话秘钥。

#### 服务端回应

服务端收到客户端的第三个随机数(pre-master key）之后，通过协商的加密算法，计算出本次通信的会话秘钥。服务端向客户端发送最后的信息：

- 加密通信算法改变通知，表示随后的信息都将用「会话秘钥」加密通信
- 服务端握手结束通知，表示服务端的握手阶段已经结束
- 之前所有内容的发生的数据做个摘要，用来供客户端校验

接下来，客户端与服务器进入加密通信，就完全是使用普通的 HTTP 协议。





###  HTTPS一定安全可靠吗？<Badge text="掌握" type="tip" />

不安全是因为用户点击接受了中间人服务器的证书。中间人服务器与客户端在 TLS 握手过程中，实际上发送了自己伪造的证书给浏览器，而这个伪造的证书是能被浏览器(客户端）识别出是非法的，于是就会提醒用户该证书存在问题。如果用户点击「继续浏览此网站」，相当于用户接受了中间人伪造的证书，那么后续整个 HTTPS 通信都能被中间人监听了。

HTTPS协议本身到目前为止还是没有任何漏洞的，即使你成功进行中间人攻击，本质上是利用了客户端的漏洞(用户点击继续访问或者被恶意导入伪造的根证书)，并不是HTTPS不够安全。



### HTTP状态码的含义？<Badge text="重要" type="danger" />

100类状态码属于**提示信息**，是协议处理中的一种中间状态，实际用到的比较少。

- 100(继续)：请求者应当继续提出请求。 服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。 
- 101(切换协议）：请求者已要求服务器切换协议，服务器已确认并准备切换。 

200类状态码表示服务器**成功**处理了客户端的请求。

- 200(成功）：表示服务器**响应成功**，也就是服务器找到了客户端请求的内容，并且将内容返回给客户端。
- 204(已创建）：请求成功并且服务器创建了新的资源。
- 206(部分内容）：服务器成功处理了部分GET请求。 

300类状态码表示客户端请求的资源发生了变动，需要客户端用新的 URL 重新发送请求获取资源，也就是**重定向**。

- 301(永久移动）：代表永久性的重定向**，值得注意的是，这种重定向跳转，从严格意义来讲不是服务器跳转，而是客户端跳转**的。这个“跳”的动作是服务器是通过回传状态码301来下达给客户端的，让客户端完成跳转。
- 302(临时移动）：代表**临时跳转**。例如：URL地址A可以向URL地址B上跳转，但这并不是永久性的，在经过一段时间后，URL地址A还可能向URL地址C上跳转。
- 304(未修改）：服务器通过返回状态码304可以告诉客户端请求资源成功，但是这个资源不是由服务器提供返回给客户端的，而是客户端本地浏览器缓存中就有的这个资源，因为可以从缓存中获取这个资源，从而节省传输的开销。

::: tip

301 和 302 都会在响应头里使用字段 `Location`，指明后续要跳转的 URL，浏览器会自动重定向新的 URL。

:::

- 400类状态码表示客户端发送的**报文有误**，服务器无法处理。
- 400(错误请求）：服务器不理解请求的语法。 
- 403(禁止）：代表请求的服务器资源**权限不够**，也就是没有权限去访问服务器的资源，或者请求的IP地址被封掉了。
- 404(未找到）：代表服务器上**没有该资源**，或者说服务器找不到客户端请求的资源，是最常见的请求错误码。

500类状态码表示客户端请求报文正确，但是**服务器处理时内部发生了错误**，属于服务器端的错误码。

- 500(服务器内部错误）：代表**程序错误**，也就是说请求的网页程序本身报错了。在服务器端的网页程序出错。由于现在的浏览器都会对状态码500做一定的处理，所以在一般情况下会返回一个定制的错误页面。
- 501(尚未实施）：服务器不具备完成请求的功能。 例如，服务器无法识别请求方法时可能会返回此代码。 
- 502(错误网关）：通常是服务器作为网关或代理时返回的错误码，表示服务器自身工作正常，访问后端服务器发生了错误。
- 503(服务不可用）：表示服务器当前很忙，暂时无法响应客户端。。 
- 504(网关超时）：服务器作为网关或代理，但是没有及时从上游服务器收到请求。 
- 505(HTTP 版本不受支持）：服务器不支持请求中所用的 HTTP 协议版本。



### HTTP缓存有哪些实现方式？<Badge text="掌握" type="tip" />

**强制缓存**：强缓存指的是只要浏览器判断缓存没有过期，则直接使用浏览器的本地缓存，决定是否使用缓存的主动性在于浏览器这边。

**协商缓存**：请求的响应码304，告诉浏览器可以使用本地缓存的资源，通过服务端告知客户端是否可以使用缓存的方式被称为协商缓存。



### HTTP1.0、HTTP1.1、HTTP2.0和HTTP3.0的区别？<Badge text="重要" type="danger" />

#### HTTP1.0

- **无连接**：每次请求都要建立连接，需要使用 keep-alive 参数建立长连接，建立连接十分消耗资源。
- **队头阻塞**：HTTP1.0规定下一个请求必须在前一个请求响应到达之前才能发送，假设前一个请求响应一直不到达，那么下一个请求就不发送，后面的请求就阻塞了。
- **缓存**：在HTTP1.0中主要使用header里的协商缓存 last-modified\if-modified-since，强缓存Expires来做为缓存判断的标准。

::: tip 提示

**If-Modified-Since**，属附带条件之一，它会告知服务器若If-Modified-Since字段值早于资源的更新时间，则希望能处理该请求。而在指定If-Modified-Since字段值的日期时间之后，如果请求的资源都没有过更新，则返回状态码304 Not Modified的响应。If-Modified-Since用于确认代理或客户端拥有的本地资源的有效性。获取资源的更新日期时间，可通过确认首部字段Last-Modified来确定。

**Expires**是RFC 2616(HTTP1.0）协议中和网页缓存相关字段。用来控制缓存的失效日期。Expires 字段声明了一个网页或 URL 地址不再被浏览器缓存的时间，一旦超过了这个时间，浏览器都应该联系原始服务器。RFC告诉我们：“由于推断的失效时间也许会降低语义透明度，应该被谨慎使用，同时我们鼓励原始服务器尽可能提供确切的失效时间。”

:::

#### HTTP1.1

- **长连接**：好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销，减轻了服务器端的负载。
- **支持管道(pipeline）网络传输**：只要第一个请求发出去了，不必等其回来，就可以发第二个请求出去，可以减少整体的响应时间。
- **缓存处理**：HTTP1.1则引入了更多的缓存控制策略，多可供选择的缓存头来控制缓存策略。
- **断点续传**：HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206(Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

#### HTTP2.0

- **header压缩**：HTTP1的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
- **多路复用**：使用多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级。
- **新的二进制帧格式**：HTTP2.0把请求在应用层切分成二进制帧并标上序号，服务器接收到二进制帧后组装成请求进行处理，从而达到并发发送请求的效果，对于服务器端，响应可以通过序号确定是哪个请求，从而不会出现混乱的问题。
- **服务器推送**：HTTP2.0引入了server push，它允许服务端推送资源给浏览器，在浏览器明确地请求之前，免得客户端再次创建连接发送请求到服务器端获取。这样客户端可以直接从本地加载这些资源，不用再通过网络。

#### HTTP3.0(QUIC)

- HTTP/3.0直接放弃使用TCP，将传输层协议改成UDP，使用UDP实现可靠传输。

- **0-RTT**：缓存当前会话的上下文，下次恢复会话的时候，只需要将之前的缓存传递给服务器，验证通过，就可以进行传输了。(**这是QUIC协议相比HTTP2.0的最大优势**）
- **多路复用**：QUIC基于UDP，一个连接上的多个stream之间没有依赖，即使丢包，只需要重发丢失的包即可，不需要重传整个连接。
- **更好的移动端表现**：QUIC在移动端的表现比TCP好，因为TCP是基于IP识别连接，而QUIC是通过ID识别链接。无论网络环境如何变化，只要ID不便，就能迅速重新连上。
- **加密认证的根文**：TCP协议头没有经过任何加密和认证，在传输过程中很容易被中间网络设备篡改、注入和窃听。QUIC的packet除了个别报文，所有报文头部都是经过认证的，报文Body都是经过加密的。只要对 QUIC 做任何更改，接收端都能及时发现，有效地降低了安全风险。
- **向前纠错机制**：向前纠错(Foward Error Connec，FEC)，每个数据包除了它本身的内容之外还包括了其他数据包的数据，因此少量的丢包可以通过其他包的冗余数据直接组装而无需重传。向前纠错牺牲了每个数据包可以发送数据的上限，但是带来的提升大于丢包导致的数据重传，因为数据重传将会消耗更多的时间(包括确认数据包丢失，请求重传，等待新数据包等步骤的时间消耗)。



### QUIC协议的概念和特点？<Badge text="掌握" type="tip" />

#### 概念

HTTP/3 基于**UDP 协议**在应用层实现了 **QUIC 协议**，具有类似 TCP 的连接管理、拥塞窗口、流量控制的网络特性，让UDP协议变得可靠。

#### 特点

**无队头阻塞**

QUIC 协议有类似 HTTP/2 Stream 与多路复用的概念，可以在同一条连接上并发传输多个 Stream。UDP 不关心数据包的顺序，也不关心是否丢包。UDP将每个数据包都有一个序号唯一标识。当某个流中的一个数据包丢失了，该流的其他数据包到达了，数据也无法被 HTTP/3 读取，QUIC 重传丢失的报文之后数据才会交给 HTTP/3。而其他流的数据报文只要被完整接收，HTTP/3 就可以读取到数据。QUIC 连接上的多个 Stream 之间并没有依赖，都是独立的，某个流发生丢包了，只会影响该流，其他流不受影响。

**快速连接建立**

对于 HTTP/1 和 HTTP/2 协议，TCP 和 TLS 是分层的，分别属于内核实现的传输层、openssl 库实现的表示层，需要分批次来握手，先 TCP 握手，再 TLS 握手。

HTTP/3 在传输数据前虽然需要 QUIC 协议握手，这个握手过程只需要 1 RTT，握手的目的是为确认双方的「连接 ID」，连接迁移就是基于连接 ID 实现的。

HTTP/3 的 QUIC 协议不与 TLS 分层，QUIC 内部包含了 TLS，它在自己的帧会携带 TLS 记录，只需 1 个 RTT 就可以完成建立连接与密钥协商。在第二次连接的时候，应用数据包可以和 QUIC 握手信息(连接信息 + TLS 信息）一起发送。

**连接迁移**

QUIC 协议没有用四元组的方式来绑定连接，而是通过**连接 ID**来标记通信的两个端点，客户端和服务器可以各自选择一组 ID 来标记自己，因此即使移动设备的网络变化后，导致 IP 地址变化了，只要仍保有上下文信息(比如连接 ID、TLS 密钥等），就可以复用原连接，消除重连的成本，达到了**连接迁移**的功能。





### QUIC如何保证可靠传输？<Badge text="了解" type="info" />

**Packet Header**：分为Long Packet Header用于首次建立连接、和Short Packet Header用于日常传输数据。QUIC也是需要三次握手来建立连接的，目的是为了协商连接ID。协商出连接ID后，后续传输时，双方只需要固定住连接 ID，从而实现连接迁移功能。Short Packet Header 中的 `Packet Number` 每个报文有独一无二的编号，并且严格递增。单调递增的设计，可以让数据包不再像 TCP 那样必须有序确认，当数据包Packet N 丢失后，只要有新的已接收数据包确认，当前窗口就会继续向右滑动，从而解决了队头阻塞的问题。

**QUIC Frame Header**：一个 Packet 报文中可以存放多个 QUIC Frame。用于传输的Stream Frame有Stream ID、Offset和length字段，Stream ID用于多个并发传输的 HTTP 消息，通过不同的 Stream ID 加以区别、Offset字段类似于 TCP 协议中的 Seq 序号，保证数据的顺序性和可靠性；Length标识了 Frame 数据的长度。如果发生丢包了进行重传，通过比较两个数据包的Stream ID 与 Stream Offset，如果都是一致，就说明这两个数据包的内容一致。

所以，QUIC 通过单向递增的 Packet Number，配合 Stream ID 与 Offset 字段信息，可以支持乱序确认而不影响数据包的正确组装，摆脱了TCP必须按顺序确认的限制。



### HTTP的GET和POST方法区别？<Badge text="重要" type="danger" />

- GET一般用来从服务器上获取资源，POST一般用来更新服务器上的资源。
- GET是幂等的，即读取同一个资源总是得到相同的数据，而POST不是幂等的，因为每次请求对资源的改变并不是相同的。
- GET不会改变服务器上的资源，而POST会对服务器资源进行改变。
- GET请求的数据会附在URL之后，即将请求数据放置在HTTP报文的请求头中，以?分割URL和传输数据，参数之间以&相连；而POST请求会把提交的数据则放置在是HTTP请求报文的请求体中。
- POST的安全性要比GET的安全性高，因为GET请求提交的数据将明文出现在URL上，而且POST请求参数则被包装到请求体中，相对更安全。
- 从请求的大小看，GET请求的长度受限于浏览器或服务器对URL长度的限制，允许发送的数据量比较小，而POST请求则是没有大小限制的。



### GET请求可以带body吗？<Badge text="掌握" type="tip" />

RFC规范并没有规定GET请求不能带 body 的。任何请求都可以带 body 的。 GET 请求是获取资源，所以根据这个语义不需要用到 body。URL 中的查询参数也不是 GET 所独有的，POST 请求的 URL 中也可以有参数的。



### 既然有HTTP协议，为什么还要有RPC？<Badge text="掌握" type="tip" />

**定义**：TCP是传输层的协议，基于TCP造出来的HTTP和各类RPC协议，它们都只是定义了不同消息格式的应用层协议而已。HTTP协议是超文本传输协议。RPC是远程过程调用。它不是一个具体的协议，而是一种调用方式。虽然大部分RPC协议底层使用TCP，但实际上它们不一定非要用TCP，改用UDP或者HTTP。HTTP主要用于b/s架构，而RPC更多用于c/s架构。

**服务发现**：HTTP中知道服务的域名，就可以通过DNS服务去解析得到IP地址，默认80端口。RPC一般会有专门的中间服务去保存服务名和IP信息，比如consul或者etcd，或者是redis。想要访问某个服务，就去这些中间服务去获得IP和端口信息。由于dns也是服务发现的一种，所以也有基于dns去做服务发现的组件，比如CoreDNS。

**底层连接方式**：HTTP协议默认在建立底层TCP连接之后会一直保持这个连接(keep alive），之后的请求和响应都会复用这条连接。RPC协议也是通过建立TCP长链接进行数据交互，但RPC协议一般还会再建个连接池，在请求量大的时候，建立多条连接放在池内，要发数据的时候就从池里取一条连接出来，用完放回去便于下次再复用。

**传输的内容**：HTTP设计初是用于做网页文本展示的，传的内容以字符串为主，有header和body。在body这块，它使用json来序列化结构体数据，内容会有冗余。RPC因为定制化程度更高，可以采用体积更小的protobuf或其他序列化协议去保存结构体数据，同时也不需要像HTTP那样考虑各种浏览器行为，比如302重定向。因此性能也会更好一些，这也是在公司内部微服务中抛弃HTTP，选择使用RPC的最主要原因。

**历史包袱**：RPC其实比HTTP出现的要早，且比目前主流的HTTP1.1**性能**要更好，所以大部分公司内部都还在使用RPC。HTTP2.0**在**HTTP1.1的基础上做了优化，性能可能比很多RPC协议都要好，但由于是这几年才出来的，所以也不太可能取代掉RPC。



### 什么是XSS攻击？有什么解决办法？<Badge text="掌握" type="tip" />

XSS是指恶意攻击者利用网站没有对用户提交数据进行转义处理或者过滤不足的缺点，进而添加一些脚本代码嵌入到web页面中去，使别的用户访问都会执行相应的嵌入代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。

#### 分类

**反射性XSS攻击(非持久性XSS攻击）**：漏洞产生的原因是攻击者注入的数据反映在响应中。

**持久性XSS攻击 (留言板场景)**：指XSS攻击代码存储在网站数据库，每当用户使用浏览器打开指定页面时，脚本就执行。与非持久性XSS攻击相比，持久性XSS攻击危害性更大。

#### 危害

- 盗取各类用户帐号，如机器登录帐号、用户网银帐号、各类管理员帐号
- 控制企业数据，包括读取、篡改、添加、删除企业敏感数据的能力
- 盗窃企业重要的具有商业价值的资料
- 非法转账
- 强制发送电子邮件
- 网站挂马
- 控制受害者机器向其它网站发起攻击

#### 防范

漏洞产生的根本原因是**太相信用户提交的数据，对用户所提交的数据过滤不足所导致的**，解决方案也应该从这个方面入手：

- 将重要的cookie标记为http only：Javascript中的document.cookie语句就不能获取到cookie了(如果在cookie中设置了HttpOnly属性，那么通过js脚本将无法读取到cookie信息，这样能有效的防止XSS攻击）

- 表单数据规定值的类型：例如：年龄应为只能为int、name只能为字母数字组合

- 对数据进行HTML编码处理

- 过滤或移除特殊的HTML标签：例如: 

  ```js
  <script> , <iframe> , < for < , > for>
  ```

- 过滤JavaScript事件的标签：例如 "onclick=", "onfocus="



### 什么是CSRF攻击？有什么解决办法？<Badge text="掌握" type="tip" />

#### 概念

CSRF就是跨站请求伪造。

- 登录受信任网站A，并在本地生成Cookie(如果用户没有登录网站A，那么网站B在诱导的时候，请求网站A的api接口时，会提示你登录）

- 在不登出A的情况下，访问危险网站B(其实是利用了网站A的漏洞）

![](https://pic.imgdb.cn/item/63ef2495f144a010073bbbc9.jpg)



#### 防范

**Token验证**

服务器发送给客户端一个token，客户端提交的表单中带着这个token，如果这个 token 不合法，那么服务器拒绝这个请求。

**隐藏令牌**

把token隐藏在http的head头中。

**Referer验证**

只接受本站的请求，服务器才做响应；如果不是，就拦截。



### 中间人攻击以及如何防范？<Badge text="掌握" type="tip" />

#### 概念

指攻击者与通讯的两端分别创建独立的联系，并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制。

![](https://pic.imgdb.cn/item/63ef232df144a01007391046.png)

**嗅探**：嗅探或数据包嗅探 是一种用于捕获流进和流出系统/网络的数据包的技术。网络中的数据包嗅探就好像电话中的监听。

**数据包注入**：攻击者会将恶意数据包注入常规数据中。这样用户便不会注意到文件/恶意软件，因为它们是合法通讯流的一部分。在中间人攻击和拒绝式攻击中，这些文件是很常见的。

**会话劫持**：当客户端和服务器端在进行一个会话时，会话中包含了很多重要信息，一些黑客会潜伏在这个会话中，最终控制这个会话，这既是会话劫持。

**SSL剥离**：SSL/TLS证书通过加密保护着的通讯安全。在SSL剥离攻击中，攻击者使SSL/TLS连接剥落，随之协议便从安全的HTTPS变成了不安全的HTTP。

#### 防范

使用HTTPS协议，禁用不安全的SSL协议，启用虚拟专用网(VPN)，

通过 **HTTPS 双向认证**来避免这种问题：一般 HTTPS 是单向认证，客户端只会验证了服务端的身份，但是服务端并不会验证客户端的身份。如果用了双向认证方式，不仅客户端会验证服务端的身份，而且服务端也会验证客户端的身份。服务端一旦验证到请求自己的客户端为不可信任的，服务端就拒绝继续通信，客户端如果发现服务端为不可信任的，那么也中止通信。





[^1]: https://xiaolincoding.com/network/2_http/http_interview.html#get-%E4%B8%8E-post
[^2]: https://xiaolincoding.com/network/2_http/http_rpc.html
[^3]: https://blog.51cto.com/u_15526925/5732537
[^4]: https://xiaolincoding.com/network/2_http/http_interview.html
[^5]: https://xiaolincoding.com/network/3_tcp/quic.html
[^6]: https://xiaolincoding.com/network/2_http/http_interview.html