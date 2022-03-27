**WebSocket**是一种[网络传输协议](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE "网络传输协议")，可在单个[TCP](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE "传输控制协议")连接上进行[全双工](https://zh.wikipedia.org/wiki/%E5%85%A8%E9%9B%99%E5%B7%A5 "全双工")通信，位于[OSI模型](https://zh.wikipedia.org/wiki/OSI%E6%A8%A1%E5%9E%8B "OSI模型")的[应用层](https://zh.wikipedia.org/wiki/%E5%BA%94%E7%94%A8%E5%B1%82 "应用层")。

默认情况下，Websocket协议使用80端口；运行在TLS之上时，默认使用443端口。

## 握手协议
WebSocket 是独立的、创建在TCP上的协议。

Websocket 通过 HTTP/1.1 协议的101状态码进行握手。

为了创建Websocket连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（Handshaking）。

### 客户端请求
```http
GET /chat HTTP/1.1
Host: server.example.com
Origin: http://example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```


### 服务端回应
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```


-   Connection必须设置Upgrade，表示客户端希望连接升级。
-   Upgrade字段必须设置Websocket，表示希望升级到Websocket协议。
-   Sec-WebSocket-Key是随机的字符串，服务器端会用这些数据来构造出一个SHA-1的信息摘要。把“Sec-WebSocket-Key”加上一个特殊字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，然后计算[SHA-1](https://zh.wikipedia.org/wiki/SHA-1 "SHA-1")摘要，之后进行[Base64](https://zh.wikipedia.org/wiki/Base64 "Base64")编码，将结果做为“Sec-WebSocket-Accept”头的值，返回给客户端。如此操作，可以尽量避免普通HTTP请求被误认为Websocket协议。
-   Sec-WebSocket-Version 表示支持的Websocket版本。RFC6455要求使用的版本是13，之前草案的版本均应当弃用。
-   Origin字段是必须的。如果缺少origin字段，WebSocket服务器需要回复HTTP 403 状态码（禁止访问）。[\[16\]](https://zh.wikipedia.org/wiki/WebSocket#cite_note-16)
-   其他一些定义在HTTP协议中的字段，如[Cookie](https://zh.wikipedia.org/wiki/Cookie "Cookie")等，也可以在Websocket中使用。

## API


## 优点
-   较少的控制开销。在连接创建后，服务器和客户端之间交换数据时，用于协议控制的数据包头部相对较小。在不包含扩展的情况下，对于服务器到客户端的内容，此头部大小只有2至10[字节](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82 "字节")（和数据包长度有关）；对于客户端到服务器的内容，此头部还需要加上额外的4字节的[掩码](https://zh.wikipedia.org/wiki/%E6%8E%A9%E7%A0%81 "掩码")。相对于HTTP请求每次都要携带完整的头部，此项开销显著减少了。

-   更强的实时性。由于协议是全双工的，所以服务器可以随时主动给客户端下发数据。相对于HTTP请求需要等待客户端发起请求服务端才能响应，延迟明显更少；即使是和Comet等类似的[长轮询](https://zh.wikipedia.org/w/index.php?title=%E9%95%BF%E8%BD%AE%E8%AF%A2&action=edit&redlink=1 "长轮询（页面不存在）")比较，其也能在短时间内更多次地传递数据。

-   保持连接状态。与HTTP不同的是，Websocket需要先创建连接，这就使得其成为一种有状态的协议，之后通信时可以省略部分状态信息。而HTTP请求可能需要在每个请求都携带状态信息（如身份认证等）。

-   更好的二进制支持。Websocket定义了[二进制](https://zh.wikipedia.org/wiki/%E4%BA%8C%E8%BF%9B%E5%88%B6 "二进制")帧，相对HTTP，可以更轻松地处理二进制内容。

-   可以支持扩展。Websocket定义了扩展，用户可以扩展协议、实现部分自定义的子协议。如部分浏览器支持[压缩](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9 "数据压缩")等。

-   更好的压缩效果。相对于[HTTP压缩](https://zh.wikipedia.org/wiki/HTTP%E5%8E%8B%E7%BC%A9 "HTTP压缩")，Websocket在适当的扩展支持下，可以沿用之前内容的[上下文](https://zh.wikipedia.org/wiki/%E4%B8%8A%E4%B8%8B%E6%96%87)，在传递类似的数据时，可以显著地提高压缩率。[\[14\]](https://zh.wikipedia.org/wiki/WebSocket#cite_note-14)