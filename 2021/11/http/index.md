# http 协议的发展


**HTTP**（HyperText Transfer Protocol）是万维网（World Wide Web）的基础协议。自 Tim Berners-Lee 博士和他的团队在 1989-1991 年间创造出它以来，HTTP 已经发生了太多的变化，在保持协议简单性的同时，不断扩展其灵活性。如今，HTTP 已经从一个只在实验室之间交换文件的早期协议进化到了可以传输图片，高分辨率视频和 3D 效果的现代复杂互联网协议。

## 一、HTTP/0.9 – 单行协议

最初版本的 HTTP 协议并没有版本号，后来它的版本号被定位在 0.9 以区分后来的版本。 HTTP/0.9 极其简单：请求由单行指令构成，以唯一可用方法`GET`开头，其后跟目标资源的路径。

```
GET /index.html
```

上面命令表示，TCP 连接（connection）建立后，客户端向服务器请求（request）网页`index.html`。

响应也极其简单的：只包含响应文档本身。

```
<html>
  <body>Hello World</body>
</html>
```

跟后来的版本不同，HTTP/0.9 的响应内容并不包含 HTTP 头，这意味着只有 HTML 文件可以传送，无法传输其他类型的文件；也没有状态码或错误代码：一旦出现问题，一个特殊的包含问题描述信息的 HTML 文件将被发回，供人们查看。服务器发送完毕，就关闭 TCP 连接。

## 二、HTTP/1.0 – 构建可扩展性

1996 年 5 月，HTTP/1.0 版本发布，内容大大增加。

- 首先，任何格式的内容都可以发送。这使得互联网不仅可以传输文字，还能传输图像、视频、二进制文件。这为互联网的大发展奠定了基础。
- 其次，除了 `GET` 命令，还引入了 `POST` 命令和 `HEAD` 命令，丰富了浏览器与服务器的互动手段。
- 再次，HTTP 请求和回应的格式也变了。除了数据部分，每次通信都必须包括头信息（HTTP header），用来描述一些元数据。
- 其他的新增功能还包括状态码（status code）、多字符集支持、多部分发送（multi-part type）、权限（authorization）、缓存（cache）、内容编码（content encoding）等。

### 2.1 请求格式

下面是一个 1.0 版的 HTTP 请求的例子。

```
GET / HTTP/1.0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
Accept: */*
```

可以看到，这个格式与 0.9 版有很大变化。

第一行是请求命令，必须在尾部添加协议版本（HTTP/1.0）。后面就是多行头信息，描述客户端的情况。

### 2.2 响应格式

服务器的响应如下。

```
HTTP/1.0 200 OK
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
```

响应的格式是"头信息 + 空行 + 数据"。其中，第一行是"协议版本 + 状态码 + 状态描述"。

### 2.3 Content-Type 字段

关于字符的编码，1.0 版规定，头信息必须是 ASCII 码，后面的数据可以是任何格式。因此，服务器回应的时候，必须告诉客户端，数据是什么格式，这就是 Content-Type 字段的作用。

下面是一些常见的 Content-Type 字段的值。

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

这些数据类型总称为 MIME type，每个值包括一级类型和二级类型，之间用斜杠分隔。

除了预定义的类型，也可以自定义类型。

MIME type 还可以在尾部使用分号，添加参数。

```
Content-Type: text/html; charset=utf-8
```

上面的类型表明，发送的是网页，而且编码是 UTF-8。

客户端请求的时候，可以使用 Accept 字段声明自己可以接受哪些数据格式。

```
Accept: */*
```

上面代码中，客户端声明自己可以接受任何格式的数据。

### 2.4 Content-Encoding 字段

由于发送的数据可以是任何格式，因此可以把数据压缩后再发送。Content-Encoding 字段说明数据的压缩方法。

```
Content-Encoding: gzip
Content-Encoding: compress
Content-Encoding: deflate
```

客户端在请求时，用 Accept-Encoding 字段说明自己可以接受哪些压缩方法。

```
Accept-Encoding: gzip, deflate
```

### 2.5 缺点

HTTP/1.0 版的主要缺点是，每个 TCP 连接只能发送一个请求。发送数据完毕，连接就关闭，如果还要请求其他资源，就必须再新建一个连接。

