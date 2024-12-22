HTTP 协议有几个与安全相关的重要机制：

1. HTTPS (HTTP Secure)
    - 使用 SSL/TLS 加密 HTTP 通信
    - 提供数据加密、身份认证和完整性保护
2. HTTP Strict Transport Security (HSTS)
    - 强制客户端使用 HTTPS 与服务器创建连接
    - 防止 SSL 剥离攻击
3. Content Security Policy (CSP)
    - 限制资源加载来源，防止 XSS 攻击
    - 可以禁用内联脚本，只允许加载指定域名的资源
4. X-Frame-Options
    - 防止网站被嵌入到 iframe 中，避免点击劫持攻击
5. X-XSS-Protection
    - 启用浏览器内置的 XSS 防护机制
6. HTTP Public Key Pinning (HPKP)
    - 防止中间人攻击和 SSL 证书劫持
7. Secure Cookies
    - 标记 Cookie 为 Secure，只通过 HTTPS 发送
    - 使用 HttpOnly 标记防止 JavaScript 访问 Cookie
8. Cross-Origin Resource Sharing (CORS)
    - 控制跨域资源访问，增强同源策略
9. Referrer Policy
    - 控制 HTTP Referer 头的发送策略，保护用户隐私
10. Subresource Integrity (SRI)
    - 验证从第三方 CDN 加载的资源完整性

这些机制共同构建了 HTTP 的多层安全防护，但需要正确配置和使用才能发挥作用。

