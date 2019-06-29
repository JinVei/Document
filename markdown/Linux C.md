# 字节序
  * 大端字节序
    > 大端字节序是指一个数据的高位字节内存低地址，低位字节存在内存高地址。
  * 小端字节序
    >  小端字节序是指数据的高位字节存储在高地址，而低位字节存储在内存低地址。
  * 现代PC大多采用小端字节序，因此小端字节序又被称为主机字节序。
  * 网络上的字节传输通常采用大端字节序，故大端字节序又称网络字节序。
  * Linux下主机字节序和网络字节序之间的装换函数：
    * 头文件：#include<netinet/in.h>
    * unsigned long int htonl(unsigned long int hostlong);

    * unsigned short int htons(unsigned short int hostshort);
    * unsigned long int ntohl(unsigned long int netlong);
    * unsigned short int ntohs(unsigned short int netshort);
    > 它们的含义很明确，比如htonl表示“host to network long”，即将长整型的主机字节序数据转化为网络字节序数据。
---
# socket相关函数和结构体
## 1）通用socket地址结构体
  * struct sockaddr 
    > 用于表示socket地址，定义如下:

        #include<bits/socket.h>
        struct sockaddr
        {
          sa_family_t sa_family;
          char sa_data[14];
        }
    * sa_family成员是地址族类型，可以取值：
      |协议族|地址族|描述|
      |-|-|-|
      |PE_UNIX|AF_UNIX|UNIX本地协议族|
      |PE_INET|AF_INET|TCP/IP协议族|
      |PE_INET6|AF_INET6|TCP/IPv6协议族|
  
    * sa_data成员用于存放socket地址值。

  * struct sockaddr_storage
    > 通用socket地址结构体，定义如下:

        #include <bits/socket.h>
        struct sockaddr_storage
        {
          sa_family_t sa_family;
          unsigned long int __ss_align;
          char __ss_padding[128-sizeof(__ss_align)];
        }
    * 这个结构体不仅提供了足够大的空间用于存放地址值，而且是内存对齐的（这是 __ss_align成员的作用）。

## 2）专用socket地址结构体
  > 专用socket地址结构体用于表述特定协议的socket地址。但所有专用socket地址类型的变量在实际使用时都需要转化为通用socket地址类型（强制转换即可），因为所有socket编程接口使用的地址参数的类型是通用的sockaddr。

  * struct sockaddr_un
    > UNIX本地域协议族专用socket地址结构体，定义如下

        #include <sys/un.h>
        struct sockaddr_un
        {
        sa_family_t sin_family;/*地址族：AF_UNIX*/
        char sun_path[108];/*文件路径名*/
        };
  * struct sockaddr_in
    > TCP/IPv4协议族专用socket地址结构体，定义如下

        struct sockaddr_in
        {
          sa_family_t sin_family;/*地址族：AF_INET*/
          u_int16_t sin_port;/*端口号，要用网络字节序表示*/
          struct in_addr sin_addr;/*IPv4地址结构体，见下面*/
        };
        struct in_addr
        {
          u_int32_t s_addr;/*IPv4地址，要用网络字节序表示*/
        };

  * struct sockaddr_in
    > TCP/IPv6协议族专用socket地址结构体，定义如下

        struct sockaddr_in6
        {
          sa_family_t sin6_family;/*地址族：AF_INET6*/
          u_int16_t sin6_port;/*端口号，要用网络字节序表示*/
          u_int32_t sin6_flowinfo;/*流信息，应设置为0*/
          struct in6_addr sin6_addr;/*IPv6地址结构体，见下面*/
          u_int32_t sin6_scope_id;/*scope ID，尚处于实验阶段*/
        };
        struct in6_addr
        {
          unsigned char sa_addr[16];/*IPv6地址，要用网络字节序表示*/
        };