TCP 连接的新建成本很高，因为需要客户端和服务器三次握手，并且开始时发送速率较慢（slow start）。所以，HTTP 1.0 版本的性能比较差。随着网页加载的外部资源越来越多，这个问题就愈发突出了。

## 三、HTTP/1.1 – 标准化的协议

在 1997 年初，HTTP/1.1 标准发布，就在 HTTP/1.0 发布的几个月后。它进一步完善了 HTTP 协议，一直用到了 20 年后的今天，直到现在还是最流行的版本。

HTTP/1.1 消除了大量歧义内容并引入了多项改进：

- 连接可以复用，节省了多次打开 TCP 连接加载网页文档资源的时间。
- 增加管线化技术，允许在第一个应答被完全发送之前就发送第二个请求，以降低通信延迟。
- 支持响应分块。
- 引入额外的缓存控制机制。
- 引入内容协商机制，包括语言，编码，类型等，并允许客户端和服务器之间约定以最合适的内容进行交换。
- 感谢`Host`头，能够使不同域名配置在同一个 IP 地址的服务器上。

### 3.1 持久连接

HTTP/1.1 的最大变化，就是引入了持久连接（persistent connection），即 TCP 连接默认不关闭，可以被多个请求复用，不用声明 `Connection: keep-alive`。

客户端和服务器发现对方一段时间没有活动，就可以主动关闭连接。不过，规范的做法是，客户端在最后一个请求时，发送 `Connection: close`，明确要求服务器关闭 TCP 连接。  
目前，对于同一个域名，大多数浏览器允许同时建立 6 个持久连接。

### 3.2 管道机制

1.1 版还引入了管道机制（pipelining），即在同一个 TCP 连接里面，客户端可以同时发送多个请求。这样就进一步改进了 HTTP 协议的效率。

