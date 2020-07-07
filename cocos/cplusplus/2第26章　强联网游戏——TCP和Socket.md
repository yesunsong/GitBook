# 第**26**章　强联网游戏——**TCP**和**Socket**

在强联网这几章（26～29章）中，要介绍一些非常有意思的内容，把强联网这几章的知识掌握好可以巩固网络基础知识，因为内容过于丰富，所以需要分为4章来介绍。

第26章介绍TCP的基础知识，包括Socket接口、跨平台处理、心跳检测、半包粘包、非阻塞、Select详解，最后以一个简单的TCP服务器和客户端作为总结。

第27章介绍一个横版RPG的单机版本的实现，这不是一个纯粹的单机游戏，其可以很优雅地变身为网络版，所以读者应好好吸收这一章的一些设计思路。

第28章介绍前后端通信流程的设计，以及服务端的实现，使用一个简易的开源服务器框架KxServer来实现服务器，并轻松将部分前端的逻辑代码移植到后端。

第29章通过少量的改动，并使用KxClient将单机版本变身为网络版，讨论并解决游戏的实时同步问题。

本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Socket接口与TCP。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　简单的TCP服务器端与客户端。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　非阻塞Socket与select()函数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　半包粘包。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　心跳与超时。

### 26.1　Socket接口与TCP

什么是Socket？什么是TCP？Socket和TCP有什么联系？

Socket也称为套接字，描述了IP和端口，用于网络通信，应用程序需要通过套接字来发起网络请求或接收网络消息。

TCP是一种面向连接的、可靠的、基于字节流的传输通信协议。TCP在两个主机中间提供了一条可靠的连接来进行数据传输。可以创建一个TCP Socket，来创建这样的一条连接。

每一条TCP连接，都是由一端的IP+端口连接到另一端的IP+端口，每台主机都有IP（多网卡的情况下可以有多个IP），用于标识唯一的一台主机，每台主机最多有65535个端口，一个端口可以对应多条连接，连接数的上限取决于操作系统，例如，服务器开一个端口，可能有几万条连接连到服务器的这个端口，这是没有问题的。

在编写强联网的实时网游时，用得最多的就是TCP了（**HTTP协议也是基于TCP协议实现的**），TCP提供可靠的连接，可以确保数据**有序**地到达目标地址，在连接正常的情况下，**不需要**担心发出去的数据会**丢失**（TCP底层实现了丢包重发的机制，通过ACK确认来判断发出的包对方是否收到）、错乱（TCP的数据包是有序的）或者错误（TCP使用一个校验和函数来校验数据是否有错误）。在使用TCP之前，先简单介绍接下来要使用到的几个Socket API，这些看似简单的Socket API实际上隐藏着大量细节，对其内部运行流程及细节和原理的了解在一定程度上决定了程序员的网络编程水平，本章试图详细讲述这些内部细节，但却并非区区数万字所能概括，因此只能提醒初学者，网络编程并非只是调用几个函数那么简单。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121424.jpeg)**说明：**本章的内容对于初学者而言，可能有些地方较为深入，第一次看本章内容时不必纠结于其中的一些概念，了解大概流程即可，**可以在看完第3篇的通篇内容之后，再回过头来细读本章。**

#### 26.1.1　TCP服务器与客户端交互流程

在介绍Socket API之前，先了解一下TCP客户端与服务端是如何通信的。TCP服务器与客户端交互的流程分为3个阶段，建立连接阶段，数据通信阶段以及关闭连接阶段。下面简单了解一下这3个阶段中的应用层需要做什么以及TCP底层做了什么。

首先是建立连接阶段，连接是由客户端主动发起的，那么在客户端连接的时候，服务端必须是处于监听状态才能连接成功（**例如，我要去你家里串门，那么你必须在家我才能进去**）。

所以服务端必须先启动并进入监听状态，这时候服务端需要依次调用3个函数，socket()函数创建套接字，bind()函数绑定网卡和端口，listen()函数进入监听状态。网卡和端口是整台主机的公有资源，所有的进程都可以访问，当某网卡的某端口已经被绑定时就不能再重复绑定（**除非所有绑定的Socket都设置了SO_REUSEADDR选项**）。

客户端需要依次调用两个函数来连接服务器，这两个函数中socket()函数创建套接字，connect()函数连接服务器。对于客户端套接字而言，bind()函数并不是必须的，在调用connect()函数时操作系统会自动选取网卡和端口，connect()函数将向服务端发送一个SYN报文，服务端Socket处于监听状态时，会回复一个SYN+ACK报文，在客户端Socket接收到这个报文后，会再次回复一个ACK报文，这3次报文的传输称为3次握手，完成3次握手后TCP连接就建立完成了。3次握手由客户端的connect()函数发起，由两端的TCP底层完成，在握手完成后，阻塞的connect()函数调用会返回连接成功。

当服务器的Socket处于监听状态时，接收到客户端发起的SYN报文，会回复SYN+ACK报文进行3次握手，在服务端Socket的TCP底层，存在两个队列，一个是正在执行3次握手的队列（正在连接队列），一个是已完成连接的队列，队列的大小由listen()函数的第二个参数指定，但这个参数对应的队列大小是实现相关的。在已监听的服务端Socket调用accept()函数可以从TCP底层的已完成连接队列中取出一个连接的Socket对象，然后进行通信，如果底层的已完成连接队列为空，那么accept()函数会阻塞，一直等到有客户端连接成功为止。

建立连接的流程如图26-1所示，但客户端的连接并不会改变服务器的监听Socket的状态，syn_recv和established状态对应客户端连接的Socket对象而不是监听Socket对象。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121425.jpeg)

图26-1　3次握手流程图