## 3）IP地址转换函数
  > 点分十进制字符串表示的IPv4地址和网络字节序整数表示的IPv4之间的转换函数：

  * 旧版本
    * 头文件
      * #include<arpa/inet.h>
    * inet_addr函数：点分十进制字符串转网络字节序
      * in_addr_t inet_addr(const char *strptr);

    * inet_aton函数：点分十进制字符串转网络字节序
      * int inet_aton(const char*cp,struct in_addr *inp);

      * 说明
        > inet_aton调用失败返回0，成功返回1

    * inet_ntoa函数：网络字节序转点分十进制字符串
      * char *inet_ntoa(struct in_addr in);

      * 说明
        > inet_ntoa不可重入，内部使用静态变量存储转化结果。

  * 新版本
  
    * 头文件：
      * #include<arpa/inet.h>
    * int inet_pton(int af,const char*src,void *dst);
    * const char*inet_ntop(int af,const void*src,char *dst,socklen_t cnt);
    * 参数说明

      * af参数指定协议族，可以是AF_INET或者AF_INET6
      * inet_pton成功时返回1，失败则返回0并设置errno。
      * cnt参数指定目标存储单元的大小
      * inet_ntop成功时返回目标存储单元的地址，失败则返回NULL并设置errno。
        
    * 给字符串的IP地址分配内存时可使用这两个宏定义来指定

          #include<netinet/in.h>
          #define INET_ADDRSTRLEN 16
          #define INET6_ADDRSTRLEN 46

## 4）创建socket
  * socket函数

        #include<sys/types.h>
        #include<sys/socket.h>
        int socket(int domain,int type,int protocol)

    * domain参数
      > 指定协议族，可以取值：PF_UNIX | PF_INET | PF_INET6

    * type参数
      > 指定服务类型，可以取以下两值中的一个：
        * SOCK_STREAM（使用TCP协议）
        * SOCK_DGRAM（使用UDP协议）。
    * type还可以用或（|）操作符设置以下标志
        * SOCK_NONBLOCK（将socket设为非阻塞）
        * SOCK_CLOEXEC（fork调用创建子进程时在子进程中关闭该socket。）

    * protocol参数：通常设置为0，使用默认值。
    * socket函数返回值
      > 调用失败返回-1，并设置errno



## 5）命名socket
  * bind函数

    > 将一个socket与socket地址绑定称为给socket命名。bind函数可以将一个socket与socket地址绑定，定义如下：

        #include<sys/types.h>
        #include<sys/socket.h>
        int bind(int sockfd,const struct sockaddr *my_addr,socklen_t addrlen);
        
    * bind成功时返回0，失败则返回-1并设置errno。
    * 其中两种常见的errno是EACCES和EADDRINUSE，它们的含义分别是：

      * EACCES
        > 被绑定的地址是受保护的地址，仅超级用户能够访问。比如普通用户将socket绑定到知名服务端口（端口号为0～1023）上时，bind将返回EACCES错误。

      * EADDRINUSE
        > 被绑定的地址正在使用中。比如将socket绑定到一个处于TIME_WAIT状态的socket地址。

## 6）监听socket
  *  listen函数

          #include <sys/socket.h>
          int listen(int sockfd,int backlog);

      * backlog参数
        > 提示内核监听队列的最大长度。监听队列的长度如果超过backlog，服务器将不受理新的客户连接，客户端也将收到ECONNREFUSED错误信息。

      * 返回值
        > listen成功时返回0，失败则返回-1并设置errno。

## 7）接受连接
  * accept函数

        #include<sys/types.h>
        #include<sys/socket.h>
        int accept(int sockfd,struct sockaddr*addr,socklen_t *addrlen);

    * sockfd参数
      > 是执行过listen系统调用的监听socket。

    * addr参数
      > 用来获取被接受连接的远端socket地址，该socket地址的长度由addrlen参数指出。

    * accept返回值
      > accept成功时返回一个新的连接socket，该socket唯一地标识了被接受的这个连接。accept失败时返回-1并设置errno。

## 8）发起连接
  * connect函数

        #include<sys/types.h>
        #include<sys/socket.h>
        int connect(int sockfd,const struct sockaddr *serv_addr,socklen_t addrlen);

    * sockfd参数
      > 用于标识本次连接的socket。
    * serv_addr参数
      > serv_addr参数是要与其连接的socket地址。
    * addrlen参数
      > addrlen参数则指定serv_addr的长度。
    * 返回值
      > connect成功时返回0。connect失败则返回-1并设置errno。
    * 其中两种常见的errno是ECONNREFUSED和ETIMEDOUT，它们的含义如下：
      * ECONNREFUSED
        > 目标端口不存在，连接被拒绝。
      * ETIMEDOUT
        > 连接超时。


