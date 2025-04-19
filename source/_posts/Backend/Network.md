---
title: Network
categories:
  - Back-End
  - Network
---

### 1. Open System Interconnect OSI

每层实现独立功能 & 协议, 和邻层使用接口通信, 互不打扰

- Application layer (message)
  - interface btw apps & network (how app communicate over network)
  - Provide network over protocols (HTTP, 域名系统: DNS, 电子邮件系统: SMTP...)
- Presentation layer (message)
  - Data translation, encryption, compression: data from App layer can be understood by other layer (JPEG, ASCII, SSL/TLS)
- Session layer (message)
  - Manage sessions btw apps
  - Dialog control & Synchronization: 提供数据交换的定界 & 同步功能, 包括建立检查点 & 恢复方案的方法
  - 视频通话: 建立和管理会话控制(视频会议软件的会话)
- Transport layer (segment - TCP / datagram - UDP)
  - End-to-End btw hosts 端到端通信 & 端口号机制
  - 附加信息: 用于判断 DATA 是否在运输途中被改变
- Network layer (Packet, Include source & target address...)
  - Route & address data btw hosts: 确保数据按时成果传送, 选择合适的网间路由和交换节点
  - 常见协议: IP (IPv4, IPv6)
- Data Link layer (Frame)
  - head
    - 发送者, 接受者, 数据类型, 包含发送&接收端 MAC 地址
  - data
    - 交互的数据
  - Ensure reliable data transfer btw 2 connected nodes
- Physical layer (Bits)

#### 1.1 TCP/IP 协议簇 TCP/IP Protocol Suite

由 `FTP`, `SMTP`, `TCP`, `UDP`, `IP`等协议构成

- 层次结构: 五层体系
  - 没有对网络接口层进行细分, 简化后的 OSI
  - 应用层: 合并 OSI 中会话层, 表示层, 应用层, 最终产出应用数据包
    - 为不同应用层协议提供服务
    - `FTP`, `DNS`, `SMTP`...
  - 其他层不变
- 可面向连接和无连接

### 2. 传输层: TCP & UDP 协议

|            | TCP                                                      | UDP                                                                                   |
| ---------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 可靠性     | 流量控制, 编号, 雪人, 计时器保证无差错, 不丢包           | ❌                                                                                    |
| 连接性     | 面向连接, 三次握手四次挥手                               | ❌                                                                                    |
| 报文       | 面向字节流, 拆分报文成多个 TCP 报文传输. 在目的站组装    | 面向报文, 不拆分, 一次全发                                                            |
| 效率       | 低                                                       | 高                                                                                    |
| 流量       | 滑动窗口                                                 | ❌                                                                                    |
| 拥塞       | 慢开始, 拥塞避免, 快重传, 快回复                         | ❌                                                                                    |
| 丢包       | 重发                                                     | ❌                                                                                    |
| 乱序       | 顺序控制                                                 | ❌                                                                                    |
| 双共性     | 全双工                                                   | 1-1, 1-多, 多-1, 多-多                                                                |
| 其他       | 报文首部有 20 字节, 额外开销大                           | 不提供复杂控制机制, 利用 IP 面向无连接的通信服务                                      |
| 应用层协议 | SMTP 电子邮件, HTTP, FTP 文件传输 (效率要求低, 准确性高) | DNS 域名转换, TFTP 文件传输, SNMP 网络管理, NFS 远程文件服务器 (效率要求高, 准确性低) |

#### 2.1 TCP 三次握手 & 四次挥手

### 3. HTTP & HTTPS

- HTTP: plain text w/o encryption, 多次握手, 性能慢, Port 80
  - 无连接: 1 request per connection. Disconnect after request-receive. 节省传输时间
  - 无状态: Can't process request according to previous state

#### 3.2 HTTPS

- `HTTPS = HTTP + SSL/TLS`
  - HTTP 协议的安全版本
  - 允许 HTTP 在安全的`SSL/TLS`协议上: Secure Sockets Layer, Transport Layer Security
  - SSL/TLS certificate issued by trusted Certificate Authority, cost expensive, 功能越强, 价格越高, Port 443
- `SSL`: 验证服务器身份, 加密通信
  - 位于`TCP/IP`协议与各种应用层协议之间
- HTTPS Handshake (SSL / TLS)
  1.  Client Hello: URL 访问, 建立 SSL 链接
  2.  Server Hello: 发送证书信息 & Public key
  3.  Certificate Verification: Client validates server certificate via CA
  4.  Key exchange: Client generate session key, encrypt it with server's public key
  5.  Session Establish: Server uses private key to decrypt session key
  6.  Secure Communication Begins

#### 3.3 SSL

- Symmetric Encryption: 协商后的密钥加密数据
  - 一个密钥进行加密&解密
- Asymmetric Encryption: 身份认证 & 密钥协商
  - 公钥: 加密后只能用私钥解
  - 私钥: 加密后只能用公钥解
- Hybrid Encryption: 对称+非对称加密
  1.  发送密文的一方使用对方的公钥进行加密处理”对称的密钥“
  2.  对方用自己的私钥解密拿到”对称的密钥“
  - (数据仍然会被黑客解决)
- Hash / Digest Algorithm 摘要算法: 验证信息的完整性
  - 散列函数, hash 函数
  - 保证“数字摘要” equivalent to 原文
  - 在原文后附上它的摘要, 就可以保证数据完整性
- Digital Signature 数字签名: 身份验证
  - 确定消息由发送方签名并发出 (无法被冒充)
  - 签名公开, 私钥加密, 公钥解密
  - 签名只有用私钥对应的公钥才能解开
  - 需要第三方 - 证书验证机构 (数字证书)