在数据通信阶段中，服务端和客户端都可以调用send()函数来发送数据，调用recv()函数来接收数据。客户端Socket必须是已连接的Socket才可以发送和接收消息，而服务端需要使用accept()函数返回的Socket来发送和接收消息。在每次数据传输到对端之后，对端的应用层就可以调用recv()函数进行接收了，同时对端的TCP底层会回复一个ACK报文给发送端，告诉发送端已经收到消息了，流程如图26-2所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121426.jpeg)

图26-2　数据发送接收流程图

send()函数会将数据从应用层复制到TCP底层的发送缓冲区中，然后由TCP进行发送，send()函数完成时，发送端并不能保证对端已经接收到。当发送缓冲区已满时，send()函数会被阻塞，直到send()函数的所有内容被拷贝进发送缓冲区。recv()函数会阻塞直到对端发送数据到本端的接收缓存区中，recv()函数会将接收缓存区的内容复制到应用层。当接收缓存区已满时，对端发送过来的数据会被丢弃，但TCP的流量控制会尽量避免这一情况发生。

最后是关闭连接阶段，关闭连接的流程如图26-3所示。任何一端都可以调用close()函数来关闭TCP连接，关闭需要经过4次握手，一般是由客户端来发起关闭。当调用close()函数关闭后，对端调用recv()函数时会返回0，这时对端需要调用close()函数来完成连接的关闭。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121427.jpeg)

图26-3　4次握手流程图

#### 26.1.2　Socket API详解

在大概了解了TCP客户端与服务端的通信流程之后，下面来看一下相关的Socket API。

##### 1．socket()函数

在使用socket()函数进行通信之前，需要先创建一个Socket对象，通过调用socket()函数传入一个协议域、一个socket类型以及指定的协议，来创建一个Socket套接字并返回套接字的描述符。在Linux下的Socket是一个文件，所以socket()函数返回了一个文件描述符。socket()函数描述符可以用来绑定、连接以及发送和接收数据。

```
int socket(int domain, int type, int protocol);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　domain：协议域，又称协议族。常用的协议族有AF_INET、AF_INET6、AF_UNIX、AF_ROUTE等。协议族决定了Socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用IPv4地址（32位）与端口号（16位）的组合，AF_UNIX决定了要用一个绝对路径名作为地址。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　type：指定Socket类型。常用的Socket类型有SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等。流式Socket（SOCK_ STREAM）是一种面向连接的Socket，针对于面向连接的TCP服务应用。数据报式Socket（SOCK_DGRAM）是一种无连接的Socket，对应于无连接的UDP服务应用。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　protocol：指定协议。常用协议有IPPROTO_TCP、IPPROTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC等，分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议。type和protocol不可以随意组合，如SOCK_STREAM不可以跟IPPROTO_UDP组合。当第3个参数为0时，会自动选择第2个参数类型对应的默认协议。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如成功则返回所创建的Socket的文件描述符，失败返回-1。

如果调用成功就返回新创建的套接字的描述符，如果失败，Windows下会返回INVALID_SOCKET，Linux下失败返回-1。套接字描述符是一个整数类型的值。每个进程的进程空间里都有一个套接字描述符表，该表中存放着套接字描述符和套接字数据结构的对应关系，套接字数据结构都是在操作系统的内核缓冲里。

##### 2．bind()函数

在进行网络通信的时候，必须把套接字绑定到一个地址上，可以使用bind()函数进行绑定。套接字的协议族决定了要绑定的地址类型，常用的TCP和UDP协议都是需要绑定到一个32位的IP地址+16位的端口号上。当不要求绑定到一个明确的地址和端口时，例如，作为客户端连接服务器时，可以不调用bind()函数进行绑定，而是在连接时让操作系统自动将套接字绑定到一个可用的地址上。

```
int bind(int sockfd, const struct sockaddr* address, socklen_t address_len);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：套接字描述符，要绑定的套接字。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　address：sockaddr结构指针，该结构中包含了要结合的地址和端口号。这个结构的定义在不同的平台有区别。实际上需要填充一个**sockaddr_in**结构体，在调用时将这个结构体指针转换为sockaddr指针传入。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　address_len：address的长度，传入实际结构的长度即可。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如成功则返回客户端的文件描述符，失败返回-1。

address结构的填充是这样的：我们需要绑定IP和端口并设置协议族，在设置端口和IP时，需要将IP和端口从主机字节序转换成**网络字节序**。htons()和htonl()函数可以完成这个功能。另外设置IP时，Windows下需要将IP设置到sockaddr_in.sin_addr.S_un.S_addr中，而在Linux下，需要设置到sockaddr_in.sin_addr.s_addr中。

```
sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(port);
//Windows下
addr.sin_addr.S_un.S_addr = inet_addr("192.168.0.29");
//Linux下
addr.sin_addr.s_addr = inet_addr("192.168.0.29");
bind(socketfd, (struct sockaddr*)addr, sizeof(addr));
```

作为服务器的套接字需要绑定IP和端口，相当于通知操作系统，在这个网卡设备上，所有发往我所监听的端口的消息让我来处理。那么一台主机可能拥有多个网卡，甚至动态地增减网卡（**未尝试过**），通过绑定INADDR_ANY，也就是"0.0.0.0"这个任意地址类型，可以绑定所有网卡。这相当于通知操作系统，只要是发往我监听的端口的消息都交给我处理，不管是哪个网卡。

```
//将inet_addr改为htonl(INADDR_ANY)
addr.sin_addr.s_addr = htonl(INADDR_ANY);
```

##### 3．listen()函数

listen()函数可以将一个已经完成绑定的套接字设置为监听状态，并使其可以接受连接请求。当**调用了listen()函数之后，这时候客户端可以连接成功，完成3次握手**。连接成功的Socket会被放到一个队列中，调用accept()函数可以从队列中取出Socket套接字并进行通信。