## 9）关闭连接
  * close函数

        #include<unistd.h>
        int close(int fd);
    * fd参数
      > 是待关闭的socket。
    * 说明
      > close系统调用并非总是立即关闭一个连接，而是将fd的引用计数减1。只有当fd的引用计数为0时，才真正关闭连接。

  * shutdown函数

        #include<sys/socket.h>
        int shutdown(int sockfd,int howto);

    * sockfd参数
      > 是待关闭的socket
    * howto参数
      > 决定了shutdown的行为，它可取以下值：

      | 可选值 | 含义 |
      |-|-|
      | SHUT_RD | 关闭sockfd上读的这一半，应用程序不能再针对socket文件描述符执行读操作，并且该缓冲区中的数据都被丢弃。 |
      | SHUT_WR | 关闭sockfd上写的这一半。sockfd的发送缓冲区中的数据会在真正关闭连接之前发送出去。应用程序不能再针对socket文件描述符执行写操作。这种情况下，连接处于半关闭状态。|
      | SHUT_RDWR | 同时关闭sock上的读和写。|

    * 返回值
      > shutdown成功时返回0，失败则返回-1并设置errno。


## 10）数据读写
  * recv/send函数
    > TCP数据读写的函数

        #include<sys/types.h>
        #include<sys/socket.h>
        ssize_t recv(int sockfd,void *buf,size_t len,int flags);

        ssize_t send(int sockfd,const void *buf,size_t len,int flags);

    * buf和len参数
      > 分别指定读缓冲区的地址和大小。

    * flags参数
      > 为数据收发提供了额外的控制，它可以是下表所示选项中的一个或几个的逻辑或。通常可以设置为0。

      |选项名|含义|send|recv|
      |-|-|-|-|
      |MSG_CONFIRM| 指示数据链层协议持续监听对方的响应，知道得到答复。它仅用于SOCK_DGRAM和SOCK_RAM类型的socket |Y|N|
      |MSG_DONTROUTE| 不查看路由表，直接将数据发送给本地局域网络内的主机。这表示发送者确切地知道目标主机就在本地网络。|Y|N|
      |MSG_DONTWAIT| 对socket的此次操作将是非堵塞的 |Y|Y|
      |MSG_MORE| 告诉内核应用程序还有更多的数据要发送，内核将超时等待的新数据写入TCP发送缓冲区后一并发送。这样可防止TCP发送过多的报文段，从而提高传输效率 |Y|N|
      |MSG_WAITALL| 读操作仅在读取到指定的数量的字节后才返回 |N|Y|
      |MSG_PEEK| 窥探读缓存中的数据，此次操作不会导致这些数据被消除 |N|Y|
      |MSG_OOB| 发送或接收紧急数据 |Y|Y|
      |MSG_NOSIGNAL| 往读端关闭的管道或者socket连接中写数据时不引发SIGPIPE |Y|N|


      * 返回值
        * recv返回已读取的长度，recv可能返回0，这意味着通信对方已经关闭连接了。recv出错时返回-1并设置errno。

        * send成功时返回实际写入的数据的长度，失败则返回-1并设置errno。

  * recvfrom/sendto函数
    > UDP数据读写的函数

        #include<sys/types.h>
        #include<sys/socket.h>
        ssize_t 
        recvfrom(int sockfd,void*buf,
                  size_t len,int flags,
                  struct sockaddr*src_addr,socklen_t *addrlen);

        ssize_t 
        sendto(int sockfd,const void*buf,
                size_t len,int flags,
                const struct sockaddr *dest_addr,socklen_t addrlen);
    * sockfd/buf/len/flags参数
      > 含义与recv/send函数一致

    * src_addr/dest_addr/addrlen
      > src_addr/dest_addr表示读写的目标的地址。addrlen表示socket地址的长度。

  * recvmsg/sendmsg函数
    > 通用数据读写函数

        #include<sys/socket.h>
        ssize_t recvmsg(int sockfd,struct msghdr *msg,int flags);
        ssize_t sendmsg(int sockfd,struct msghdr *msg,int flags);

    * flags参数同上。

    * msg参数
      > msghdr结构体类型的指针，msghdr结构体的定义如下

          struct msghdr
          {
            void *msg_name;/*socket地址*/
            socklen_t msg_namelen;/*socket地址的长度*/
            struct iovec *msg_iov;/*分散的内存块，见后文*/
            int msg_iovlen;/*分散内存块的数量*/
            void *msg_control;/*指向辅助数据的起始位置*/
            socklen_t msg_controllen;/*辅助数据的大小*/
            int msg_flags;/*复制函数中的flags参数，并在调用过程中更新*/
          };

      * msg_name参数
        > 指向一个socket地址结构变量。它指定通信对方的socket地址。对于面向连接的TCP协议，该成员没有意义，必须被设置为NULL。

      * msg_iov参数
        > 指向一个iovec结构体类型的数组，iovec结构体用于表示数据读写的内存块，的定义如下：
        
            struct iovec
            {
            void *iov_base;/*内存起始地址*/
            size_t iov_len;/*这块内存的长度*/
            };

      * msg_iovlenc参数
        > 指示msg_iov指向的数组的长度。

  
