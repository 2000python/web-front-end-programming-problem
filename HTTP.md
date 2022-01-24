# 网络知识点（读书笔记）

[HTTP权威指南[文字版][中文版].pdf](https://drive.google.com/file/d/1A_uESQLpD-vFK3ofZns9GiObXKZ1C6m1/view?usp=drivesdk)

[TCP-IP详解卷1：协议.pdf](https://drive.google.com/file/d/1zR8mOrwQjigzX8-9CW1xsMbr4o4gC8QH/view?usp=drivesdk)

## 1 TCP连接

全球几乎所有的HTTP通信都是TCP/IP承载的。

比如，我想获取一个页面，浏览器会做以下工作（部分）：

![https://rainsin-1305486451.file.myqcloud.com/img/TCP/1-%E8%BF%9E%E6%8E%A5%E5%B7%A5%E4%BD%9C.png](https://rainsin-1305486451.file.myqcloud.com/img/TCP/1-%E8%BF%9E%E6%8E%A5%E5%B7%A5%E4%BD%9C.png)

### 1.1 TCP/IP的分层

TCP/IP是一种常用的分组交换网络分层协议集，由好多的协议组成，如下图所示：

![分层](https://rainsin-1305486451.cos.ap-nanjing.myqcloud.com/img/TCP/1-%E5%88%86%E5%B1%82.png)

TCP和UDP是两种最为著名的运输层协议,二者都使用IP作为网络层协议。虽然TCP使用不可靠的IP服务,但它却提供一种可靠的运输层服务。一些 TCP的应用,如Telnet和Rlogin、FTP以及SMTP等。这些应用通常都是用户进程。

UDP为应用程序发送和接收数据报。一个数据报是指从发送方传输到接收方的一个信息单元(例如,发送方指定的一定字节数的信息)。但是与TCP不同的是,UDP是不可靠的,它不能保证数据报能安全无误地到达最终目的。一些UDP的应用，如DNS：域名系统,TFTP：简单文件传送协议,以及BOOTP：引导程序协议，SNMP也使用了UDP协议,但是由于它还要处理许多其他的协议。

IP是网络层上的主要协议,同时被 TCP和UDP使用。TCP和UDP的每组数据都通过端系统和每个中间路由器中的IP层在互联网中进行传输。在上图中,我们给出了一个直接访问IP的应用程序。这是很少见的,但也是可能的，一些较老的选路协议就是以这种方式来实现的，当然新的运输层协议也有可能使用这种方式。

ICMP是IP协议的附属协议。IP层用它来与其他主机或路由器交换错误报文和其他重要信息。两个流行的诊断工具,Ping和Traceroute,它们都使用了ICMP。

IGMP是Internet组管理协议。它用来把一个UDP数据报多播到多个主机。

ARP(地址解析协议)和RARP(逆地址解析协议)是某些网络接口(如以太网和令牌环网)使用的特殊协议,用来转换 I P层和网络接口层使用的地址。

HTTP的安全版本HTTPS只是在HTTP和TCP之间加了一个安全加密层（TLS或SSL）。

这其中我们主要了解应用层协议：HTTP、传输层协议：TCP、网络层协议：IP

### 1.2 TCP是可靠的数据通道

HTTP连接🔗实际上是TCP连接及其使用规范。TCP为HTTP提供了一条可靠的比特传输管道，

在TCP/IP协议族中,网络层IP提供的是一种不可靠的服务。也就是说,它只是尽可能快地把分组从源结点送到目的结点,但是并不提供任何可靠性保证。而另一方面, TCP在不可靠的IP层上提供了一个可靠的运输层。为了提供这种可靠的服务,TCP采用了超时重传、发送和接收端到端的确认分组等机制。由此可见,运输层和网络层分别负责不同的功能。

### 1.3 封装、分用及透明

#### 封装

HTTP要传送一条报文时，会以流的形式将报文数据通过一条确认的TCP连接按序传输。

TCP收到数据流时会将数据流切割成一段段的数据块（段）包装成TCP报文段传递给下一层，