由于3次握手需要一段时间，所以一个连接有可能处于**正在执行3次握手**的半连接状态下，操作系统会为它另外维护一个队列。已连接和半连接队列的大小是有限制的，第2个参数backlog被定义为这两个队列的大小总和，这个参数所能接受的取值范围与操作系统的实现相关。一般将其设置为100。当大量进程在同一个瞬间连接服务器时，如果队列已满，那么新的连接的SYN请求会被丢弃，这时候TCP的超时机制会让客户端自动重发SYN请求。

```
int listen(int sockfd, int backlog);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：套接字描述符。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　backlog：排队等待应用程序accept()函数的最大连接数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如成功则返回客户端的文件描述符，失败返回-1。

##### 4．accept()函数

当有客户端连接上服务器完成3次握手时，会被放到一个队列中，调用accept()函数可以从队列中取出一个Socket套接字进行通信。一个队列中可能有多个已连接的套接字等待accept()函数调用。**这是一个阻塞操作**！非阻塞的accept()函数在没有新的客户端需要accept()函数调用时，会立即返回失败。

```
int accept( int sockfd, struct socketaddr* addr, socklen_t* len);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：一个已经成功调用了listen()函数的套接字描述符。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　addr：这是一个输出参数，返回客户端连接的地址信息，通过这个结构可以知道客户端的IP和端口，这个信息可以作为日志记录，也可以作黑名单或白名单使用。如果不关心客户端的地址信息，可以设置为NULL。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　len：接收返回地址的缓冲区长度，如果不关心客户端的地址信息，可以设置为NULL。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如成功则返回客户端的文件描述符，失败返回-1。

##### 5．connect()函数

当希望连接TCP服务器时，需要调用connect()函数，传入一个未连接的Socket套接字，以及要连接的服务器IP和端口来连接服务器。connect()是一个阻塞的函数，会向服务器发起3次握手，当3次握手成功之后或者连接失败才返回。当套接字是一个非阻塞套接字时，connect()函数会立即返回。当连接成功或失败时，Socket会变成可写，这时需要判断Socket是否连接成功。

```
int connect(int sockfd, const struct sockaddr* server_addr, socklen_t
addrlen)
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：一个未连接的套接字描述符。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　server_addr：要连接的TCP服务器地址信息，参考bind()函数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　addrlen：server_addr结构体的大小。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如成功则返回0，否则返回-1。

##### 6．recv()函数

对一个已连接的套接字调用recv()函数可以接收数据，这个函数是一个阻塞函数，会一直阻塞住，直到有数据可读才会返回。recv()函数会从TCP缓存区中读取数据到传入的缓存区中，如果TCP缓存区的内容大于接收缓存区的容量，那么需要调用多次recv()函数来接收。非阻塞的recv()函数会将数据读出，如果没有数据则立即返回。

```
int recv(int sockfd, char* buf, int len, int flags);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：一个已连接的套接字描述符。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　buf：用于接收数据的缓冲区。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　len：缓冲区长度。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　flags：指定调用方式，0表示读取数据到缓存区，并从输入队列中删除。MSG_PEEK表示查看当前数据，数据将被复制到缓冲区中，但并不从输入队列中删除，MSG_OOB表示处理带外数据。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：若无错误发生，recv()函数返回读入的字节数，如果连接已中止则返回0，否则返回-1。

##### 7．send()函数

对一个已连接的套接字，调用send()函数可以发送数据，这个函数是一个阻塞函数，正常情况下会将要发送的数据复制到TCP底层的发送缓存区，并发送到连接的另一端，当另一端接收到时（**并不需要程序调用recv**()函数），会回复一个ACK报文来告诉连接的一端已经接收到了（**如果没有收到这个确认，那么TCP会重发这个包**）。

send()函数并不一定能将要发送的数据全部发送出去，当TCP缓冲区快满的时候，例如，要发送100个字节的数据，而TCP发送缓存区只能放下10个字节的数据，非阻塞的send()函数会一直等到将所有要发送的内容写到发送缓冲区之后函数才返回。而非阻塞的send()函数会返回10，只有前面10个字节的数据会被发送出去，剩下的内容需要再次调用send()函数发送。

```
int send( int sockfd, const char* buf, int size, int flags);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：套接字。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　buf：待发送数据的缓冲区。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　size：待发送缓冲区长度。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　flags：调用方式标志位，一般为0。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如果成功则返回发送的字节数，失败则返回-1。

##### 8．close()函数

close()函数可以关闭一个Socket套接字，当要关闭一条连接的时候，可以调用close()函数来关闭。close()函数并不会立即关闭连接，而是发送一个FIN报文给对方，进入4次握手流程，当4次握手流程走完，Socket会进入TIME_WAIT状态，不论是客户端还是服务端，谁主动关闭，谁就会进入TIME_WAIT状态，这个状态会占用一定的资源，在一段时间后才会完全关闭（**大约是2～3分钟**）。

```
int close(int sockfd);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sockfd：要关闭的Socket套接字。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　返回值：如成功则返回0，失败返回-1。

#### 26.1.3　Windows Socket API详解

在Windows下使用Socket，需要做一些调整，首先Socket套接字在Windows下并不是一个int类型，而是SOCKET类型。socket()函数如果失败，会返回INVALID_SOCKET，而不是-1，将Socket API的套接字类型由int变为SOCKET即可。

在Windows下需要包含WinSocket的头文件WinSock2.h。在Windows下还需要引用WinSocket的库文件，可以加上以下代码：

```
#pragma comment(lib, "ws2_32.lib")
```

