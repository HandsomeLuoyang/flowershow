# TCP/IP网络模型

## 应用层

应用层：包装应用数据，由应用自定义，如HTTP的格式如下：
```HTML
GET /network/base/tcp_ip.html HTTP/1.1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36
sec-ch-ua: "Chromium";v="128", "Not;A=Brand";v="24", "Google Chrome";v="128"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Accept: */*
Postman-Token: fc4b20b9-7201-42d9-a10a-4c467f99d3e9
Host: www.baidu.com
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

request body...
```

以key value的形式发送请求头，间隔一行带上请求体

这种格式，和各个请求头的含义说明就是HTTP应用协议的具体体现

## 传输层

应用层的协议具体到了某个应用，在网络上不具有通用性

不是所有的应用都会采用HTTP协议，需要另一个更通用的载体做一个封装

传输层通用的只有两个协议：**TCP** 和 **UDP**

### UDP

加上UDP头，只负责发送，没有可靠性保证，速度快，性能好

### TCP

TCP 的全称叫传输控制协议（_Transmission Control Protocol_），大部分应用使用的正是 TCP 传输层协议，比如 HTTP 应用层协议。TCP 相比 UDP 多了很多特性，比如**流量控制**、**超时重传**、**拥塞控制**等，这些都是为了保证数据包能可靠地传输给对方。

如果应用需要传输的数据超过了MSS（TCP最大报文传输速度），那么数据包就会被分块，一个分块就是一个**TCP段**

如果某一个TCP段传输过程中丢了，只需要重新传输丢掉的那个TCP段，而无需全部重传

在一个设备中可能存在多个应用同时在传输数据，我们使用**端口**来区分不同的应用的数据通道，比如经典的22端口就是用于远程登陆服务器使用，80就是HTTP服务的端口，443就是HTTPS的端口

在浏览器中会有多个标签页，它们会被临时分配一个端口

## 网络层

应用层解决的是同一个应用在不同设备之间的消息协议

传输层解决的是不区分应用的一个消息协议，可以理解成是操作系统的消息协议，通过端口告诉操作系统需要寻找的是哪个应用

那么我们还需要一个标识来在网络中唯一标识一台机器

这就是网络层的作用

网络层会在传输层的基础上生成一个IP头

我们一般用IP协议给设备进行编号，**对于IPV4协议，地址一共32位，分成了四段（192.168.1.1），为4个8位**

只有IP地址是不够的，因为我们不可能通过遍历的方式去寻找设备，我们需要一个索引，一个目录

在不添加额外信息的情况下，我们把IP地址分成了两种意义：

- 网络号：标识该IP地址属于哪个子网，也就是一个目录
- 主机号：标识在该子网下面的不同主机

在这里引入了一个新的概念叫做 **子网掩码** ，子网掩码也是4个8位，用于与IP地址一起计算出网络号和主机号

例如子网掩码是"255.255.255.0"，IP地址是"10.100.122.2"

![[Pasted image 20240902145818.png#pic_center]]

将 255.255.255.0 与IP地址进行进行**按位与运算**，就可以得到主机号。

网络号就是目录，而在一个网络下面可能还存在其他的网络划分，因此这个目录是多级的

假设你有一个IP地址段`192.168.1.0/24`，其默认的子网掩码为`255.255.255.0`，这意味着有256个可用的IP地址（包括网络地址和广播地址），可以用来分配给主机。

而这个网络可以进一步划分：

- **原始子网掩码：255.255.255.0**
    - 二进制表示：`11111111.11111111.11111111.00000000`
- **新子网掩码（借用1位）：255.255.255.128**
    - 二进制表示：`11111111.11111111.11111111.10000000`

通过借用1位，网络被划分为两个子网，每个子网有128个地址：

- 第一个子网：`192.168.1.0/25`，范围是`192.168.1.0`到`192.168.1.127`
- 第二个子网：`192.168.1.128/25`，范围是`192.168.1.128`到`192.168.1.255`

在这个例子中，原来的`/24`掩码（255.255.255.0）被扩展为`/25`（255.255.255.128），即网络部分增加了1位，用来标识子网。

只有这种多级目录的存在，才能在上百上千亿级别的网络主机中快速寻找到指定的主机

有了IP协议就可以进行**路由**

路由器在收到一个网络包之后，只需要看到IP地址的网络号即可通过它的算法进行转发，像是开了高德导航一样快速。

## 网络接口层

事实上，只通过IP地址是不能完全定位到一台机器的，举个最浅显的例子，现在家里基本都安装了宽带，安装了路由器，那么路由器会有一个网关，这个网关会有一个公网地址（不一定有，可能需要申请），会有一个局域网地址

这个局域网地址可能是`192.168.1.1`，而内部的设备并不一定拥有公网地址，它们统一使用的都是路由器的公网地址进行网络通信，它们只有一个局域网的地址比如`192.168.1.35`，并且如果是动态分配的话，这个地址是可变的，且不同的局域网下面可能都有这个地址

那么问题来了，这台设备与其他设备通信的时候填的地址事实上不是自己的地址，而是家的地址（路由器地址），那么信息到了家门口，该把它给谁呢？

这个时候需要一个新的身份认证标识了，就是 **MAC地址** ，它是真正的全球唯一，不可变，绑定在网卡上的一个地址，在信息到了家门口的时候可以用这个地址找到信息真正的主人


MAC 头部是以太网使用的头部，它包含了接收方和发送方的 MAC 地址等信息，我们可以通过 ARP 协议获取对方的 MAC 地址。

所以说，网络接口层主要为网络层提供「链路级别」传输的服务，负责在以太网、WiFi 这样的底层网络上发送原始数据包，工作在网卡这个层次，使用 MAC 地址来标识网络上的设备。


![[Pasted image 20240902152810.png]]

![[Pasted image 20240902152816.png]]

网络接口层的传输单位是帧（frame），IP 层的传输单位是包（packet），TCP 层的传输单位是段（segment），HTTP 的传输单位则是消息或报文（message）。但这些名词并没有什么本质的区分，可以统称为数据包。