## 11）带外数据
  > 输层协议使用带外数据（out-of-band，OOB）来发送一些重要的数据，如果通信一方有重要的数据需要通知对方时，协议能够将这些数据快速地发送到对方。

  * sockatmark函数
    > sockatmark判断sockfd是否处于带外标记，即下一个被读取到的数据是否是带外数据。如果有带外数据，我们就可以利用带MSG_OOB标志的recv调用来接收带外数据。

        #include<sys/socket.h>
        int sockatmark(int sockfd);

      * 返回值
        * 有带外数据时sockatmark返回1
        * 没有带外数据时sockatmark返回0

## 12）地址信息函数
  * getsockname/getpeername函数

        #include<sys/socket.h>
        int getsockname(int sockfd,struct sockaddr*address,socklen_t *address_len);
        int getpeername(int sockfd,struct sockaddr*address,socklen_t *address_len);

    * 说明
      > getsockname获取sockfd对应的本端socket地址；getpeername获取sockfd对应的远端socket地址。
    * 返回值
      > 成功时返回0，失败返回-1并设置errno。

  * gethostbyaddr/gethostbyname函数

        #include<netdb.h>
        struct hostent*gethostbyname(const char *name);
        struct hostent*gethostbyaddr(const void *addr,size_t len,int type);

    * name参数
      > 指定目标主机的主机名
    * addr参数和len参数
      > addr参数指定目标主机的IP地址，len参数指定addr所指IP地址的长度。
    * type参数
      > 指定addr所指IP地址的类型，其合法取值包括AF_INET（用于IPv4地址）和AF_INET6（用于IPv6地址）。
    * 返回值
      > 这两个函数返回的都是hostent结构体类型的指针，hostent结构体的定义如下：

          #include<netdb.h>
          struct hostent
          {
          char *h_name;/*主机名*/
          char**h_aliases;/*主机别名列表，可能有多个*/
          int h_addrtype;/*地址类型（地址族）*/
          int h_length;/*地址长度*/
          char**h_addr_list/*按网络字节序列出的主机IP地址列表*/
          };


## 13）配置socket选项
  * getsockopt/setsockopt函数

        #include<sys/socket.h>
        int getsockopt(int sockfd,int level,int option_name,void*option_value,socklen_t *restrict option_len);
        int setsockopt(int sockfd,int level,int option_name,const void *option_value,socklen_t option_len);

    * sockfd参数
      > 指定被操作的目标socket。

    * level参数
      > 指定要操作哪个协议的选项（即属性），比如IPv4、IPv6、TCP等。

    * option_name参数
      > 指定要操作的选项的名字。
    
    * option_value和option_len参数
      > 分别是被操作选项的值和长度
    
    * 可配置的选项如下表

    | level | option_name | value_type | 说明 |
    |-|-|-|-|
    |SOL_SOCKET(通用socket选项，与协议无光)| SO_DEBUG|int| 打开调试信息|
    |SOL_SOCKET| SO_REUSEADDR|int| 重用本地地址|
    |SOL_SOCKET| SO_TYPR|int| 获取socket类型|
    |SOL_SOCKET| SO_ERROR|int| 提取并清除socket错误状态|
    |SOL_SOCKET| SO_DONTROUTE|int| 不查看本地路由表|
    |SOL_SOCKET| SO_RCVBUF|int| TCP接收缓冲区的大小|
    |SOL_SOCKET| SO_SNDBUF|int| TCP发送缓冲区的大小|
    |SOL_SOCKET| SO_OOBINLINE|int| 接收到的带外数据将存留在普通数据的输入队列中（在线存留），此时我们不能使用带MSG_OOB标志的读操作来读取带外数据（而应像读取普通数据那样读取带外数据。）|
    |SOL_SOCKET| SO_LINGER|linger| 诺有数据待发送，则延时关闭|
    |SOL_SOCKET| SO_RCVLOWAT|int| TCP接收缓冲区低水位标记|
    |SOL_SOCKET| SO_SNDLOWAT|int| TCP发送缓冲区低水位标记|
    |SOL_SOCKET| SO_RCVTIMEO|timeval| 接收数据超时|
    |SOL_SOCKET| SO_SNDTIMEO|timeval| 发送数据超时|
    |IPPROTO_IP(IPv4选项)| IP_TOS|int| 服务类型|
    |IPPROTO_IP| IP_TTL|int| 存活时间|
    |IPPROTO_IPV6(IPv6选项)| IPV6_NEXTHOP|sokeaddr_in6| 下一条IP地址|
    |IPPROTO_IPV6| IPV6_RECVPKTINFO|int| 接收分片信息|
    |IPPROTO_IPV6| IPV6_DONTFRAG|int| 禁止分片|
    |IPPROTO_IPV6| IPV6_RECVTCLASS|int| 接收通信信息|
    |IPPROTO_TCP(TCP选项)| TCP_MAXSRG|int| TCP最大报文段大小|
    |IPPROTO_TCP| TCP_NODELAY|int| 禁止Nagle算法|