另外，要在Windows下使用Socket，还需要先调用WSAStartup()函数来初始化WinSocket，当不需要使用Socket时，再调用WSACleanup()函数进行清理。下面是使用WSAStartup()函数初始化WinSocket的代码，当初始化失败时，会调用WSACleanup()函数进行清理。

```
WSADATA wsaData;
WORD wVersionRequested = MAKEWORD( 2, 2 );
int err = WSAStartup( wVersionRequested, &wsaData );
if ( err != 0 )
{
    return;
}
if ( LOBYTE( wsaData.wVersion ) != 2 ||  HIBYTE( wsaData.wVersion ) != 2 )
{
    WSACleanup();
    return;
}
```

### 26.2　简单的TCP服务器端与客户端

在了解完TCP服务器与客户端的工作流程之后，本节来实现一个最简单的TCP通信示例，相当于是网络编程的Hello World——回显服务器。服务器可以在Windows运行，也可以在Linux下运行。

#### 26.2.1　TCP服务器实现

首先是服务器的实现，服务器会在8888端口进行监听，当一个客户端连接上时，等待客户端发送消息，接收到客户端发送的消息再原样发送给客户端，然后关闭客户端。

如果是在Windows下，需要创建一个Win32控制台项目（WIN32预处理是Win32工程属性中定义的），首先要包含一些头文件，然后做一些预处理以方便代码的跨平台。

```
#include<stdio.h>
//消除平台相关的时间，Socket差异
#ifdef WIN32
#include <WinSock2.h>
typedef SOCKET sock;
typedef int sockLen;
#define badSock (INVALID_SOCKET)
#pragma comment(lib, "ws2_32.lib")
#else
#include<sys/types.h>
#include<errno.h>
#include<fcntl.h>
#include<unistd.h>
#include<sys/socket.h>
#include<netinet/in.h>
#include<arpa/inet.h>
typedef int sock;
typedef socklen_t sockLen;
#define badSock -1
#endif
```

在main()函数中，实现我们的服务器。

```
int main()
{
    int err = 0;
    char buf[512];
    initSock();
    //创建一个TCP Socket
    sock server = socket(AF_INET, SOCK_STREAM, 0);
    check(server != badSock);
    //绑定到端口
    sockaddr_in addr = sockAddr(8888);
    err = bind(server, (sockaddr*)&addr, sizeof(addr));
    check(err != -1);
    //开始监听
    err = listen(server, 10);
    check(err != -1);
    do
    {
        //取出一个客户端连接
        sock client = accept(server, NULL, NULL);
        //接收数据并打印
        int n = recv(client, buf, sizeof(buf), 0);
        printf("server recv %d byte: %s", n, buf);
        //发送回给客户端并关闭客户端
        n = send(client, buf, n, 0);
#ifdef WIN32
        closesocket(client);
#else
        close(client);
#endif
    } while(true);
    cleanSock();
    return 0;
}
```

服务器用到了自定义的initSock()、cleanSock()和sockAddr()函数，用来初始化Socket、清理Socket，以及获取Socket地址结构体。具体实现代码如下。

```
void initSock()
    {
    #ifdef WIN32
        WSADATA wsaData;
        WORD wVersionRequested = MAKEWORD( 2, 2 );
        int err = WSAStartup( wVersionRequested, &wsaData );
        if ( err != 0 )
        {
            return;
        }
        if ( LOBYTE( wsaData.wVersion ) != 2 ||HIBYTE( wsaData.wVersion ) !=
2 )
        {
            WSACleanup();
            return;
        }
    #endif
    }
    void cleanSock()
    {
    #ifdef WIN32
        WSACleanup();
    #endif
    }
    sockaddr_in sockAddr(int port)
    {
        sockaddr_in addr;
        addr.sin_family = AF_INET;
        addr.sin_port = htons(port);
   #ifdef WIN32
       addr.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
   #else
       addr.sin_addr.s_addr = htonl(INADDR_ANY);
   #endif
       return addr;
   }
   sockaddr_in sockAddr(int port, const char* ip)
   {
       sockaddr_in addr;
       addr.sin_family = AF_INET;
       addr.sin_port = htons(port);
   #ifdef WIN32
       addr.sin_addr.S_un.S_addr = inet_addr(ip);
   #else
       addr.sin_addr.s_addr = inet_addr(ip);
   #endif
       return addr;
   }
```

#### 26.2.2　TCP客户端实现

最后是客户端的实现，需要创建一个新的程序，客户端实现了连接服务器，发送数据，接收数据后打印并关闭的工作，代码如下。

```
　　int main()
　　{
　　    int err = 0;
　　    char buf[512];
　　    initSock();
　　    //创建一个TCP Socket
　　    sock client = socket(AF_INET, SOCK_STREAM, 0);
　　    check(client == badSock);
　　    //连接服务器
　　    sockaddr_in addr = sockAddr(8888, "127.0.0.1");
　　    err = connect(client, (sockaddr*)&addr, sizeof(addr));
　　    check(err == -1);
　　    //发送数据到服务器
　　    sprintf(buf, "hello world!");
　　    send(client, buf, strlen(buf) + 1, 0);
　　    //接收数据并打印
　　    int n = recv(client, buf, sizeof(buf), 0);
　　    printf("client recv %d byte: %s\n", n, buf);
　　#ifdef WIN32
　　        closesocket(client);
　　#else
　　        close(client);
　　#endif
　　   //暂停
　　   getchar();
　　   cleanSock();
　　   return 0;
　　}
```

运行服务器，然后再运行客户端，可以看到服务器打印了一句server recv 13 byte : hello world!，而客户端也打印了一句client recv 13 byte : hello world!，因为将hello world!后面的\0结尾也发送过去了，所以接收到的是13个字节。

### 26.3　非阻塞Socket与select()函数

