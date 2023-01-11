# Windows 网络编程

## 第一章 网络编程基础

### 1.1 计算机网络基础

#### 1 计算机协议与分层

##### OSI

1981年，ISO制定了Open System Interconnection Reference Model (简称OSI)

|      |            |          |                |
| :--: | :--------: | :------: | :------------: |
|  7   |   应用层   | 上层协议 |      主机      |
|  6   |   表示层   | 上层协议 |                |
|  5   |   会话层   | 上层协议 |                |
|  4   |   传输层   | 下层协议 |                |
|  3   |   网络层   | 下层协议 | 路由器、交换机 |
|  2   | 数据链路层 | 下层协议 |   链路交换机   |
|  1   |   物理层   | 下层协议 |                |

第5层到第7层统称为上层协议，负责端到端的数据通信。

第1层到第4层统称为下层协议，负责建立网络通信连接的链路。

##### TCP/IP

TCP/IP协议筷是美国国防部高级研究规划局在 20世纪60年研究的计算机网络 (Advanced Research Projects Agency Network，简称 ARPANET) 使用的参考模型，后来的因特网 (Internet）也采用了该参考模型。

|      |                   |                   |
| :--: | :---------------: | :---------------: |
|  4   |      应用层       | 对应OSI模型 5 6 7 |
|  3   |      传输层       |   对应OSI模型 4   |
|  2   | 网络互联层/网络层 |   对应OSI模型 3   |
|  1   |    网络接口层     |  对应OSI模型 2 1  |

每层主要任务：

1. 网络接口层：监视数据在主机和网络层之间的通信，将网络层接收到的数据分割进行特定的、可被主机传输的帧，并交付主机物理层进行实际的数据传送
    1. ARP 地址解析协议
    2. WI-FI的802.11
    3. ...
2. **网络互联层：是核心；主要解决主机到主机的通信问题**
    1. IP
    2. ICMP
    3. ...
3. **传输层：使得源端主机和目标端主机上的应用试题可以进行会话**
    1. TCP 传输控制协议
    2. UDP 用户数据协议
4. **应用层：面向不同网络引入不同协议**
    1. 基于TCP协议：
        1. FTP 文件传输协议
        2. HTTP 超文本传输协议
        3. Telnet 远程终端协议
        4. SMTP 简单邮件传输协议
        5. POP3 邮局协议版本3
        6. ...
    2. 基于UDP协议
        1. NTP 网络时间协议
        2. TFTP 简单文件传输协议
        3. SNMP 简单网络管理协议
        4. ...
    3. 基于TCP UDP两种协议
        1. DNS 域名解析服务
        2. ...
    4. ...

#### 2 网络寻址方式

在TCP/IP中，寻址发生在两个级别上：

1. 网络接口层：处理物理地址，即MAC地址
2. 网络互联层：处理逻辑分层地址，即IP地址

##### MAC

MAC（Media Access Control，介质访问控制）地址是识别网络节点的标识；也称为物理地址、网卡地址或硬件地址。

在网络底层的物理传输过程中，是通过物理地址来识别主机的，它一般也是全球唯一的。

以太网卡的物理地址大小是48bit。

##### IP

IP地址：网络层的IP地址寻址主机或网络设备

| 类别 |      网络标识      | 主机标识 |          地址范围          |
| :--: | :----------------: | :------: | :------------------------: |
|  A   |  1字节，最高位为0  |  3字节   |     1.0.0.0～126.0.0.0     |
|  B   | 2字节，最高位为10  |  2字节   | 128.0.0.0～191.255.255.255 |
|  C   | 3字节，最高位为110 |  1字节   | 192.0.0.0～223.255.255.255 |
|  D   |    最高位为1110    |          | 224.0.0.0～239.255.255.255 |
|  E   |   最高位为11110    |          | 204.0.0.0～255.255.255.254 |

##### 端口号

端口号：传输层的端口号用来确定网络应用。

范围：0～65535，自定义应用一般大于1024。

##### 名称解析

名称解析：为了方便用名称或域名来标识网络资源，但对网络协议只认识IP地址，因此要通过名称解析转换为IP地址。

名称解析服务的主要来源有两种，分别是：

1. 域名系统( Domain Name System，DNS)
2. 本地配置数据库（操作系统中用于本地名称与IP地址映射的特殊机制）。

WinSock动态库中的名称解析：

```c
// Windows Socket函数，获取主机名称
gethostname(char FAR *name, int namelen)
  
// 根据主机名获取主机信息，返回struct  hostent FAR *；hostent里有主机的IP地址
gethostbyname( const char FAR *name )：
```

##### 网络地址转换(NAT)

网络地址转换(NAT)：将保留的私有地址转化为公有地址。

是IPv4方案中解决地址短缺的有效方法。

NAT的实现方式有三种:

1. 静态转换
2. 动态转换
3. 端口多路复用

##### NetBIOS

NetBIOS：是Network Basic Input/Output System的简称，用于局域网通信的一套API。作用是为了给局域网提供网络以及其他特殊功能。

报文格式 ：NetBIOS数据报报文，NetBIOS名字报文，NetBIOS会话报文。

利用Netbios函数可以获取MAC地址信息

#### 3 网络通信模型

1. 客户/服务器模型 (C/S)
    + 服务器的特点
        1. 服务器的特权和复杂性
            1. 鉴别：验证客户身份
            2. 授权：给某些客户允许访问特定服务
            3. 数据安全：确保数据不泄漏和损坏
        2. 无连接和面向连接的服务器对应2个主要的传输协议(TCP、UDP)
        3. 无状态和有状态服务器
            1. 服务器是否维护和客户交互的状态信息，主要取决于应用协议
        4. 循环服务器和并发服务器
            1. 循环服务器：通过单线程内设置循环控制实现对多个客户请求的逐一响应。比较适合少量客户持续请求的情况
            2. 并发服务器：通过多线程或单线程异步I/O机制处理多个用户的请求。比较适合大量客户短期请求的情况
2. 浏览器/服务器模型 (B/S)
    + 浏览器(Browser)+WWW服务器
    + HTTP请求和响应(HTTP协议)
    + 优点：分布性、业务扩展简单方便、容易维护等
    + 缺点：页面刷新响应速度相对慢、功能弱化(难以实现传统模式下的特殊功能)
3. P2P模型 (Peer to Peer)
    + 网络中每个节点都是对等的
    + 特点：
        + 非中心化
        + 可扩展
        + 健壮性
        + 自治性
        + 高性价比
        + 隐私保护
        + 负载均衡

#### 4 网络数据

##### 字节顺序

不同体系结构中，计算机存储数据的方式可能不同(字节顺序)

+ 主机字节顺序：不同机器以及操作系统中的字节顺序，分大端顺序和小端顺序
+  网络字节顺序：网络中发送和接收数据的字节顺序，统一为大端顺序
+ 如：0x12345678; 
    + 大端：0x12,0x34,0x56,0x78
    + 小端：0x78,0x56,0x34,0x12

```c
// WinSock定义了一些函数来实现转换
u_long WSAAPI ntohl(u_long netlong);
u_long WSAAPI htonl(u_long hostlong);
u_short WSAAPI ntohs(u_short netlong);
u_short WSAAPI htons(u_short hostlong);
```

##### c语言结构体对齐规则

1. 第一个成员的首地址为0.
2. 每个成员的首地址是自身大小的整数倍
3. 结构体的总大小，为其成员中所含最大类型的整数倍。

```c
// 例子

struct test
{
  char a;   //1+3
  int  b;    //4
  short c;    //2+2
};
```

##### 网络应用程序分类

1. 远程控制类：远程主机监控
2. 网络探测类：主机扫描、网络爬虫
3. 网络管理类：网络监控、网络流量分析
4. 远程通信类：聊天软件、网络游戏
5. 信息发布类：FTP服务、WWW服务

## 第二章 套接字基础

### 1 TCP/IP协议软件接口

操作系统内核集成了对TCP/IP的具体实现，具有常用协议应用能力

目前常用的协议软件接口有：

1. Berkeley Socket:加州大学伯克利分校为Berkeley UNIX操作系统定义的套接字接口；
2. Transport Layer Interface:  AT＆T为其System V定义的传输层接口；
3. **Windows Sockets: Windows系统下的套接字接口。**

### 2 套接字

应用程序通过IP地址和端口号，与系统内核里的协议实现建立关系

套接字接口不直接使用协议类型来标识通信时使用的协议，而是采用一种更为灵活的方式：**协议簇加套接字类型**

常用协议簇：

1. AF_INET：IPv4协议簇；
2. AF_INET6：IPv6协议簇；
3. AF_IPX：IPX/SPX协议簇；
4. AF_NETBIOS：NetBIOS协议簇。
5. AF_UNIX：UNIX，同一主机上的程序之间的通信

常用套接字类型：

1. 流式套接字（SOCK_STREAM）：
    + 流式套接字用于提供面向连接、可靠的数据传输服务。
2. 数据报套接字（SOCK_DGRAM）：
    + 数据报套接字用于提供无连接的数据传输服务。该服务并不能保证数据传输的可靠性。
3. 原始套接字（SOCK_RAW）：
    + 当使用前两种套接字无法完成数据收发任务时，原始套接字提供了更加灵活的数据访问接口，使用它可以在网络层上对套接字进行编程。

### 3 套接字接口的特点

1. 同一个程序可以同时使用多个套接字，不同套接字可以完成不同的传输任务。
2. 多个应用程序可同时使用同一套接字，不过不常见。
3. 每个套接字都有一个关联的本地TCP或UDP端口，端口标识主机上的套接字，进而标识了主机上的特定应用程序。
4. TCP和UDP的端口号是独立使用的，但有时一个端口号也可能关联多个套接字，此时不能仅仅用端口来标识套接字。

### 4 Windows套接字

Windows套接字简称为WinSock

常见版本有WinSock 2和WinSock1.1

Windows继承的BSD Socket主要基本函数：

|      函数名称      | 功能描述                                                  |
| :----------------: | --------------------------------------------------------- |
|   **accept( )**    | **在一个套接宇上接受连接请求**                            |
|    **bind( )**     | **将本地地址与套接字绑定**                                |
| **closesocket( )** | **关闭现有的套接宇**                                      |
|   **connect( )**   | **在指定的套接字与远端计算机之问建立连接**                |
|  gethostbyaddr( )  | 根据网络地址获得计算机信息                                |
|  gethostbyname( )  | 根据计算机名获得计算机信息                                |
|   getpeername( )   | 获得与套接字相连的远端地址信息                            |
|   getsockname( )   | 获得与套接字相连的本地地址信息                            |
| **getsockopt( )**  | **获得套接字的选项值**                                    |
|      htonl( )      | 将口long从主机字节顺序转换为TCP/IP 的网络字节顺序         |
|      htons( )      | 将ushort 从主机字节顺序转换为 TCP/iP 的网络字节顺序       |
|    inet_addr( )    | 将IPv4 的点分宇符形式地址转换为无符号的 4字节整数形式地址 |
|    inet_ntoa( )    | 将IPv4 的无符号4字节整数形式地址转换为点分字符形式地址    |
| **ioctisocket( )** | **控制套接字的 I/O 模式**                                 |
|   **listen( )**    | **将套接字设置为监听状态**                                |
|      ntohl( )      | 将u_long从TCP/IP 网络字节顺序转换为主机字节顺序           |
|      ntohs( )      | 将u_short从TCP/IP 网络字节顺序转换为主机字节顺序          |
|    **recv( )**     | **从套接字上接收数据**                                    |
|  **recvfrom( )**   | **从套接字上接收数据报并获得其源地址**                    |
|     select( )      | 确定一个或多个套接字的状态并等待其是否成功                |
|    **send( )**     | **从套按字上发送数据**                                    |
|   **sendto( )**    | **从套接字上向指定目的地址发送数据报**                    |
|   setsockopt( )    | 设置套接宇选项值                                          |
|    shutdown( )     | 在套接宇上禁用数据的发送或接收                            |
|   **socket( )**    | **创建套接字**                                            |

WinSock 扩展的主要API

|         函数名称         | 功能描述                                         | 版本 |
| :----------------------: | ------------------------------------------------ | ---- |
|      gethostname( )      | 从本地获得计算机名                               | 1.1  |
| WSAAsyncGetHostByAddr( ) | 根据网络地址异步获得计算机信息                   | 1.1  |
| WSAAsyncGetHostByName( ) | 根据汁算机名异屰荻得汁算机信息                   | 1.1  |
|    WSAAsyncSelect( )     | 通知套接字有基于 Windows 消息的网络事件发生      | 1.1  |
|     **WSAClean( )**      | **结束 Windows Sockets DLL 的使用**              | 1.1  |
|    WSAGetLastError( )    | 获得上次失败操作的错误状态                       | 1.1  |
|    **WSAStartup( )**     | **初始化 Windows Sockets DLL 的使用**            | 1.1  |
|       WSAAccept( )       | 基于条件函数的返回值有条件地接受连接请求         | 2.0  |
|  WSAAddressToString( )   | 将 Soekaddr结构的所有成员转换为易读的字符串      | 2.0  |
|     WSACloseEvent( )     | 关闭己打开的事件对象句柄                         | 2.0  |
|      WSAConnect( )       | 与另一个套接字应用程序建立;连找，交换连接数据    | 2.0  |
|    WSACreateEvent( )     | 創建事件対象                                     | 2.0  |
|    WSAEventSelect( )     | 设置与所提供的 FD_xxx 网络事件集合相关的事件对象 | 2.0  |
|     **WSAloctl( )**      | **控制套接字的 I/O 模式**                        | 2.0  |
|        WSARecv( )        | 从己连接的套接字上接收数据                       | 2.0  |
|       WSARecvEx( )       | 从己连接的套按字上接收数据，和reov()一致         | 2.0  |
|      WSARevFrom( )       | 从套接字上接收数据扱并荻得其源地址               | 2.0  |
|     WSAResetEvent( )     | 将指定事件对象的状态复位为非信号状态             | 2.0  |
|        WSASend( )        | 从已连接的套接字上发送数据                       | 2.0  |
|       WSASendTo( )       | 将数据发送到指定的地址，可以适当使用重叠I/O      | 2.0  |
|      WSASetEvent( )      | 将指定的事件对象设置为信号状态                   | 2.0  |
|       WSASocket( )       | 创建套接字                                       | 2.0  |
|  WSAStringToAddress( )   | 将数值字符串转换为 Sockaddr 结构                 | 2.0  |

### 5 套接字的主要I/O模式

1. 阻塞模式：是指在指定套接字上调用函数执行操作时，在执行没有有完成之前，函数不会立即返回。
2. 非阻塞模式：是指在指定套接宇上调用函数执行操作时，无论操作是否完成，函数都会立即返回。

### 6 Windows套接字的I/O模式

在Windows系统中提供了7种套接字的I/O模式：

1. 阻塞I/O模式
2. 非阻塞I/O模式
3. I/O复用模式
4. WSAAsyncSelect模式
5. WSAEventSelect模式
6. 重叠I/O模式
7. 完成端口模式

阻塞与非阻塞的I/O模式以及I/O复用模式是网络通信中I/O处理的常用方式

## 第三章 流式套接字编程

### 1 TCP协议

Transmission Control Protocol，传输控制协议：面向连接的、可靠的、基于字节流的传输层通信协议，由RFC 793定义。

特点：

1. 端到端通信
2. 面向连接
3. 可靠交付
4. 具有流量控制
5. 双工传输
6. 流模式

TCP三次握手，四次挥手

通信流程：

1. 服务器：在流式套接字的通信流程中，服务器是服务提供方，被动接受客户端的连接请求
    1. 创建套接字，并指定套接字类型为流式套接字；
    2. 绑定本地的地址和通信端口；
    3. 启动监听等待客户端的连接请求；
    4. 和客户端连接成功；
    5. 进行数据发送接收；
    6. 关闭套接字。
2. 客户端：客户端是服务请求方，主动发起连接请求
    1. 创建套接字，并指定套接字类型为流式套接字；
    2. 设置服务器的地址和通信端口；
    3. 向服务器发起连接请求；
    4. 和服务器连接成功；
    5. 进行数据发送接收；
    6. 关闭套接字。



### 非阻塞模式

```c
int ioctlsocket( SOCKET s, long cmd, u_long FAR *argp );

该函数用来控制套接口的模式(  Winsock 1 定义)
s：一个标识套接口的描述字 
cmd：对套接口s的操作命令 
argp：指向cmd命令所带参数的指针
	如cmd = FIONBIO，argp所指向的参数值为0 表示阻塞模式，如值为1表示非阻塞模式

```





### 流式套接字代码

```c
// server
	WSADATA wsa_data;
	WORD winsock_ver = MAKEWORD(2, 0);   
	int ret = WSAStartup(winsock_ver, &wsa_data);  // 初始化WinSock  
	SOCKET sock;
	int nPort = 5000;    // 题目要求的端口
	if((sock = socket(AF_INET, SOCK_STREAM, 0)) == INVALID_SOCKET) 	{
		return -1;	
	}                                          // 创建套接字
	int addr_len = sizeof(SOCKADDR_IN);
	SOCKADDR_IN addr_svr;                   // 地址
	addr_svr.sin_family = AF_INET;
	addr_svr.sin_port = htons(nPort);           // 端口
	addr_svr.sin_addr.S_un.S_addr = INADDR_ANY;   // 任意地址
	ret = bind(sock, (SOCKADDR*)&addr_svr, addr_len);  // 绑定地址 
	if (ret != 0)	{
		closesocket(sock);
		return -1;
	}
	ret = listen(sock, 100);                          // 启动侦听
	if (ret == SOCKET_ERROR)
	{
		closesocket(sock);
		return -1;
	}
	SOCKADDR_IN addr_clt;
	// 等待客户端连接
	SOCKET sock_clt = accept(sock, (SOCKADDR*)&addr_clt, &addr_len);	
if (sock_clt == INVALID_SOCKET)
	{
		closesocket(sock);
		return -1;
	}
	char msg[1024];
	int nNumRead;
	while( 1 )
	{
		memset(msg, 0, 1024 );
		nNumRead = recv(sock_clt, msg, 1024, 0);        // 接收数据
		
		if( nNumRead > 0 )             // 接收到的数据大小 > 0
		{
			// 这部分代码根据题目具体要求实现
			// ……
		}
		
	}
	closesocket(sock);   // 关闭套接字
	WSACleanup();    //释放WinSock                       
```

```c
// client
	WSADATA wsa_data;
	WORD winsock_ver = MAKEWORD(2, 0);  
	int ret = WSAStartup(winsock_ver, &wsa_data);  // 初始化WinSock 
	int nPort = 5000;    // 题目要求的端口
	char strIP[] = "10.5.100.13";  // 服务器地址
	SOCKADDR_IN addrSrv;  
	addrSrv.sin_family = AF_INET;  
	addrSrv.sin_port = htons(nPort);  //端口
	addrSrv.sin_addr.S_un.S_addr = inet_addr((strIP));            
	//创建套接字  
	SOCKET sock = socket(AF_INET, SOCK_STREAM, 0);    
	if(SOCKET_ERROR == sock) {  
		return -1;  
	}  
	//向服务器发出连接请求                          
	if(connect(sock,(struct sockaddr*)&addrSrv,sizeof(addrSrv))== INVALID_SOCKET)
	{  
		closesocket(sock);
		return -1;  
	}
	char szBuffer[1024];
	while( 1 )
	{
		// 这部分代码根据题目具体要求实现
		// ……
		gets(szBuffer);                                      
		send(sock,szBuffer, strlen(szBuffer),0);              // 发送数据

	}
	closesocket(sock);   // 关闭套接字
	WSACleanup();    //释放WinSock        
```



## 第四章 数据报套接字编程

###  1 UDP协议

User Datagram Protocol ，用户数据报协议：无连接的、不可靠的、基于报文的传输层通信协议，由RFC 768定义。

UDP协议传输特点

1. **多对多通信**：UDP在通信实体的数据量上具有更大的灵话性，多个发送方可以向一个接收方发送报文，一个发送方也可以向多个接收方发送数据，更重要的是，UDP能让应用使用底层网络的广播或组播设施交付报文。
2. **不可靠服务**：UDP提供的服务是不可靠交付的，即报文可以丢失、重复或失序，它没有重传设施，如果发生故障，也不会通知发送方。
3. **缺乏流量控制**：UDP不提供流量控制，当数据包到达的速度比接收系统或应用的处理速度快时，只是将其丢弃而不会发出警告或提示。
4. **报文模式**：UDP提供了面向报文的传输方式，在需要传输数据时，发送方准确指明要发送的数据的字节数。UDP将这些数据放置在一个外发报文中，在接收方，UDP一次交付一个传入报文。因此当有数据交付时，接收到的数据拥有和发送方应用程序所指定的一样的报文边界。

通信流程：

1. 服务器：在数据报套接字的通信流程中，服务器一般是被动进行通信的一方
    1. 创建套接字，并指定套接字类型为数据报套接字；
    2. 绑定本地地址和通信端口；
    3. 等待接收客户端的数据；
    4. 和客户端进行数据的发送接收；
    5. 关闭套接字。
2. 客户端
    1. 创建套接字，并指定套接字类型为数据报套接字；
    2. 设置服务器的地址和通信端口；
    3. 向服务器发送数据；
    4. 和服务器进行数据发送接收；
    5. 关闭套接字。

### 数据报套接字代码

```c
// server
WSADATA wsa_data;
	WORD winsock_ver = MAKEWORD(2, 0);   // WinSock 2.0版本
	WSAStartup(winsock_ver, &wsa_data);  // 初始化WinSock     
	SOCKET sock;
	if((sock=socket(AF_INET,SOCK_DGRAM,0)) == INVALID_SOCKET) {
		return -1;	}                                 //创建套接字
	SOCKADDR_IN saddr;  
	//设置地址信息  
	int addr_len = sizeof(SOCKADDR_IN);
	memset(&saddr, 0, addr_len);
	saddr.sin_family = AF_INET;  
	saddr.sin_port = htons(3000);  // 题目要求的端口
	saddr.sin_addr.s_addr =  INADDR_ANY;   // 任意地址              
	int ret = bind(sock,(struct sockaddr*)&saddr,addr_len);       // 绑定地址
	if(ret != 0)  	{  
		closesocket(sock);
		return -1;
	}  
	char recvbuf[256];
	char sendbuf[256];
	SOCKADDR_IN addr_clt;
	int nNum;
	while ( 1 )
	{
		// 这部分代码根据题目具体要求实现
		memset(recvbuf, 0, 256 );  
		// 接收数据                       
		int nNum=recvfrom(sock,recvbuf, 256,0,(sockaddr*)&addr_clt,&addr_len); 
		if(nNum  > 0 )  
		{  
			strcpy(sendbuf, "ok");       
			// 发送数据，需要客户端的地址，因为数据报是无连接的
			sendto(sock,sendbuf, strlen(sendbuf),0, (sockaddr*)&addr_clt, addr_len); 
		}  
	}
	closesocket(sock);   // 关闭套接字
WSACleanup();    //释放WinSock

```

```c
// client
WSADATA wsa_data;
	WORD winsock_ver = MAKEWORD(2, 0);   // WinSock 2.0版本
	int ret = WSAStartup(winsock_ver, &wsa_data);  // 初始化WinSock
	SOCKET sock;
	if((sock = socket(AF_INET, SOCK_DGRAM, 0)) == INVALID_SOCKET)	{
		return -1;		}                       // 创建套接字
	char strServerIP[] = "192.118.1.100";   // 服务器地址
	SOCKADDR_IN SrvAddr;
	int addr_len = sizeof(struct sockaddr_in);  
	memset(&SrvAddr, 0, addr_len);
	//设置地址信息，ip信息  
	SrvAddr.sin_family = AF_INET;  
	SrvAddr.sin_port = htons(3000);  //  端口，从主机字节顺序转变成网络字节顺序
	SrvAddr.sin_addr.s_addr = inet_addr(strServerIP);        // 服务器地址    
	char recvbuf[256];
	char sendbuf[256];
	int nNum;

	while( 1 )
	{
		// 这部分代码根据题目具体要求实现
		gets(sendbuf); 
		// 发送数据，需要服务器的地址，因为数据报是无连接的                  
		ret=sendto(sock,sendbuf,strlen(sendbuf),0, (sockaddr*)&SrvAddr,addr_len); 
		memset(recvbuf, 0, 256 );  
		// 接收数据
		nNum = recvfrom(sock,recvbuf, 256,0,(sockaddr*)& SrvAddr,&addr_len); 
		if(nNum  > 0 )  
		{
			printf("%s\n", recvbuf);   
		}
	}
	closesocket(sock);   // 关闭套接字
WSACleanup();    //释放WinSock                      
```



## 第五章 原始套接字编程

原始套接字 发送和接受数据流 与标准套接字(TCP/ IP)有何不同

原始套接字可以接收发送底层数据，并可以对底层数据进行控制，即可以越过TCP/IP协议栈的部分层次进行数据的收发

原始套接字能做的 但是标准套接字不行

1. 原始套接字可以发送和接收ICMP、IGMP，能够处理在IP头中预定义的网络层上的协议分组
2. 原始套接字发送和接收内核不处理其协议字段的IPv4数据包
3. 原始套接字可以控制IPv4首部，通过设置IP_HDRINCL套接字选项可以控制IPv4首部字段的自行构造

适用场合







## 第六章 应用层网络编程

应用层的网络编程初略可以分为两大部分：

1. 通用的应用层协议相关的网络编程
2. 用户特定的应用层需求的网络编程。

### 1 应用层网络协议

1. 电子邮件：用于实现电子邮件收发的功能，如SMTP、POP3。
2. Web应用：也称WWW或Web服务，如HTTP、HTTPS。
3. 文件传输：用于实现通过网络传输文件的功能，如FTP、TFTP。
4. 域名解析：用于实现网络设备名字到IP地址映射的网络服务，如DNS。
5. 网络管理：用于管理与监视网络设备状况，如(SNMP)。
6. 远程登录：用于实现登录远程主机的功能，如Telnet 、SSH(Secure Shell)。

##### FTP※

File Transfer Protocol，文件传输协议。用于Internet上的控制文件的双向传输。基于TCP的应用层协议，用户可通过客户端向（从）服务器上传（下载）文件。 

FTP标准TCP端口号为21，PORT方式数据传输端口为20。

FTP支持两种模式：

1. Standard (PORT模式，主动方式)。
    + 在这种模式中服务器被认为是发起者。
2. Passive (PASV模式，被动方式)。
    + 客户端运行的pasv命令告诉服务器等待客户端的连接，而不是自己发起新连接。这里的关键区别在于客户端发起到服务器所提供的IP地址和端口的传输数据的连接。在这种关系中，服务器被认为是数据通讯的被动者。

##### SMTP

SMTP协议的实现采用的也是客户/服务器模型，基于TCP协议传输数据，使用的端口号为25。

SMTP通信模型：用户向某个目的地址发送邮件的请求，实际是从邮件客户端向目的地址的SMTP服务器发送邮件数据。在邮件的发送过程中需要有SMTP发送方(客户端)和SMTP接收方(服务器)，SMTP发送方与SMTP接收方之间建立一个双向传送通道。SMTP接收方可能是最终接收的SMTP服务器，也可能是中间传递的SMTP服务器。SMTP发送方负责发送SMTP命令，SMTP接收方收到命令后做出应答返回SMTP发送方。整个邮件的发送过程基本是由一组SMTP服务器接力完成的

##### POP3

POP3协议的实现采用的也是客户/服务器模型，基于TCP协议传输数据，使用的端口号为110。

POP3工作过程是：SMTP协议把邮件发送到服务器上，用户调用邮件客户端连接邮件服务器，然后下载未阅读过的电子邮件。这种离线访问模式是一种存储转发服务，将邮件从邮件服务器送到个人终端主机上。

**邮件服务器**通常是由**SMTP服务器**和**POP3服务器组成**：

+ SMTP服务器负责发送(或转发)邮件、接收邮件并存储，POP3服务器负责让用户通过邮件客户端下载邮件。

##### DNS

域名系统将域名解析为IP地址，通常使用UDP端口53(也有使用随机端口)

DNS攻击：

+ 攻击目标：更改LDNS的缓存，使要攻击的域名对应IP改为攻击者希望的IP
+ 攻击条件和限制：攻击者无法截断和篡改LDNS发出和接收的网络数据，无法通过攻击直接获取LDNS缓存的读写能力
+ 在LDNS发出域名查询请求时，攻击者仿冒权威域名服务器发送伪造的响应包并尽快回应（比真正的响应包快一步送达），该伪造响应包里有伪造的域名的A记录，A记录里的IP是攻击者伪造的IP，只要该报文被LDNS接收，并且把记录记入缓存，目标即达成

##### HTTP

HTTP协议的实现采用的也是客户/服务器模型，基于TCP协议传输数据，缺省使用的端口号为80。

HTTP协议也是基于请求响应模型。客户端向服务器发送一个HTTP请求，请求头包含请求的方法、URI(Uniform Resource Identifier，统一资源标识符)、协议版本、可接收的消息类型和客户端信息等。服务器则返回一个状态行作为响应，内容包括消息协议的版本、成功或失败编码加上包含服务器信息、实体元信息以及可选的实体内容。

+ HTTP协议是无状态的，也就是说用户在多次向同一台服务器发送相同HTTP请求时，服务器的响应都是相同。
+ HTTP协议是无连接的，虽然HTTP协议的传输是基于TCP的，但是HTTP本身是无连接的。客户端和服务器的连接是基于一种请求应答模式。也就是客户端和服务器建立一个连接后，客户端提交一个请求，服务器收到请求后回送一个响应，客户端收到响应后二者就断开此连接。

HTTP请求由三部分组成：请求行、消息头、请求正文。

HTTP响应也是由三个部分组成：状态行、消息报头、响应正文。

#### WinInet

WinInet(Windows Internet)API是Windows平台对套接字进行封装后的一组API函数，WinInet让程序员可以通过WinInet直接处理HTTP、FTP、Gopher协议。

特点：

1. WinInet 是一个网络编程接口，包含了Internet 应用层协议 HTTP、FTP和Gopher的实现接口；
2. 借助WinInet 接口，可不必去了解 WinSock、TCP/IP协议和特定Internet应用层协议的细节就可以编写Internet 客户端程序；
3. WinInet 为HTTP、FTP、Gopher提供了统一的函数集，也就是Win32 API 接口；
4. WinInet 简化了HTTP、FTP、Gopher的编程，可轻松地将 Internet 集成到应用程序中。

#### Internet应用相关类

1. CInternetSession类

    + ```c
        /*
         * 类描述 使用类CInternetSession可创建并初始化一个或多个Internet会话。
         * 
         * pstrAgent：该字符串用于标识应用程序
         * dwContext：操作的上下文标识符
         * dwAccessType：访问类型
         * pstrProxyName：首选CERN代理的名称
         * pstrProxyBypass：代理服务器地址可选列表
         * dwFlags：缓存选项
         */
        CInternetSession( LPCTSTR pstrAgent = NULL, DWORD_PTR dwContext = 1, DWORD dwAccessType = PRE_CONFIG_INTERNET_ACCESS, LPCTSTR pstrProxyName = NULL, LPCTSTR pstrProxyBypass = NULL, DWORD dwFlags = 0 );
        
        
        
        
        /**
         * 函数描述 调用此函数建立FTP连接并返回指向CFtpConnection对象的指针
         *
         * @param pstrServer：FTP服务器名字
         * @param pstrUserName：FTP登录用户名称
         * @param pstrPassword：FTP登录密码
         * @param nPort：服务器上使用的TCP/IP端口
         * @param bPassive：FTP会话的被动或主动模式
         * 
         */
        CFtpConnection* GetFtpConnection( LPCTSTR pstrServer, LPCTSTR pstrUserName = NULL, LPCTSTR pstrPassword = NULL, INTERNET_PORT nPort = INTERNET_INVALID_PORT_NUMBER, BOOL bPassive = FALSE ); 
        ```

2. CFtpConnection类

    + 使用该类可以管理和FTP服务器的连接，从而可以操作FTP服务器上的文件

3. CHttpConnection类

    + 使用该类可以管理和HTTP服务器的连接，从而可以操作HTTP服务器

4. CFtpFileFind类

    + 可以通过网络搜索FTP服务器上的文件

5. CTreeCtrl类

    + 操作节点

## 第七章 网络底层编程

网络底层编程会面临更加深入的网络数据分析和更加灵活的数据构造需求。网络底层通常指网络模型中的下基层：OSI数据链路和物理层  TCP的网络接口层

### 1 WinPcap

WinPcap是一个基于Win32平台的，用于捕获网络数据包并进行分析的开源库。

WinPcap提供了以下功能：

1. 捕获原始数据包
2. 根据规则过滤捕获数据包
3. 发送原始数据包
4. 收集并统计网络流量

典型应用：

1. 网络与协议分析器 (network and protocol analyzers) 
2. 网络监视器 (network monitors) 
3. 网络流量记录器 (traffic loggers) 
4. 网络流量发生器 (traffic generators) 
5. 用户级网桥及路由 (user-level bridges and routers) 
6. 网络入侵检测系统 (network intrusion detection systems ) 
7. 网络扫描器 (network scanners) 
8. 安全工具 (security tools)





简述Winpcap捕获数据包的过程。

+ 答：
    1. 使用pcap_findalldevs()发现网卡信息
    2. 使用pcap_open_live()获得数据包捕获描述字
    3. 使用pcap_loop开始捕获数据包，其中要设置回调函数处理捕获的数据
    4. 使用pcap_breakloop()结束捕获 

考虑对抓取的数据包如何进行过滤？如何只捕获UDP和ICMP数据包？

+ 答：首先使用pcap_compile()编译过滤规则，将编译好的过滤规则使用pcap_setfilter()设置到Winpcap驱动里。使用过滤规则ICMP or UDP 可以只捕获UDP和ICMP数据包

主要函数：

```c
// 获得网卡信息的列表
int pcap_findalldevs(pcap_if_t **, char *)     
/*
pcap_if_t*类型的网卡信息列表的地址，需要使用pcap_freealldevs函数释放资源;
char型指针,当打开列表错误时返回错误信息
返回值: 为int型，当失败时返回-1
*/


// 获得数据包捕获描述字用于捕获数据包
pcap_t * pcap_open_live(char *device, int snaplen, int promisc, int to_ms, char *ebuf);    
/*
参数：
   device网络设备名称
   snaplen定义捕获数据的最大字节数
   promisc是否将网络接口置于混杂模式
   to_ms参数指定超时时间（毫秒）
   ebuf在函数出错时用于传递错误消息
*/


// 返回链路层的类型
	 int   pcap_datalink(pcap_t *p);      
参数：
	 p使用pcap_open_live获得的捕获标识字
返回值：（20多种类型)
	DLT_EN10MB： 以太网（10Mb, 100Mb, 1000Mb, 或者更高）。
	DLT_RAW：原始IP（raw IP）；数据包以IP头开始。


// 过滤捕获的数据包
int pcap_compile(pcap_t *p,struct bpf_program *fp,char *str,int optimize,bpf_u_int32 netmask);      
参数：
	p使用pcap_open_live获得的捕获标识字
	str将该字符串编译到过滤程序中
	fp该结构在函数中被赋值
	optimize参数控制结果代码的优化
	netmask参数指定本地网络的网络掩码

// 过滤捕获的数据包
int pcap_setfilter(pcap_t *p, struct bpf_program *fp);      
参数：
	p使用pcap_open_live获得的捕获标识字
	fp使用在pcap_compile函数中获得的结果

// 捕获数据包
int pcap_loop(pcap_t * p,int cnt, pcap_handler callback, uchar * user);
参数:
	p使用pcap_open_live获得的捕获标识字
	cnt捕获数据包的个数
  callback回调函数
  user用户使用参数，和callback之间参数的关联

// 回调函数callback原型如下:
pcap_callback(u_char* argument,const struct pcap_pkthdr* packet_header,const u_char* packet_content);
回调函数必须是全局函数或静态函数
argument和user相关联
packet_header捕获到的数据包头部基本信息
packet_content捕获到的数据包的内容


// 中断捕获数据包
void	pcap_breakloop(pcap_t * p);
参数:
	p使用pcap_open_live获得的捕获标识字


// 
pcap_findalledvs() 
    pcap_if_t  name设备名
pcap_open_live()
    pcap_t 捕获标识字
pcap_compile() 
    bpf_program 编译后的过滤条件
pcap_setfilter()
pcap_loop()
    启动捕获,里面设置回调函数
		回调函数参数是捕获的数据包头，数据包内容
pcap_breakloop():结束捕获
```



### 2 ARP

地址解析协议(Address Resolution Protocol)，根据IP地址获取物理地址的TCP/IP协议。I

P地址处于OSI模型的网络层，物理地址(MAC)处于数据链路层，只有通过地址解析将这两层连接起来，从而保证网络通信顺利进行。

ARP欺骗：

+ 地址解析协议是建立在网络中各个主机互相信任的基础上的；
+ 网络上的主机可以自主发送ARP应答消息，其他主机收到应答报文时不会检测该报文的真实性就会将其记入本机ARP缓存；
+ 攻击者就向某主机发送伪装的ARP应答报文，这将使该主机发送的某些信息无法到达预期的主机或到达错误的主机，这就构成了ARP欺骗