### 4. DNS Domain Names System

- 翻译官: 域名和对应 IP 地址转换的服务器
- `三级域名.二级域名.顶级域名`: `www.xxx.com`
- Recursive query 递归查询: A 问 B, B 一定要给 A 想要的答案
- Iterative query 迭代查询: 如果 B 没有答案, B 告诉 A 如何获得
- 域名缓存
  - 浏览器缓存: 浏览器对获取的 IP 进行缓存
  - OS 缓存: `host` 文件

#### 4.1 访问步骤

寻找`www.baidu.com`的 IP 地址

1. local check: 搜索本地 DNS 缓存 (browser & OS DNS cache)
2. 若没有命中, OS asks **local DNS server** (Recursive query)
   - 若没有命中, (iterative query: Root -> TLD -> Authoritative DNS)
     1. Local -> Root: get TLD address (`.com`域名由`xxx`管理, 你去问问它)
     2. Local -> TLD: get Authoritative DNS server address(负责`baidu.com`的是`yyy`, 你去问它)
     3. Local -> Authoritative: 找到有`baidu.com`权限域名服务器, 得到 IP 地址
3. cache IP at all levels (DNS, OS, browser)
4. Browser uses IP to connect to server

### 5. GET & POST

本质都是 TCP 链接 (传输层协议), 有区别是因为 HTTP 规定 & 浏览器/服务器的限制

|                                     | GET                               | POST                                                             |
| ----------------------------------- | --------------------------------- | ---------------------------------------------------------------- |
| 用途                                | 请求指定资源, 只能用于获取数据    | 将实体提交到指定资源, 导致服务器状态变化                         |
| 浏览器回退                          | 无害                              | 再次请求                                                         |
| URL bookmark                        | 可以                              | 不可以                                                           |
| 浏览器主动 cache                    | 会                                | 需要手动                                                         |
| URL                                 | url 编码                          | 支持多种编码方式                                                 |
| 请求参数                            | 被保留在浏览器历史记录            | 不会被保留                                                       |
| URL 传送参数长度                    | 有限制                            | 无限制                                                           |
| 参数数据类型                        | ASCII                             | 无限制                                                           |
| 安全性 (传输上都不安全, 需要 HTTPS) | 不安全, 参数暴露在 URL            | 相对安全                                                         |
| 参数位置                            | URL                               | request body                                                     |
| 数据请求                            | 发送 http header & data, 响应 200 | 先发 head, 响应 100; 再发 data, 响应 200 (not always send twice) |

### 6. HTTP Status code

- 1xx - 临时响应: 请求已被接收, 需要继续处理
- 2xx - 请求已被服务器接收, 理解, 接受
- 3xx - 重定向: 完成请求需要进一步操作
- 4xx - 客户端错误, 妨碍服务器处理
- 5xx - 服务器无法完成请求, 服务器错误 / 异常

### 7. Web Socket

网络传输协议, 位于应用层

- 节省服务器资源和带宽&实时通讯: 在单个 TCP 连接上进行全双工通信
- 一次握手
  - 创建持久性连接 & 双向数据传输
- 出现之前用的是轮询
  - 不断发送 HTTP 请求 (有无数据, 有的话就回应)
  - 消耗大量带宽 & CPU 资源

#### 7.1 特点

- 全双工: 数据在 A 和 B 可以同时传输
- 二进制:
  - 侧重实时通信, 和 HTTP/2 完全不同, 不存在多路复用, 优先级...
  - 不需要服务器推送
- 协议名
  - `ws`, `wss`代表明文&密文的`websocket`协议
  - `ws://www.xxx.com:443/...`
  - `wss://www.xxx.com:80/...`

### 8. CDN Content Delivery Network

- Speed
  - 根据用户位置分配最近的资源, 上网访问的是最近的 CDN 节点 (边缘节点), 其缓存了源站内容的代理服务器
- w/o CDN:
  - 提交 domain -> 浏览器解释 domain -> DNS 解析得到 target 主机 IP 地址 -> 根据 IP 地址访问发出请求 -> 收到数据并回复
- w/ CDN:
  - DNS 返回 CNAME, CDN 通过 CNAME 找到边缘服务器 (as a proxy)
- CNAME: Canonical Name, 向 CDN 的全局负载均衡
  - 由域名的权威 DNS (Authoritative DNS) 配置并返回
    - 比如 Cloud-flare DNS, AWS Route 53
  - 在域名解析中作为代理

#### 8.1 负载均衡系统

- 由于没有返回 IP 地址, 本地 DNS 向负载均衡系统再次发送请求, 进入到 CDN 的全局负载均衡系统进行智能调度

1. Browser -> Local DNS server for domain (`www.xxx.com`)
2. Local DNS -> Authoritative DNS for CNAME (`cdn.xxx.net`)
3. Local DNS -> CDN's DNS for actual IP (optimal edge server IP) using CNAME
   - Intelligent scheduling: 健康状况, 服务能力, 响应时间...
4. Browser -> CDN edge server IP by HTTP request for content delivery

#### 8.2 缓存代理

- 缓存系统
  - Parent cache layer 一级节点: 直连源站, 缓存配置高
  - Edge cache layer 二级节点: 直连用户, 缓存配置低
- Cache hit ratio 命中率: 命中缓存的概率
- Origin fetch rate 回源率: 用代理的方式回源站取的概率
  - if 二级缓存, else if 一级缓存, else 回源站
