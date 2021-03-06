---
title: http1
date: 2017-07-09 15:24:30
tags:
    - HTTP
---
# Http 基础知识（一）
Web 使用一种名为 HTTP (HyperText Transfer Protocol，超文本传输协议)的协议作为规范，完成从客户端到服务器端等一系列运作流程。

<!--more-->

## TCP/IP 协议族
HTTP 协议属于 TCP/IP 内部的一个子集。
把与互联网相关联的协议集合起来总称为 TCP/IP。也有说 法认为，TCP/IP 是指 TCP 和 IP 这两种协议。还有一种说法认为 ，TCP/IP 是在 IP 协议的通信过程中，使用到的协议族的统称。
TCP/IP 协议族按层次分别分为以下 4 层:
- 应用层
- 传输层
- 网络层
- 数据链路层

### 应用层
应用层决定了向用户提供应用服务时通信的活动。
TCP/IP 协议族内预存了各类通用的应用服务。比如，FTP(File Transfer Protocol，文件传输协议)和 DNS(Domain Name System，域名系统)服务就是其中两类。
HTTP 协议也处于该层。

### 传输层
传输层为应用层提供处于网络连接中的两台计算机之间的数据传输服务。
在传输层有两个性质不同的协议:TCP(Transmission Control Pr otocol，传输控制协议)和 UDP(User Data Protocol，用户数据报协议)。

### 网络层
网络层用来处理在网络上流动的数据包。
数据包是网络传输的最小数据单位。该层规定了通过怎样的路径(所谓的传输路线)到达对方计算机，并把数据包传送给对方。

### 数据链路层
用来处理连接网络的硬件部分。
包括控制操作系统、硬件的设备 驱动、NIC(Network Interface Card，网络适配器，即网卡)， 及光纤等物理可见部分(还包括连接器等一切传输媒介)。硬件上的范畴均在链路层的作用范围之内。
![](https://ws3.sinaimg.cn/large/006tKfTcly1fhdofz0s1kj31ca10owg6.jpg)
一个例子：
1. 首先作为发送端的客户端在应用层( HTTP 协议)发出一个想看某个 Web 页面的 HTTP 请求。
2. 接着，为了传输方便，在传输层(TCP 协议)把从应用层处收到 的数据(HTTP 请求报文)进行分割，并在各个报文上打上标记 序号及端口号后转发给网络层。
3. 在网络层(IP 协议)，增加作为通信目的地的 MAC 地址后转发 给链路层。这样一来，发往网络的通信请求就准备齐全了。
4. 接收端的服务器在链路层接收到数据，按序往上层发送，一直到 应用层。当传输到应用层，才能算真正接收到由客户端发送过来 的 HTTP 请求。
![](https://ws4.sinaimg.cn/large/006tKfTcly1fhdol78xu0j30z40v0dhv.jpg)
发送端在层与层之间传输数据时，每经过一层必定会打上一个该层所属的首部信息，反之，接收端在层与层传输数据时，每经过一层会把对应的首部消去。
这种把数据信息包装起来的方法称为封装。

## 与 HTTP 关系密切的协议：IP、TCP、DNS

### 负责传输的 IP 协议
按层次分，IP(Internet Protocol)网际协议位于网络层。
IP 协议的作用是把各种数据包传送给对方。而要保证确实传送到 对方那里，则需要满足各类条件。其中两个重要的条件是 IP 地址 和 MAC 地址(Media Access Control Address)。
**IP 地址**指明了节点被分配到的地址，**MAC 地址**是指网卡所属的 固定地址。IP 地址可以和 MAC 地址进行配对。IP 地址可变换， 但 MAC 地址基本上不会更改。

#### 使用 ARP 协议凭借 MAC 地址进行通信
IP 间的通信依赖 MAC 地址。在网络上，通信的双方在同一局域 网(LAN)内的情况是很少的，通常是经过多台计算机和网络设 备中转才能连接到对方。而在进行中转时，会利用下一站中转设 备的 MAC 地址来搜索下一个中转目标。这时，会采用 **ARP 协议** (Address Resolution Protocol)。ARP 是一种用以解析地址的 协议，根据通信方的 IP 地址就可以反查出对应的 MAC 地址。

### 确保可靠的 TCP 协议
TCP 位于传输层，提供可靠的字节流服务
所谓的字节流服务(Byte Stream Service)是指，为了方便传输 ，将大块数据分割成以报文段(segment)为单位的数据包进行 管理。而可靠的传输服务是指，能够把数据准确可靠地传给对方 。一言以蔽之，TCP 协议为了更容易传送大数据才把数据分割， 而且 TCP 协议能够确认数据最终是否送达到对方。

#### TCP 建立连接三次握手
1. 首先Client端发送连接请求报文
2. Server段接受连接后回复ACK报文，并为这次连接分配资源
3. Client端接收到ACK报文后也向Server段发生ACK报文，并分配资源，这样TCP连接就建立了。

#### TCP 断开连接四次挥手
1. Client端发起中断连接请求，也就是发送FIN报文。(我Client端没有数据要发给你了)
2. Server 端发送 ACK。（告诉Client端，你的请求我收到了，但是我还没准备好，请继续你等我的消息）
3. 当Server端确定数据已发送完成，则向Client端发送FIN报文。（告诉Client端，好了，我这边数据发完了，准备好关闭连接了）
4. Client端收到FIN报文后，向 Server 发送ACK（Server端收到ACK后，就知道可以断开连接了），如果Server端没有收到ACK则可以重传，如果Client端等待了2MSL后依然没有收到回复，则证明Server端已正常关闭。（那我Client端也可以关闭连接了）

### 负责域名解析的 DNS 服务
DNS(Domain Name System)服务是和 HTTP 协议一样位于应用层的协议。它提供域名到 IP 地址之间的解析服务。DNS 协议提供通过 域名查找 IP 地址，或逆向从 IP 地址反查域名的服务。

## URI 与 URL
- URI：统一资源标志符，可以唯一标识一个资源
- URL：统一资源定位符，可以提供找到该资源的路径

URL 是 URI 的子集
URI 可以是 URL、URN 或者 URL 与 URN 的混合
URL 一定是 URI，反之则不然。

### URI 格式
格式：
    `http://user:pass@www.example.com:80/home/index.html?age=11#mask`
- http：协议方案名
- user:pass：登录信息（认证）
- www.example.com：服务器地址
- 80：端口号
- /hone/index.html：文件路径
- age=11：查询字符串
- mask：片段标识符
**协议方案名**：
    http:、https:、ftp:等，在获取资源时要指定协议类型。
**登录信息(认证)**：
    指定用户名和密码作为从服务器端获取资源时必要的登录信息，此项是可选的。
**服务器地址**：
    使用绝对URI必须指定待访问的服务器地址。
**服务器端口号**：
    指定服务器连接的网络端口号，此项是可选的。
**路径**：
    指定服务器上的文件路径来定位特定资源。格式为: /home/index.html
**参数**：
    为应用程序提供访问资源所需的附加信息。
    例如：ftp://127.27.27.27/pub/pic;type=d
**查询字符串**：
    针对已指定的文件路径内的资源，可以使用查询字符串传入任意参数，此项是可选的。
**片段标识符**：
    通常可标记出以获取资源中的子资源(文档内的某一个位置)，此项是可选的。


## 参考链接
- [TCP的三次握手与四次挥手过程介绍](http://blog.csdn.net/u011726984/article/details/50781212)
- [TCP协议中的三次握手和四次挥手(图解)](http://blog.csdn.net/whuslei/article/details/6667471/)