#### 26.3.1　非阻塞Socket

在26.2节的例子中，使用了阻塞的Socket进行通信，accept()、connect()、recv()和send()这几个函数的调用会导致程序阻塞，当处于阻塞状态下，程序就做不了其他事情了。例如，客户端在调用connect()函数连接服务器的时候，我们希望场景的Loading动画正常播放，而当connect，阻塞时，Loading动画会卡住。在游戏中调用recv()函数接收数据时，整个游戏都会被阻塞。将Socket设置为非阻塞可以很好地解决这个问题，所有的函数执行后都会立即返回，不会阻塞游戏。

使用多线程也可以解决阻塞问题，但多线程用起来相对比较危险，特别是在访问公共资源的时候，需要为这些公共资源上锁，锁的粒度太大，容易影响效率，锁的粒度太小，没有锁好可能导致程序出现各种BUG，并且多线程的BUG比较难定位，除非要做的事情足够简单、独立，完全在掌控之中，或者是对多线程有着丰富的经验，可以让一切都在掌控之中，否则建议还是使用非阻塞的Socket。

在Windows下是通过ioctlsocket()函数来设置套接字的非阻塞，而在iOS和Android下是通过fcntl()函数，可以用预处理来封装一下阻塞设置的代码。

```
void setNonBlock(sock s, bool noblock)
{
#ifdef WIN32
    //when noblock is true, nonblock is 1
    //when noblock is false, nonblock is 0
    ULONG nonblock = noblock;
    ioctlsocket(s, FIONBIO, &nonblock);
#else
    int flags = fcntl(s, F_GETFL, 0);
    noblock ? flags |= O_NONBLOCK : flags -= O_NONBLOCK;
    fcntl(s, F_SETFL, flags);
#endif
}
```

#### 26.3.2　select()函数的使用

当将Socket设置为非阻塞之后，可以使阻塞的socket()函数立即返回，例如recv()函数，需要实时地知道对方有没有发送消息过来，可以通过在游戏主循环里不停地调用recv()，当接收到数据的时候，recv()函数会返回接收到的字节数，没有数据则返回-1（**错误码一般为EAGAIN或者EWOULDBLOCK**），**非阻塞操作中，EAGAIN或者EWOULDBLOCK错误是可以接受的**。在主循环里面不断地调用recv()函数的方法效率较低（**假设是服务器这么做，只要有几百个连接，对性能的影响就已经非常大了**）。另外对于非阻塞的connect()函数，我们需要知道什么时候连接成功，可以使用socket()函数来发送或接收数据，也需要在主循环中不断地检测是否连接成功。

推荐的做法是使用**IO复用**配合非阻塞Socket，什么是IO复用？IO复用是一种高效管理连接的IO模型，可以很高效地管理成百上千的连接，当Socket有数据来到，或者要发送，IO复用会通知应用程序，然后让应用程序去调用recv()或send()，而不是让所有的Socket不断地去调用recv()函数。使用select()函数可以实现简单的IO复用，在前面几章中已经使用了select()函数。Epoll和IOCP分别是Linux和Windows下（**目前**）最强大的IO复用实现，用于高效地管理成千上万的连接，但对于一般的Cocos2d-x客户端而言，select()函数就足够了，并且select()函数在Linux、Mac和Windows下都有实现，很容易就可以编写出跨平台的select()。我们可以在游戏的主循环update中，调用select()函数来检测套接字触发的事件，根据事件进行相应的处理。

select()的函数原型如下。

```
int select(int maxfd, fd_set* readfds, fd_set* writefds, fd_set* errorfds,
struct timeval* timeout);
```

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　maxfd参数在Linux下是所有待检测的Socket套接字描述符中，最大的描述符的值+1，而Windows下该值可以忽略。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　fd_set结构体是Socket套接字的集合，readfds、writefds和errorfds对应监听套接字是否可读、可写以及异常。这几个参数既是输入参数又是输出参数。当Socket套接字有数据可读时，套接字会被放到readfds中，当Socket套接字有数据可写时，或调用connect()函数的套接字连接成功时，会被放到writefds中，当调用connect的套接字连接失败时，会被放到errorfds中。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　timeout可以指定select()函数的超时时间，timeval结构体包含两个变量，tv_sec为秒，tv_usec为微秒（百万分之一秒），select()函数会按照指定的时间进行等待，如果在指定的时间内触发了事件，则返回，否则会在时间结束后返回。当timeout传入NULL时，select()函数会一直阻塞直到监听到事件才返回。指定的时间为0时，select()函数不阻塞。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　select()函数的返回值等于0时，表示超时，无事件触发。大于0时，返回值表示准备就绪的Socket套接字数量，小于0时表示出错。

在调用select()函数之前需要设置好fd_set，而在调用完select()函数之后，需要根据返回值来判断fd_set中是否有套接字准备就绪。通过下面几个宏可以操作fd_set。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　FD_CLR(s, *set)将套接字从set中移除。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　FD_ISSET(s, *set)判断套接字在set中是否处于就绪状态。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　FD_SET(s, *set)将套接字添加到set中。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　FD_ZERO(*set)将set清空。

Windows下select()函数详细文档可参与https://msdn.microsoft.com/en-us/library/ms740141(VS.85).aspx。

select()函数可以用来批量检测套接字是否可读、可写或异常，在使用select()函数的时候有以下几步。

（1）将需要检测的套接字放到对应的套接字集合中（fd_set对象在系统底层是一个数组，有监听可读，可写和异常3个集合）。

（2）在主循环中，调用select()函数，将几个集合作为参数传入，同时传入最大套接字ID以及等待的时间参数。

（3）在调用完select()函数之后，根据返回的结果进行处理，处理完之后，重新执行步骤（1）。