---

# 高级IO函数
## 1）管道
  * pipe函数

        #include<unistd.h>
        int pipe(int fd[2]);
    * pipe函数可用于创建一个管道，以实现进程间通信。
    * 返回值
      > 该函数成功时返回0，如果失败，则返回-1并设置errno。
    * fd[2]参数
      > 往fd[1]写入的数据可以从fd[0]读出。并且，fd[0]只能用于从管道读出数据，fd[1]则只能用于往管道写入数据。

  * socketpair函数

        #include<sys/types.h>
        #include<sys/socket.h>
        int socketpair(int domain,int type,int protocol,int fd[2]);

    * domain参数和protocol参数
      > 同socket函数，domain只能使用UNIX本地域协议族AF_UNIX。
    * fd[2]参数
      > socketpair函数创建双通道管道，故fd[0]和fd[1]均可读可写。
    * 返回值
      > socketpair成功时返回0，失败时返回-1并设置errno。
    
## 2）dup/dup2函数

    #include<unistd.h>
    int dup(int file_descriptor);
    int dup2(int file_descriptor_one,int file_descriptor_two);
  * dup函数创建一个新的文件描述符，该新文件描述符和原有文件描述符file_descriptor指向相同的文件、管道或者网络连接。
  * 返回值
    * 失败时返回-1并设置errno，否则返回一个文件描述符。
    * dup返回的文件描述符总是取系统当前可用的最小整数值。
    * dup2返回一个不小于file_descriptor_two的整数值。

## 3）sendfile函数
    #include<sys/sendfile.h>
    ssize_t sendfile(int out_fd,int in_fd,off_t *offset,size_t count);

  * in_fd参数
    > 是待读出内容的文件描述符
  * out_fd参数
    > 是待写入内容的文件描述符
  * offset参数
    > 指定从读入文件流的哪个位置开始读，如果为空，则使用读入文件流默认的起始位置。
  * count参数
    > 指定在文件描述符in_fd和out_fd之间传输的字节数。
  * 返回值
    * sendfile成功时返回传输的字节数，失败则返回-1并设置errno。
  * 注
    > in_fd必须是一个支持类似mmap函数的文件描述符，即它必须指向真实的文件，不能是socket和管道；而out_fd则必须是一个socket。

## 4）readv/writev函数

    #include<sys/uio.h>
    ssize_t readv(int fd,const struct iovec *vector,int count);
    ssize_t writev(int fd,const struct iovec *vector,int count);
  * vector参数
    > 类型是iovec结构数组。
  * count参数
    > 是vector数组的长度
  * 返回值
  > readv和writev在成功时返回读出/写入fd的字节数，失败则返回-1并设置errno。

## 5）mmap/munmap函数
  > 用于申请一段内存空间，可以将文件映射到其中。

    #include<sys/mman.h>
    void*mmap(void *start,
              size_t length,
              int prot,
              int flags,
              int fd,
              off_t offset);
    int munmap(void *start,size_t length);

  * start参数
    > start参数允许用户使用某个特定的地址作为这段内存的起始地址。如果它被设置成NULL，则系统自动分配一个地址。
  * length参数
    > 指定内存段的长度。
  * prot参数
    > 用来设置内存段的访问权限。取以下几个值的按位或：
    * PROT_READ
      > 内存段可读
    * PROT_WRITE
      > 内存段可写
    * PROT_EXEC
      >内存段可执行
    * PROT_NONE
      > 内存段不能被访问。
  * offset参数
    > 设置从文件的何处开始映射。

  * flags参数
    > 控制内存段内容被修改后程序的行为。可以被设置为下表中的常用值的按位或（其中MAP_SHARED和MAP_PRIVATE不能同时指定）

    |常用值|含义|
    |-|-|
    |MAP_SHARED|在进程间共享这段内存。对该内存段的修改将反映到被映射的文件中|
    |MAP_PRIVATE|内存段为调用进程所有。对该内存段的修改将不会反映到被映射的文件中|
    |MAP_ANONYMOUS|这段内存不是从文件映射而来，其内容初始化为0，这种情况下，mmap函数的最后两个参数将被忽略|
    |MAP_FIXED|内存段必须位于start参数指定的地址处。start必须是内存页面大小的整倍数|
    |MAP_HUGETLE|按照“大内存页面”来分配内存空间|

  * 返回值
    > mmap函数成功时返回指向目标内存区域的指针，失败则返回MAP_FAILED（（void*）-1）并设置errno。munmap函数成功时返回0，失败则返回-1并设置errno。


