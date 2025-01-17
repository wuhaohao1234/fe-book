## 有意思的 14k
在日常性能优化会发现一个有意思的现象，如果 HTML gzip 后 14k 的话会比 15k 快很多，而 15k 的 HTML 却和 15k 没有太大区别，这个奇怪的 14k 是怎么回事？

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1734183439200-03ffcd84-0250-4bd6-9a78-1d588d077475.png)

我们知道 HTTP/3 之前网页内容的传输依赖 TCP 协议，最初的 TCP 的实现方式是在连接建立成功后便会向网络中发送大尺寸的数据包，假如网络出现问题，很多这样的大包会积攒在路由器上，很容易导致网络中路由器缓存空间耗尽，从而发生拥塞

## TCP 慢启动
为了有效管理网络拥塞，TCP 使用了一套称为拥塞控制的机制，其中之一就是慢启动（Slow Start）。在慢启动的策略下，新建立的 TCP 连接不能一开始就发送大尺寸的数据包，而只能从一个小尺寸的包开始发送，在发送和数据被对方确认的过程中去计算对方的接收速度，来逐步增加每次发送的数据量，最后到达一个稳定的值，进入高速传输阶段

准确一些来描述，慢启动的基本流程是这样的：

1. 初始拥塞窗口（Initial Congestion Window, IW）：TCP 连接开始时，发送方设定一个较小的拥塞窗口
2. 指数增长：每当接收到一个往返时间（RTT）内的确认（ACK）时，拥塞窗口增加一个 MSS，使得发送的数据量呈指数级增长
3. 阈值（ssthresh）：当拥塞窗口增长到一定阈值时，慢启动阶段结束，进入拥塞避免阶段，此时拥塞窗口的增长转为线性

## 为什么是 14k
根据 [RFC 6928](https://tools.ietf.org/html/rfc6928) 的建议，当前推荐的初始拥塞窗口大小是 10 个 MSS（Maximum Segment Size） ，在常见的网络环境中 MSS 通常为 1460 字节，因此初始拥塞窗口的大小约为：

IW = 10×1460 字节 = 14,600 字节 (14KB)

如果初始 HTML 文件的大小不超过 14KB，那么整个文件可以在初始阻塞窗口（IW）内被完整传输，这样一来 HTML 文件的传输不会受到慢启动带来的带宽限制，能够更快地到达浏览器，从而提升用户体验

:::success
如果使用了流式渲染，只要保证 Pipe 给浏览器第一段 HTML 小于 14k 也可以避免 TCP 慢启动问题

:::

## HTTP/3 还有慢启动限制吗
虽然 HTTP/3 使用基于 UDP 实现的 QUIC 作为其传输层协议，但 QUIC 设计之初就集成了拥塞控制机制，因此 QUIC 仍然采用类似于 TCP 慢启动的机制来管理数据传输

1. **<font style="color:rgb(51, 51, 51);">初始拥塞窗口</font>**<font style="color:rgb(51, 51, 51);">：QUIC 使用初始拥塞窗口来限制初始数据传输量，通常也为约 10 个 MSS（约 14KB），与 TCP 相似</font>
2. **<font style="color:rgb(51, 51, 51);">指数增长</font>**<font style="color:rgb(51, 51, 51);">：在连接初期，QUIC 的拥塞窗口会随着收到的 ACK 不断增加，呈指数级增长，直到达到阈值</font>
3. **<font style="color:rgb(51, 51, 51);">阈值管理</font>**<font style="color:rgb(51, 51, 51);">：一旦拥塞窗口达到预设的阈值，QUIC 转入拥塞避免阶段，采用线性增长策略</font>

HTTP/3 使用 QUIC 协议带来了诸多性能优化，包括更快的连接建立、更好的多路复用和更灵活的拥塞控制。然而慢启动作为拥塞控制的一部分仍然存在于 QUIC 中

所以即使网站升级了 HTTP/3 后，仍旧需要将初始 HTML 控制在 14KB 以下，确保整个文件可以在 QUIC 的慢启动阶段内被完整传输，最大限度地利用初始拥塞窗口，缩短从请求到首次渲染的时间（Time to First Byte, TTFB）