![](https://s2.loli.net/2022/03/29/ZHcY6E2Xhwtakvb.png)

举例来说，客户端需要请求两个资源。以前的做法是，在同一个 TCP 连接里面，先发送 A 请求，然后等待服务器做出回应，收到后再发出 B 请求。管道机制则是允许浏览器同时发出 A 请求和 B 请求，但是服务器还是按照顺序，先回应 A 请求，完成后再回应 B 请求。

只有幂等的请求（GET，HEAD）能使用 pipelining，非幂等请求比如 POST 不能使用，因为请求之间可能会存在先后依赖关系。

### 3.3 Content-Length 字段

一个 TCP 连接现在可以传送多个回应，势必就要有一种机制，区分数据包是属于哪一个回应的。这就是 Content-length 字段的作用，声明本次回应的数据长度。

```
Content-Length: 3495
```

上面代码告诉浏览器，本次回应的长度是 3495 个字节，后面的字节就属于下一个回应了。

在 1.0 版中，Content-Length 字段不是必需的，因为浏览器发现服务器关闭了 TCP 连接，就表明收到的数据包已经全了。

### 3.4 分块传输编码

使用 Content-Length 字段的前提条件是，服务器发送回应之前，必须知道回应的数据长度。

对于一些很耗时的动态操作来说，这意味着，服务器要等到所有操作完成，才能发送数据，显然这样的效率不高。更好的处理方法是，产生一块数据，就发送一块，采用"流模式"（stream）取代"缓存模式"（buffer）。

因此，1.1 版规定可以不使用 Content-Length 字段，而使用"分块传输编码"（chunked transfer encoding）。只要请求或回应的头信息有 Transfer-Encoding 字段，就表明回应将由数量未定的数据块组成。

```
Transfer-Encoding: chunked
```

每个非空的数据块之前，会有一个 16 进制的数值，表示这个块的长度。最后是一个大小为 0 的块，就表示本次回应的数据发送完了。下面是一个例子。

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

### 3.5 其他功能

1.1 版还新增了许多动词方法：`PUT`、`PATCH`、`HEAD`、 `OPTIONS`、`DELETE`。

另外，客户端请求的头信息新增了 `Host` 字段，用来指定服务器的域名。

`Host: www.example.com`  
有了 Host 字段，就可以将请求发往同一台服务器上的不同网站，为虚拟主机的兴起打下了基础。

### 3.6 缺点

**不支持服务器推送消息**，因此当客户端需要获取通知时，只能通过定时器不断地拉取消息，这无疑浪费大量了带宽和服务器资源。

虽然 1.1 版允许复用 TCP 连接，但是同一个 TCP 连接里面，所有的数据通信是按次序进行的。服务器只有处理完一个回应，才会进行下一个回应。要是前面的回应特别慢，后面就会有许多请求排队等着。这称为"**队头堵塞**"（Head-of-line blocking）。

![](https://s2.loli.net/2022/03/29/nGNHB9SZQRWViPJ.png)

为了避免这个问题，只有两种方法：一是减少请求数，二是同时多开持久连接。这导致了很多的网页优化技巧，比如 文件合并、域名分片等等。

如果 HTTP 协议设计得更好一些，这些额外的工作是可以避免的。

## 四、SPDY - 开拓者

http1.0 和 1.1 虽然存在这么多问题，业界也想出了各种优化的手段，但这些方法手段都是在尝试绕开协议本身的缺陷，都有种治标不治本的感觉。直到 2012 年 google 如一声惊雷提出了 SPDY 的方案，大家才开始从正面看待和解决老版本 http 协议本身的问题，这也直接加速了 http2.0 的诞生。与 HTTP/1.1 相比，主要的改变有：

- 实现无需先入先出的多路复用
- 为简化客户端和服务器开发的消息—帧机制
- 强制性压缩（包括 HTTP 头部）
- 优先级排序
- 双向通讯

### 4.1 SPDY 的目标

SPDY 的目标在一开始就是瞄准 http1.x 的痛点，即延迟和安全性。

如果以降低延迟为目标，应用层的 http 和传输层的 tcp 都是都有调整的空间，不过 tcp 作为更底层协议存在已达数十年之久，其实现已深植全球的网络基础设施当中，如果要动必然伤经动骨，业界响应度必然不高，所以 SPDY 的手术刀对准的是 http。

- 降低延迟，客户端的单连接单请求，server 的 FIFO 响应队列都是延迟的大头。
- http 最初设计都是客户端发起请求，然后 server 响应，server 无法主动 push 内容到客户端。
- 压缩 http header，http1.x 的 header 越来越膨胀。而且由于 http 的无状态特性，header 必须每次 request 都重复携带，很浪费流量。

SPDY 的设计如下：

![](https://s2.loli.net/2022/03/29/vqmDXNwafn6jcBM.png)

SPDY 位于 HTTP 之下，TCP 和 SSL 之上，这样可以轻松兼容老版本的 HTTP 协议(将 http1.x 的内容封装成一种新的 frame 格式)，同时可以使用已有的 SSL 功能。SPDY 的功能可以分为基础功能和高级功能两部分，基础功能默认启用，高级功能需要手动启用。

### 4.2 SPDY 基础功能

- 多路复用（multiplexing）。多路复用通过多个请求 stream 共享一个 tcp 连接的方式，解决了 http1.x holb（head of line blocking）的问题，降低了延迟同时提高了带宽的利用率。
- 请求优先级（request prioritization）。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY 允许给每个 request 设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的 html 内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。
- header 压缩。前面提到过几次 http1.x 的 header 很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。SPDY 对 header 的压缩率可以达到 80%以上，低带宽环境下效果很大。

### 4.3 SPDY 高级功能

- server 推送（server push）。http1.x 只能由客户端发起请求，然后服务器被动的发送 response。开启 server push 之后，server 通过 X-Associated-Content header（X-开头的 header 都属于非标准的，自定义 header）告知客户端会有新的内容推送过来。在用户第一次打开网站首页的时候，server 将资源主动推送过来可以极大的提升用户体验。
- server 暗示（server hint）。和 server push 不同的是，server hint 并不会主动推送内容，只是告诉有新的内容产生，内容的下载还是需要客户端主动发起请求。server hint 通过 X-Subresources header 来通知，一般应用场景是客户端需要先查询 server 状态，然后再下载资源，可以节约一次查询请求。

### 4.4 SPDY 的成绩

SPDY 的成绩可以用 google 官方的一个数字来说明：页面加载时间相比于 http1.x 减少了 64%。而且各大浏览器厂商在 SPDY 诞生之后的 1 年多里都陆续支持了 SPDY，不少大厂 app 和 server 端框架也都将 SPDY 应用到了线上的产品当中。

2015 年 9 月，Google 宣布了移除对 SPDY 支持的计划，拥抱 HTTP/2，并将在 Chrome 51 中生效。

## 五、HTTP/2 - 为了更优异的表现

这些年来，网页愈渐变得的复杂，甚至演变成了独有的应用，可见媒体的播放量，增进交互的脚本大小也增加了许多：更多的数据通过 HTTP 请求被传输。HTTP/1.1 链接需要请求以正确的顺序发送，理论上可以用一些并行的链接（尤其是 5 到 8 个），带来的成本和复杂性堪忧。比如，HTTP 管线化（pipelining）就成为了 Web 开发的负担。

SPDY 的诞生和表现说明了两件事情：一是在现有互联网设施基础和 http 协议广泛使用的前提下，是可以通过修改协议层来优化 http1.x 的。二是针对 http1.x 的修改确实效果明显而且业界反馈很好。正是这两点让 IETF（Internet Enginerring Task Force）开始正式考虑制定 HTTP2.0 的计划，最后决定以 SPDY／3 为蓝图起草 HTTP2.0，SPDY 的部分设计人员也被邀请参与了 HTTP2.0 的设计。

HTTP/2 在 HTTP/1.1 有几处基本的不同:

- HTTP/2 是二进制协议而不是文本协议。不再可读，也不可无障碍的手动创建，改善的优化技术现在可被实施。
- 这是一个复用协议。并行的请求能在同一个链接中处理，移除了 HTTP/1.x 中顺序和阻塞的约束。
- 压缩了 headers。因为 headers 在一系列请求中常常是相似的，其移除了重复和传输重复数据的成本。
- 其允许服务器在客户端缓存中填充数据，通过一个叫服务器推送的机制来提前请求。

### 5.1 新的二进制格式（Binary Format）

http1.x 诞生的时候是明文协议，其格式由三部分组成：start line（request line 或者 status line），header，body。要识别这 3 部分就要做协议解析，http1.x 的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认 0 和 1 的组合。基于这种考虑 http2.0 的协议解析决定采用二进制格式，实现方便且健壮。

http2.0 用 binary 格式定义了一个一个的 frame，和 http1.x 的格式对比如下图：

![](https://s2.loli.net/2022/03/29/rRtDlJVOsEuyQv9.png)

虽然看上去协议的格式和 http1.x 完全不同了，实际上 http2.0 并没有改变 http1.x 的语义，只是把原来 http1.x 的 header 和 body 部分用 frame 重新封装了一层而已。具体的协议关系可以用下图表示：

![](https://s2.loli.net/2022/03/29/nHwMu3Lb8erIy4z.png)

HTTP/2 把响应报文划分成了两个**帧（Frame）**，图中的 HEADERS（首部）和 DATA（消息负载） 是帧的类型，也就是说一条 HTTP 响应，划分成了两个帧来传输，并且采用二进制来编码。

HTTP/2 **二进制帧**的结构如下图：

![](https://s2.loli.net/2022/03/29/smEnjvRSe5Nug71.png)

帧头（Frame Header）很小，只有 9 个字节，帧开头的前 3 个字节表示帧数据（Fream Playload）的**长度**。

帧长度后面的一个字节是表示**帧的类型**，HTTP/2 总共定义了 10 种类型的帧，一般分为**数据帧**和**控制帧**两类，如下表格：

![](https://s2.loli.net/2022/03/29/U6SRzyGXJ5uoYk3.png)

帧类型后面的一个字节是**标志位**，可以保存 8 个标志位，用于携带简单的控制信息。

帧头的最后 4 个字节是**流标识符**（Stream ID），但最高位被保留不用，只有 31 位可以使用，因此流标识符的最大值是 2^31，大约是 21 亿，它的作用是用来标识该 Fream 属于哪个 Stream，接收方可以根据这个信息从乱序的帧里找到相同 Stream ID 的帧，从而有序组装信息。

HTTP/2 通过让所有数据流共用同一个连接，可以更有效地使用 TCP 连接，让高带宽也能真正的服务于 HTTP 的性能提升。

### 5.2 多路复用

http2.0 要解决的一大难题就是多路复用（MultiPlexing）。

我们都知道 HTTP/1.1 的实现是基于请求-响应模型的。同一个连接中，HTTP 完成一个事务（请求与响应），才能处理下一个事务，也就是说在发出请求等待响应的过程中，是没办法做其他事情的，如果响应迟迟不来，那么后续的请求是无法发送的，也造成了**队头阻塞**的问题。

而 HTTP/2 通过 Stream 这个设计，**多个 Stream 复用一条 TCP 连接，达到并发的效果**，解决了 HTTP/1.1 队头阻塞的问题，提高了 HTTP 传输的吞吐量。

![](https://s2.loli.net/2022/03/29/8efVYQuEq76STmB.png)

从上图中可以看到：

- 1 个 TCP 连接包含一个或者多个 Stream，Stream 是 HTTP/2 并发的关键技术；
- Stream 里可以包含 1 个或多个 Message，Message 对应 HTTP/1 中的请求或响应，由 HTTP 头部和包体构成；
- Message 里包含一条或者多个 Frame，Frame 是 HTTP/2 最小单位，以二进制压缩格式存放 HTTP/1 中的内容（头部和包体）；

在 HTTP/2 连接上，**不同 Stream 的帧是可以乱序发送的（因此可以并发不同的 Stream ）**，因为每个帧的头部会携带 Stream ID 信息，所以接收端可以通过 Stream ID 有序组装成 HTTP 消息，而**同一 Stream 内部的帧必须是严格有序的**。

客户端和服务器**双方都可以建立 Stream**， Stream ID 也是有区别的，客户端建立的 Stream 必须是奇数号，而服务器建立的 Stream 必须是偶数号。

HTTP/2 还可以对每个 Stream 设置不同**优先级**，帧头中的「标志位」可以设置优先级，比如客户端访问 HTML/CSS 和图片资源时，希望服务器先传递 HTML/CSS，再传图片，那么就可以通过设置 Stream 的优先级来实现，以此提高用户体验。

### 5.3 header 压缩

http1.x 的 header 由于 cookie 和 user agent 很容易膨胀，而且每次都要重复发送。http2.0 使用 encoder 来减少需要传输的 header 大小，通讯双方各自 cache 一份 header fields 表，既避免了重复 header 的传输，又减小了需要传输的大小。高效的压缩算法可以很大的压缩 header，减少发送包的数量从而降低延迟。

### 5.4 压缩算法的选择

SPDY/2 使用的是 gzip 压缩算法，但后来出现的两种攻击方式 `BREACH` 和`CRIME`使得即使走 ssl 的 SPDY 也可以被破解内容，最后综合考虑采用的是一种叫`HPACK`的压缩算法。HPACK 算法主要包含三个组成部分：

- 静态字典；
- 动态字典；
- Huffman 编码（压缩算法）；

HTTP/2 为高频出现在头部的字符串和字段建立了一张**静态表**，它是写入到 HTTP/2 框架里的，不会变化的，静态表里共有 `61` 组。

客户端和服务器两端都会建立和维护「**字典**」，用长度较小的索引号表示重复的字符串，再用 Huffman 编码压缩数据，**可达到 50%~90% 的高压缩率**。

### 5.5 服务端推送

HTTP/1.1 不支持服务器主动推送资源给客户端，都是由客户端向服务器发起请求后，才能获取到服务器响应的资源。

比如，客户端通过 HTTP/1.1 请求从服务器那获取到了 HTML 文件，而 HTML 可能还需要依赖 CSS 来渲染页面，这时客户端还要再发起获取 CSS 文件的请求，需要两次消息往返，如下图左边部分：![](https://s2.loli.net/2022/03/29/F4jy3JXlC7s5IPi.png)

如上图右边部分，在 HTTP/2 中，客户端在访问 HTML 时，服务器可以直接主动推送 CSS 文件，减少了消息传递的次数。

### 5.6 HTTP/2 的现状

在 2015 年 5 月正式标准化后，HTTP/2 取得了极大的成功。根据 W3Techs 的数据，截至 2021 年 10 月，全球有 46.5%的网站支持了 HTTP/2。

## 六、HTTP/3 - 面向未来

HTTP/2 通过头部压缩、二进制编码、多路复用、服务器推送等新特性大幅度提升了 HTTP/1.1 的性能，而美中不足的是 HTTP/2 协议是基于 TCP 实现的，于是存在的缺陷有三个。

- 队头阻塞；
- TCP 与 TLS 的握手时延迟；
- 网络迁移需要重新连接；

### 6.1.1 队头阻塞

HTTP/2 多个请求是跑在一个 TCP 连接中的，那么当 TCP 丢包时，整个 TCP 都要等待重传，那么就会阻塞该 TCP 连接中的所有请求。

因为 TCP 是字节流协议，TCP 层必须保证收到的字节数据是完整且有序的，如果序列号较低的 TCP 段在网络传输中丢失了，即使序列号较高的 TCP 段已经被接收了，应用层也无法从内核中读取到这部分数据，从 HTTP 视角看，就是请求被阻塞了。

### 6.1.2 TCP 与 TLS 的握手时延迟

发起 HTTP 请求时，需要经过 TCP 三次握手和 TLS 四次握手（TLS 1.2）的过程，因此共需要 3 个 RTT 的时延才能发出请求数据。

![](https://s2.loli.net/2022/03/29/KwdmhePqVtA2L9j.png)

### 6.1.3 网络迁移需要重新连接

一个 TCP 连接是由四元组（源 IP 地址，源端口，目标 IP 地址，目标端口）确定的，这意味着如果 IP 地址或者端口变动了，就会导致需要 TCP 与 TLS 重新握手，这不利于移动设备切换网络的场景，比如 4G 网络环境切换成 WIFI。

这些问题都是 TCP 协议固有的问题，无论应用层的 HTTP/2 在怎么设计都无法逃脱。要解决这个问题，就必须把**传输层协议替换成 UDP**，这个大胆的决定，HTTP/3 做了！

### 6.2 QUIC 协议

我们深知，UDP 是一个简单、不可靠的传输协议，而且是 UDP 包之间是无序的，也没有依赖关系。

而且，UDP 是不需要连接的，也就不需要握手和挥手的过程，所以天然的就比 TCP 快。

当然，HTTP/3 不仅仅只是简单将传输协议替换成了 UDP，还基于 UDP 协议在「应用层」实现了 **QUIC 协议**，它具有类似 TCP 的连接管理、拥塞窗口、流量控制的网络特性，相当于将不可靠传输的 UDP 协议变成“可靠”的了，所以不用担心数据包丢失的问题。

QUIC 协议的优点有很多，这里举例几个，比如：

- 无队头阻塞；
- 更快的连接建立；
- 连接迁移；

### 6.2.1 无队头阻塞

QUIC 协议也有类似 HTTP/2 Stream 与多路复用的概念，也是可以在同一条连接上并发传输多个 Stream，Stream 可以认为就是一条 HTTP 请求。

由于 QUIC 使用的传输协议是 UDP，UDP 不关心数据包的顺序，如果数据包丢失，UDP 也不关心。

不过 QUIC 协议会保证数据包的可靠性，每个数据包都有一个序号唯一标识。当某个流中的一个数据包丢失了，即使该流的其他数据包到达了，数据也无法被 HTTP/3 读取，直到 QUIC 重传丢失的报文，数据才会交给 HTTP/3。

而其他流的数据报文只要被完整接收，HTTP/3 就可以读取到数据。这与 HTTP/2 不同，HTTP/2 只要某个流中的数据包丢失了，其他流也会因此受影响。

所以，QUIC 连接上的多个 Stream 之间并没有依赖，都是独立的，某个流发生丢包了，只会影响该流，其他流不受影响。

![](https://s2.loli.net/2022/03/29/82HVWhGA6kzNSCP.png)

### 6.2.2 更快的连接建立

对于 HTTP/1 和 HTTP/2 协议，TCP 和 TLS 是分层的，分别属于内核实现的传输层、openssl 库实现的表示层，因此它们难以合并在一起，需要分批次来握手，先 TCP 握手，再 TLS 握手。

HTTP/3 在传输数据前虽然需要 QUIC 协议握手，这个握手过程只需要 1 RTT，握手的目的是为确认双方的「连接 ID」，连接迁移就是基于连接 ID 实现的。

但是 HTTP/3 的 QUIC 协议并不是与 TLS 分层，而是**QUIC 内部包含了 TLS，它在自己的帧会携带 TLS 里的“记录”，再加上 QUIC 使用的是 TLS1.3，因此仅需 1 个 RTT 就可以「同时」完成建立连接与密钥协商，甚至在第二次连接的时候，应用数据包可以和 QUIC 握手信息（连接信息 + TLS 信息）一起发送，达到 0-RTT 的效果**。

![](https://s2.loli.net/2022/03/29/vdRrNy8QaTZk7C2.png)

### 6.2.3 连接迁移

前面提到，基于 TCP 传输协议的 HTTP 协议，由于是通过四元组（源 IP、源端口、目的 IP、目的端口）确定一条 TCP 连接，那么当移动设备的网络从 4G 切换到 WIFI 时，意味着 IP 地址变化了，那么就必须要断开连接，然后重新建立连接，而建立连接的过程包含 TCP 三次握手和 TLS 四次握手的时延，以及 TCP 慢启动的减速过程，给用户的感觉就是网络突然卡顿了一下，因此连接的迁移成本是很高的。

而 QUIC 协议没有用四元组的方式来“绑定”连接，而是通过**连接 ID**来标记通信的两个端点，客户端和服务器可以各自选择一组 ID 来标记自己，因此即使移动设备的网络变化后，导致 IP 地址变化了，只要仍保有上下文信息（比如连接 ID、TLS 密钥等），就可以“无缝”地复用原连接，消除重连的成本，没有丝毫卡顿感，达到了**连接迁移**的功能。

### 6.3 更简单的二进制帧结构

HTTP/3 同 HTTP/2 一样采用二进制帧的结构，不同的地方在于 HTTP/2 的二进制帧里需要定义 Stream，而 HTTP/3 自身不需要再定义 Stream，直接使用 QUIC 里的 Stream，于是 HTTP/3 的帧的结构也变简单了。

![](https://s2.loli.net/2022/03/29/ICY1abUDOiduorW.png)

从上图可以看到，HTTP/3 帧头只有两个字段：类型和长度。根据帧类型的不同，大体上分为数据帧和控制帧两大类，HEADERS 帧（HTTP 头部）和 DATA 帧（HTTP 包体）属于数据帧。

### 6.4 压缩算法升级

HTTP/3 在头部压缩算法这一方便也做了升级，升级成了 **QPACK**。与 HTTP/2 中的 HPACK 编码方式相似，HTTP/3 中的 QPACK 也采用了静态表、动态表及 Huffman 编码。

对于静态表的变化，HTTP/2 中的 HPACK 的静态表只有 61 项，而 HTTP/3 中的 QPACK 的静态表扩大到 91 项。

HTTP/2 和 HTTP/3 的 Huffman 编码并没有多大不同，但是动态表编解码方式不同。

所谓的动态表，在首次请求-响应后，双方会将未包含在静态表中的 Header 项更新各自的动态表，接着后续传输时仅用 1 个数字表示，然后对方可以根据这 1 个数字从动态表查到对应的数据，就不必每次都传输长长的数据，大大提升了编码效率。

可以看到，**动态表是具有时序性的，如果首次出现的请求发生了丢包，后续的收到请求，对方就无法解码出 HPACK 头部，因为对方还没建立好动态表，因此后续的请求解码会阻塞到首次请求中丢失的数据包重传过来**。

QUIC 会有两个特殊的单向流，所谓的单向流只有一端可以发送消息，双向则指两端都可以发送消息，传输 HTTP 消息时用的是双向流，这两个单向流的用法：

- 一个叫 QPACK Encoder Stream， 用于将一个字典（key-value）传递给对方，比如面对不属于静态表的 HTTP 请求头部，客户端可以通过这个 Stream 发送字典；
- 一个叫 QPACK Decoder Stream，用于响应对方，告诉它刚发的字典已经更新到自己的本地动态表了，后续就可以使用这个字典来编码了。

这两个特殊的单向流是用来**同步双方的动态表**，编码方收到解码方更新确认的通知后，才使用动态表编码 HTTP 头部。

### 6.5 HTTP/3 的现状

2019 年 9 月，HTTP/3 支持已添加到 Cloudflare 和 Google Chrome（Canary build）。Firefox Nightly 在 2019 年秋季之后添加支持。

截至 2021 年 6 月，HTTP/3 仍然是草案状态。

> 计网小作业，大部分是转载
