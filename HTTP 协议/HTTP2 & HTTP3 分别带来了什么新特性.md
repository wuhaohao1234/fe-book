<font style="color:rgb(51, 51, 51);">HTTP/2 和 HTTP/3 是对 HTTP 协议的重大改进版本，分别引入了许多新特性，以改善网络传输效率、延迟和安全性</font>

### <font style="color:rgb(51, 51, 51);">HTTP/2 的新特性</font>
1. **二进制分帧：**<font style="color:rgb(51, 51, 51);">HTTP/2 使用二进制格式而不是文本格式进行消息编码，这提高了解析和传输的效率</font>
2. **多路复用（Multiplexing）**：<font style="color:rgb(51, 51, 51);">在单个 TCP 连接中同时发送多个请求和响应，而不是像 HTTP/1.1 那样需要排队和阻塞，这种方式减少了延迟和改进了带宽利用率</font>
3. **流优先级和依赖性（Stream Prioritization and Dependency）**：<font style="color:rgb(51, 51, 51);">客户端可以对各个流进行优先级排序，允许更重要的资源先被处理</font>
4. **头部压缩（Header Compression）**：<font style="color:rgb(51, 51, 51);">使用 HPACK 算法压缩 HTTP 头部，这减少了冗余数据的传输，提高了网络效率</font>
5. **服务器推送（Server Push）**：<font style="color:rgb(51, 51, 51);">服务器可以在客户端请求资源之前主动推送额外的资源，从而减少延迟。例如，服务器可以在请求 HTML 页面时主动推送 CSS 文件</font>

### <font style="color:rgb(51, 51, 51);">HTTP/3 的新特性</font>
1. **基于 QUIC 协议**：<font style="color:rgb(51, 51, 51);">HTTP/3 使用 QUIC 协议作为传输层，而不是 TCP。QUIC 是基于 UDP 构建的，旨在解决 TCP 的一些问题，比如延迟敏感性和连接重建</font>
2. **更快的连接建立**：<font style="color:rgb(51, 51, 51);">QUIC 结合了 TCP 和 TLS 的握手过程，使得连接建立速度更快，减少了初始连接的延迟</font>
3. **消除了队头阻塞（Head-of-Line Blocking）**：<font style="color:rgb(51, 51, 51);">在 TCP 上的 HTTP/2，多路复用虽然改善了队头阻塞问题，但仍然依赖于单一 TCP 连接的可靠性，任何数据包丢失都会阻塞整个连接上的所有流，使用 QUIC 丢失的包只会阻塞其对应的流</font>
4. **更强的安全特性**：<font style="color:rgb(51, 51, 51);">QUIC 内置了 TLS 1.3 加密，提供切换封包的保护，并消除了一些中间人攻击的可能性</font>
5. **改进的迁移和恢复**：<font style="color:rgb(51, 51, 51);">QUIC 可以更快速地处理网络变化，如从 Wi-Fi 到移动网络的切换，改善连接的稳定性和继续性</font>

### <font style="color:rgb(51, 51, 51);">小结</font>
+ **<font style="color:rgb(51, 51, 51);">HTTP/2</font>**<font style="color:rgb(51, 51, 51);"> 专注于提高传输效率，通过多路复用和头部压缩减少延迟</font>
+ **<font style="color:rgb(51, 51, 51);">HTTP/3</font>**<font style="color:rgb(51, 51, 51);"> 则引入了基于 QUIC 的传输层，实现更佳的连接管理、恢复机制和安全性，提高了性能和可靠性</font>

<font style="color:rgb(51, 51, 51);">这两种版本的改进都大幅优化了网络的传输性能，特别是在高延迟的环境下，为现代 Web 应用提供了更高效和流畅的体验。它们在降低延迟和提高吞吐量方面的工作，显著提升了用户的访问速度和整体浏览体验</font>



> [HTTP3为什么抛弃了经典的TCP，转而拥抱 QUIC 呢](https://juejin.cn/post/7384266820466180148)
>