## 6）splice函数
  > splice函数用于在两个文件描述符之间移动数据，也是零拷贝操作。fd_in和fd_out必须至少有一个是管道（socket或其他）文件描述符。

    #include<fcntl.h>
    ssize_t splice(int fd_in,loff_t*off_in,int fd_out,loff_t *off_out，
    size_t len,unsigned int flags);

  * fd_in参数
    > 是待输入数据的文件描述符。如果fd_in是一个管道文件描述符，那么off_in参数必须被设置为NULL。

  * off_in参数
    > 表示从输入数据流的何处开始读取数据。若off_in被设置为NULL，则表示从输入数据流的当前偏移位置读入;

  * fd_out/off_out参数的含义与fd_in/off_in相同

  * len参数
    > 指定移动数据的长度

  * flags参数
    > 控制数据如何移动，它可以被设置下表中的某些值的按位或。
    
    |常用值|含义|
    |-|-|
    |SPLICE_F_NONBLOCKS|非堵塞操作|
    |SPLICE_F_MORE|给内核一个提示，后续还会调用splice|
  
  * 返回值
    > splice函数调用成功时返回移动字节的数量。它可能返回0，表示没有数据需要移动。失败时返回-1并设置errno。


## 7）tee函数
  > tee函数在两个管道文件描述符之间复制数据，也是零拷贝操作。它不消耗数据，因此源文件描述符上的数据仍然可以用于后续的读操作。

    #include<fcntl.h>
    ssize_t tee(int fd_in,int fd_out,size_t len,unsigned int flags);
  * 该函数的参数的含义与splice相同
  * 返回值同splice

## 7）fcntl函数
  > 提供了对文件描述符的各种控制操作。

    #include<fcntl.h>
    int fcntl(int fd,int cmd，...);

  * cmd参数
    > 指定执行何种类型的操作。

  * 第三个参数
    > 根据操作类型的不同，该函数可能还需要第三个可选参数arg。

  * fcntl函数支持的常用操作及其参数和返回值

  |操作分类|操作|含义|arg的类型|返回值|
  |-|-|-|-|-|
  |复制文件描述符|F_DUPFD|创建一个新的文件描述符，其值大于arg|long|新创建的文件描述符的值|
  |复制文件描述符|F_DUPFD_CLOEXEC|与F_DUPFD相似，但为创建的文件描述符设置close-on-exec标志|long|新创建的文件描述符的值|
  |获取和设置文件描述符的标志|F_GETFD|获取该fd上的标志|无|该fd上的标志|
  |获取和设置文件描述符的标志|F_SETFD|设置该fd上的标志|long|0|
  |获取和设置文件描述符的状态标志|F_GETFL|获取该fd的状态标志，这些标志包括由ioen系统调用设置的标志（O_APPEND/O_CREAT等）和访问模式（O_RDONLY/O_RDWR等）|void|fd的状态标志|
  |获取和设置文件描述符的状态标志|F_SETFL|设置fd的状态标志。但部分标志是不能被修改（比如访问模式标志）|long|0|
  |管理信号|F_GETOWN|获得SIGIO和SIGURG信号的宿主进程的PID或进程组的ID|无|信号的宿主的PID或者进程组的组ID|
  |管理信号|F_SETOWN|设置SIGIO和SIGURG信号的宿主进程的PID或进程组的ID|long|0|
  |管理信号|F_GETSIG|获取当应用程序被通知fd可读或可写时，是哪个信号通知该事件的|无|信号值，0表示SIGIO|
  |管理信号|F_SETSIG|设置当应用程序被通知fd可读或可写时，系统应该触发哪个信号来通知应用程序|long|0|