关于select()函数的使用，也可以参考http://blog.csdn.net/piaojun_pj/article/details/5991968中的内容。

#### 26.3.3　调整TCP服务器为Select模型

下面将前面例子的TCP服务器调整为select()函数+非阻塞Socket来实现。这意味着可以同时处理多个连接，所以这里会用一个stl的set容器来管理连接。首先将监听的Socket设置为非阻塞，这样调用accept()函数就不会阻塞住了，accept()函数返回的Socket也需要设置非阻塞，这样与客户端的通信也不会阻塞了。

接下来使用select()函数来检测这些套接字是否可读，select()函数返回结果之后，需要遍历所有的Socket来检测这些Socket是否触发了事件，然后进行处理。示例代码如下。

```
//添加两个全局变量方便处理
fd_set g_inset;
set<sock> g_clients;
int main()
{
    int err = 0;
    int maxfd = 0;
    initSock();
    //创建一个TCP Socket
    sock server = socket(AF_INET, SOCK_STREAM, 0);
    check(server != badSock);
    setNonBlock(server, true);
    //绑定到端口
    sockaddr_in addr = sockAddr(8888);
    err = bind(server, (sockaddr*)&addr, sizeof(addr));
    check(err == -1);
    //开始监听
    err = listen(server, 10);
    check(err == -1);
    //设置select()函数的等待时间
    timeval t;
    t.tv_sec = 0;
    t.tv_usec = 1000;
    //清空全局的fd可读集合，将服务器的监听socket()函数添加进去
   FD_ZERO(&g_inset);
   FD_SET(server, &g_inset);
   do
   {
       //每次都重置inset，这样只需要维护好g_inset即可
       fd_set inset = g_inset;
       int ret = select(maxfd, &inset, NULL, NULL, &t);
       if(ret > 0)
       {
           //如果是服务器就绪，说明有客户端连接
           if(FD_ISSET(server, &inset))
           {
               processAccept(server, maxfd);
               --ret;
           }
           //判断是否有客户端套接字就绪，有则处理
           for(set<sock>::iterator iter = g_clients.begin();
               iter != g_clients.end() && ret > 0; )
           {
               sock client = *iter;
               if(FD_ISSET(client, &inset))
               {
                   --ret;
                   //如果客户端关闭从g_inset和g_clients中清除该客户端
                   if(!processClient(client))
                   {
                       FD_CLR(client, &g_inset);
                       g_clients.erase(iter++);
                       continue;
                   }
               }
               ++iter;
           }
           //当select()函数触发的事件处理完，会提前结束遍历
       }
   } while(true);
   return 0;
}
```

上面代码中调用了两个函数，即processAccept()函数以及processClient()函数，分别用来处理客户端连接以及客户端发送消息。

```
void processAccept(sock &server, int &maxfd)
{
    sock client = accept(server, NULL, NULL);
    if(client != badSock)
    {
        //添加到g_clients进行管理，并设置到g_inset中，在下次循环时监听该套接字
        g_clients.insert(client);
        FD_SET(client, &g_inset);
        //将客户端Socket设置为非阻塞
        setNonBlock(client, true);
        //在Linux下，需要为select()函数传入一个正确的maxfd
#ifndef WIN32
        if(maxfd < client)
        {
            maxfd = client;
        }
#endif
    }
}
bool processClient(sock &client)
{
    char buf[512];
    //接收数据并打印
    int n = recv(client, buf, sizeof(buf), 0);
    if(n > 0)
    {
        printf("server recv %d byte: %s\n", n, buf);
        //发送回客户端并关闭客户端
        n = send(client, buf, n, 0);
    }
    else if(n == 0)
    {
        //关闭Socket，返回false
        printf("client close");
#ifdef WIN32
        closesocket(client);
#else
        close(client);
#endif
        return false;
    }
    return true;
}
```

通过调整之后，服务端就可以同时处理多个客户端了，这也称作并发处理。select()函数适合处理1000以内的连接数，并且很多系统限制了select()函数能处理的最大连接数为1024，强行修改这个值，可以使select()函数能处理更多的连接数，但同时效率也会随之下降。如果希望处理更多的连接，epoll()和iocp()函数拥有更强大的并发处理能力。对于一般的客户端程序，大概了解非阻塞操作就够用了，在游戏每一帧的update去调用一次非阻塞recv()函数的消耗并不算大，但建议仍然应该使用select()函数来进行轮询。

### 26.4　半包粘包

在了解了非阻塞和select()函数的使用之后，接下来需要了解一下数据接收相关的处理，在初学阶段，基本上消息处理都很轻松，但随着传输频率的提高以及传输数据的复杂，就会开始接触到半包粘包的问题。

#### 26.4.1　什么是半包粘包

**半包粘包是使用TCP的时候经常会碰到的问题，这个问题是必须了解并解决的一个问题**！这是由于TCP的特性导致，但这本身不是TCP的缺陷，而是应用层需要处理的内容，TCP本来就是流式套接字，只管把数据有序地发送到对端。这里的包是应用层的概念。TCP的更底层会有IP包，但与应用层的包不一样，应用层的包是程序逻辑上的概念。

半包指的是你一次性发送100个字节的内容，但是对方调用recv()函数只接收到了50个字节。而连包是指你第一次发送了50个字节，然后再发送50个字节，对方调用recv()函数一次接收到了100个字节。

举个例子，开学之际，学校的食堂服务器要向全校的师生连续发3条信息：热烈欢迎，新老师生，前来用餐。那么在半包的情况下，可能第一次收到“热烈”，第二次收到“欢迎”，然后是剩下的内容。而连包的情况可能第一次收到的是。“热烈欢迎新老师生前”，第二次收到“来用餐”。

半包在什么情况下容易出现呢？一般是一次性发送的数据量太大，TCP无法一次性发送完，例如，需要将一个视频文件（几十MB）发送给对方，TCP底层会对其进行分片，数据发送到对方的Socket中无法一次性发送完，因此会出现半包。

相对于半包，粘包的情况非常常见，因为TCP有一个延迟发送的规则，调用send()函数会将数据写入到TCP的发送缓冲区中，但不会立即发送，因为在TCP中会有很多细小的分节（ACK报文），TCP在发送的时候捎上这些小分节可以很好地节约带宽，而且一般一个send()函数会对应一次recv()函数，就是收到数据之后，处理完成，返回，TCP的这个规则可以很好地适应这种情况，将recv()函数的ACK报文和send()函数的数据一起发出，而且当**快速的调用send()函数两次的时候，对方调用recv()函数接收到的一般也会是这两次send()函数的所有数据**。

除了将数据一次性发送的规则之外，TCP的接收是底层先将数据复制到系统的TCP接收缓冲区（**每个连接都有一个**），连续发送N次的数据都会先放到接收缓冲区里，recv()函数的调用是从这个系统的缓冲区中读取数据，所以当调用recv()函数接收的时候，可能之前发送的几个包都在这里了。一次调用recv()函数就可以全部接收过来。

对于半包，处理的策略是不能丢弃，因为一旦将这个半包丢弃，后面的数据会全部错乱，因此需要把半包缓存起来，等待剩余的数据，然后拼成一个完整的包再处理。对于粘包的处理策略是，将数据包一个个从一连串的内存中区分开，然后单独处理。

#### 26.4.2　处理半包粘包

处理半包粘包的重点在于，如何判断这个包是一个半包还是粘包亦或是正常的包，常用的有两种判断方法。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　第一种是为每个数据包定义一个包头，**在包头中填上这个数据包的大小**，根据这个字段和接收到的实际数据大小来判断包是否完整。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　第二种是**在每个包的最后加上一个结束标识**，当判断到这个结束标识的时候，认为数据包已经完整，否则认为数据包不完整，粘包的情况也可以使用这个结束标识来区分。

一般情况下使用第一种，因为第一种的效率和适用性会更好一些，第二种方法需要校验接收到的所有数据来查找结束标识，并且在数据内容中，不得出现与该标识相同的内容，否则数据包会解析错误，而第一种方法就没有这些问题了，下面的代码将介绍半包粘包是如何处理的。

下面是一个TcpClient接收数据的回调，TcpClient封装了一个Socket，当这个Socket可读时，外部会调用TcpClient的onRecv()函数进行接收。代码里用到了m_ProcessModule的RequestLen()和Process()函数，这两个函数的功能分别是根据数据包结合协议计算出该数据包完整长度是多少，以及处理一个完整的数据包。

在接收到数据之后，先判断是否存在半包，如果是则先将数据拼接到半包之后，然后根据接收到的数据是否完整来进行处理，优先处理半包拼接成的数据包，再将剩余的粘包进行遍历处理，直到处理完所有的包或者碰到半包无法处理，或者数据解析异常。

```
int CTcpClient::onRecv()
    {
        char buf[512];
        int requestLen = 0;
        int ret = recv(m_socket, buf, sizeof(buf), 0);
        if(ret <= 0) return -1;
        char* processBuf = buffer;
        char* stickBuf = NULL;
        //m_RecvBuffer缓存了上一次没有接收完的半包
        //如果存在半包，需要把新的内容追加到半包后面
        //这时有两种情况，接收到的数据长度大于半包所剩余的内容长度，或者小于等于半包所
          剩余的内容长度
        if (NULL != m_RecvBuffer)
        {
            unsigned int newsize = ret;
            if ((m_RecvBufferLen - m_RecvBufferOffset) < (unsigned int)ret)
            {
                newsize = m_RecvBufferLen - m_RecvBufferOffset;
                stickBuf = processBuf + newsize;
            }
            //复制到接收缓冲区中
            memcpy(m_RecvBuffer + m_RecvBufferOffset, processBuf, newsize);
            m_RecvBufferOffset += newsize;
            ret += m_RecvBufferLen;
            processBuf = m_RecvBuffer;
        }
        //对包进行处理，m_ProcessModule是一个处理对象
        //这里用来查询包的长度
        requestLen = m_ProcessModule->RequestLen(processBuf, ret);
        if (requestLen <= 0 || requestLen > MAX_PKGLEN)
        {
            //解析出来的包数据错误
            return requestLen;
        }
        //如果未到达预期的包长度，说明是半包
        //将半包缓存到m_RecvBuffer中
        if (ret < requestLen)
        {
            if (NULL == m_RecvBuffer)
            {
                m_RecvBuffer = new char[requestLen];
                m_RecvBufferLen = requestLen;
            m_RecvBufferOffset = ret;
            memcpy(m_RecvBuffer, processBuf, ret);
        }
        return ret;
    }
    //如果等于或超过了预期的长度，可以逐个处理
    //直到处理完所有的包或者碰到半包无法处理
    while (ret >= requestLen)
    {
        m_ProcessModule->Process(processBuf, requestLen, this);
        processBuf += requestLen;
        //m_RecvBuffer中只会有一个半包，并且最先处理的就是半包
        //处理完这个半包后就可以释放缓存半包的内存了
        if (NULL != m_RecvBuffer)
        {
            processBuf = stickBuf;
            delete [] m_RecvBuffer;
            m_RecvBuffer = NULL;
            m_RecvBufferOffset = m_RecvBufferLen = 0;
        }
        //如果存在粘包，继续处理后面的包
        //直到处理完所有的包，或碰到半包、数据异常才返回
        ret -= requestLen;
        if (ret > 0
            && NULL != processBuf)
        {
            //取出下一个包所需的长度
            requestLen = m_ProcessModule->RequestLen(processBuf, ret);
            if (requestLen <= 0 || requestLen > MAX_PKGLEN)
            {
                return requestLen;
            }
            //半包缓存
            else if (ret < requestLen)
            {
                //在这里m_RecvBuffer一定是NULL
                m_RecvBuffer = new char[requestLen];
                m_RecvBufferLen = requestLen;
                m_RecvBufferOffset = ret;
                memcpy(m_RecvBuffer, processBuf, ret);
                return ret;
            }
        }
    }
｝
```

### 26.5　心跳与超时

现在我们已经可以正确地处理数据以及正常的连接关闭了，但这还不够，还需要掌握一些网络异常处理的能力。说到网络异常，最常见的网络异常就是断网。网络的正常关闭，是由一方调用close()函数发起的，通过发送FIN报文给对方来进行4次握手关闭。当程序崩溃时，操作系统会关闭套接字，完成4次握手。**所以程序的正常退出以及异常退出，操作系统都会帮我们回收，关闭套接字资源**。当然，自己创建的套接字自己关闭是一个良好的习惯。

当将网线拔掉、WiFi信号突然中断或者主机突然断电时，是没有任何消息通知到TCP的，我们的socket仍然可以发送、接收数据，但数据无法传输到对端，数据是否传输到了对端，TCP是通过对端回复的ACK报文来确定的。如果没有回复，则视为丢包，丢包的情况下，TCP的处理是超时未收到确认的ACK报文，则进行重发。由于没有任何消息通知到TCP，所以从TCP层的角度来看，当前的连接是正常的，但实际上已经无法工作了。这种情况下，我们知道网络断了，但是TCP不知道。

当我们很快速地断网再重连，这时候Socket的连接并不一定会产生异常。也就是说，物理网线的断开跟TCP连接的断开不是一回事，是一条虚拟的连接，由TCP协议维护的虚拟连接，不要把这条连接与网线混为一谈，虽然数据是通过网线发出去，这肯定没错，所以这条网线用来描述TCP连接感觉就非常地恰当，但是，UDP是无连接的，它的数据传输就不用经过网线了吗？所以，TCP上面说的这条连接，跟大部分初学者理解的物理网线连接不是一回事，TCP连接更像是一条想象中的连接。

如图26-4演示了物理连接中断时，TCP的数据传输情况。实际上拔网线，同本机到对端主机的物理线路中任何一个环节中断了，在TCP的角度来看都是一样的。如果在这种状态下，将主机断电，或者将应用程序关闭，那么这个TCP连接的对端，就永远收不到任何消息了。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121428.jpeg)

图26-4　连接中断

#### 26.5.1　TCP的死连接

这种在TCP层处于连接状态，而通信两边的数据又无法传输的TCP连接称之为死连接。对于这个问题，客户端和服务端所处的情况不同，解决问题的策略也不同。死连接会占用系统资源，当服务器存在大量的死连接而没有及时清理时，会降低服务器的运行效率，甚至耗光服务器的Socket资源，导致服务器无法连接（现在的硬件设备不太容易出现这种情况）。所以对于服务器而言，只需要能在一段时间内检测出死连接并关闭清理即可。这段时间可以是30分钟、2个小时，不需要过于频繁地检查。

对于客户端而言，在大多数情况下，希望程序能够立即检测到网络断开的消息。及时反馈是客户端的需求，程序员可以**在客户端发起请求的同时添加一个计时器，当收到服务端的响应时，移除这个计时器，当计时器的时间到了之后还没有接收到服务端的响应，那么视为连接已中断**。当然，也可以使用和服务端一样的检测方法，在没有用户触发请求的情况下，来检测连接是否断开。

#### 26.5.2　检测死连接

由于死连接的Socket在逻辑上并没有任何异常，所以无法从Socket本身检测到任何错误。那么如何判断一个连接是否已死呢？死连接的特点就是不会有任何消息发过来，你的消息也无法到达对端。在实际中用的最多的就是心跳包。

什么是心跳包？一般是一个非常小的包，在里面程序员可以按照自己的协议发送任何数据，它的作用就是让程序员接收到数据，**当一段时间没有接收到数据时，关闭这个连接**，所以，**心跳包还需要借助定时器来实现**，每次收到心跳包，都更新一下定时器的超时时间。心跳包这个说法很形象！人死了，就没有心跳了，而连接死了，也不会有心跳。心跳包一般是用来检测死链接用的，但不仅仅局限于这个用途，心跳包还可以用于各种检测，如看服务端是否有新的公告，或者时间之类的校验，不过最多的还是检测死链接。

用心跳来检测死连接，因为死连接不会有心跳，玩家的网线断了，玩家客户端的心跳肯定是发不到服务端的，这种情况下，服务端就可以根据客户端的心跳频率判断它是不是死链接，假设客户端在30分钟内都没有心跳，那就可以断定客户端死了。服务端这时候可以主动关闭这条连接，心跳脉搏的频率可以自己定，5分钟到1个小时都是正常的范围，**根据业务需求来定一个合理的超时时间**。如果不想这么麻烦，也可以用TCP自带的Socket选项SO_KEEPALIVE来设置TCP的保活定时器，系统默认是2小时监测一次。

服务端也可以主动发送一个数据到客户端，但一般不这么做，像这种既可以是客户端主动发送又可以是服务端主动发送的事情，肯定是让客户端来做的，因为每个客户端做一次，放到服务端可能就是数万次了。往死链接上发送数据时，如果对方主机在一定时间内没有收到响应，则会返回一个RST报文，意思是这条连接已经失效了，重置这条连接。