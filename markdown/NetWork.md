# 1. TCP/IP协议族
## 1）协议类型：
* 链路层
  * RARP协议
  * ARP协议
* 网络层  
  * ICMP协议（ping）
    >用于检测网络连接
  * IP
* 传输层
  * TCP（基于IP）
  * UDP（基于IP）
* 应用层
  * FTP
  * DNS协议
  * HTTP/HTTPS（基于TCP/UDP）
---

## 2）ARP协议
> ARP协议实现任意网络层地址到任意物理地址的转换。
> 数据链路层使用物理地址寻址一台机器，因此网络层必须先将目标机器的IP地址转化成物理地址，才能使用数据链路层的服务，这就是ARP协议的用途。（将IP地址转换成MAC地址）
### 工作原理：
  * 主机向自己所在的网络广播一个ARP请求，改请求包含目标机器的网络地址。此网络上的其他机器都将收到这个请求，但只有被请求的目标机器（或者包含目标机器的路由的机器）会回应一个ARP应答，其中包含自己的物理地址。

  * 通常ARP维护一个高速缓存，其中包含经常访问（比如网关地址）或最近访问的机器的IP地址到物理地址的映射。这样就避免了重复的ARP请求，提高了发送数据报的速度。
###  其他
  * Linux下可以使用arp命令来查看和修改ARP高速缓存。
---

## 3）IP协议
>IP协议是TCP/IP协议族的动力，它为上层协议提供无状态、无连接、不可靠的服务。

  * IP数据报存在乱序和重复的数据报。（TCP协议层会处理乱序的和重复的报文段。）

  * 不可靠：不能保证数据报准确地到达接收端，只承诺尽最大的努力（发送失败时会收到从路由或者接收端发送过来的ICMP的错误消息）。
  * 分片和重组：一段IP数据报可能被切成多段IP分片（可能发生在发送端或者中转路由上）发送出去，在接收端IP模块会将IP分片重组成IP数据报。
###  路由机制
 > IP模块实现数据报路由的核心数据结构是路由表。这个表按照数据报的目标IP地址分类，同一类型的IP数据报将被发送到相同的下一跳路由器或者目标机器。
  * 路由表可以通过动态更新或者静态更新。大型的路由器通常通过BGP、RIP、OSPF等协议来发现路径，并更新自己的路由表。
###  相关命令
 * netstat
 * route命令
---

## 4）TCP协议
* 提供可靠的、面向连接的和基于流的服务。

* 可靠性：使用超时重传、数据确认等方式来确保数据被正确地发送到目的端。
* 发送应答机制：发送端发送的每个TCP报文都必须得到接收方的应答，才认为这个TCP报文段传输成功。
* 超时重传机制：发送端在发出一段报文之后启动定时器，如果在定时时间内未收到应答，它将重发该报文。
* 接收通告窗口：TCP的数据报的报头里用16位空间来表述己方的接收缓冲区可接收多少字节的数据。
* TCP连接是双工的，一条TCP连接里有两条数据通道，一条数据通道为己方发送缓冲区流向对方接收缓冲区；另一条为对方发送缓冲区流向己方接收缓冲区。它允许两个方向的数据通道可独立被关闭。
* 内核为此存储相关的数据（TCP连接状态，TCP各类定时器，发送缓冲区，接收缓冲区）
* 延迟确认数据：接收到对方发送过来的报文时，不急对报文做应答，而是延迟一段时间查看本端是否有数据需要发送，如果有，则与确认信息一起发出。
* Nagle算法：要求一个TCP连接通道双方在任意时刻都最多只能发送一个未被确认的TCP报文，在该报文段的确认到达之前不能发送其他TCP报文。另一方面，发送方在等待确认的同时收集和堆积本端需要发送的数据，并在确认到来时以一个TCP报文段将它们全部发出（将缓冲区里堆积的数据和对接收到的数据的确认信号一起发出去）。

### TCP报头结构
  ![TCP建立和关闭时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_TCP_1.jpg)

### TCP状态转移
  ![TCP建立和关闭时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_TCP_2.jpg)
### TCP连接的建立和关闭
 * 超时重连策略：在建立连接的时候，会向对方发送同步报文，如果在一段时间内有没接收到同步报文的应答，TCP将尝试重连。在5次重连失败后TCP模块将放弃连接并通知应用程序。
 * TCP建立和关闭时序图
   ![TCP建立和关闭时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_TCP_Sequence.jpg)

### socket 与 TCP/IP协议的关系：
   * > socket是通用网络编程接口，实现TCP/IP协议族的系统调用API（应用程序编程接口）。
### 相关命令：
* tcpdump
---

## 5）UDP协议
* 提供不可靠、无连接和基于数据报的服务。

* 数据可能在传输的过程丢失。
* 每个UDP数据报都有一个长度，接收端必须以该长度为最小单位将其所有内容一次性读出，否则数据将被截断。
* 当一个UDP数据被发送出去后，UDP内核缓冲区中的该数据报将被丢弃。如需重发，只能重新从用户态将数据拷贝到内核态的缓冲区。
---

# 2. HTTP协议
  * 无状态协议

## 1）HTTP报文结构
  * HTTP报文分成两种报文：请求报文和响应报文。
  * HTTP报文大致可分为报文首部和报文体两部分。报文首部由多行首部字段组成（每个首部字段均以CR+LF为结尾）。此外报文体也可以为空。对于请求报文，报文首部由请求行和多个首部字段组成；对于响应报文，报文首部由状态行和多个首部字段组成。
  * 报文结构图：
    ![报文结构图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_Header1.jpg)
  * 请求报文结构：
    ![请求报文结构](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_Header2.jpg "请求报文结构")
  * 响应报文结构：
    ![响应报文结构](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_Header3.jpg "响应报文结构")
  * 报文示例：
    ![报文结构图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_Instance1.jpg "HTTP报文实例")

### 请求行
  > 包含用于请求的方法、请求URI、HTTP版本。
### 状态行 
  > 包含表明响应结果的状态码。
### 首部字段
  > 用于表示请求和响应的各种条件和属性。
### 报文主体
  > 报文的实体数据。
---
## 2）请求方法
  ![请求方法](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_RequestMethod.jpg)
---
## 3）响应状态码解释
  > HTTP状态码用于表示服务器对HTTP请求的处理结果的状态。
  * 状态码可分为五类：
    | - | 类别 |  解释 |
    |-|-|-|
    | 1XX | 信息性状态码 | 接收的请求正常进行处理中 |
    | 2XX | 成功状态码 | 请求正常处理完毕 |
    | 3XX | 重定向状态码 | 需要进行附加操作以完成请求 |
    | 4XX | 客户端错误状态码 | 服务器无法客户端的处理请求 |
    | 5XX |服务器错误状态码 | 服务器处理请求过程中内部出现错误 |
### 常见的2XX状态码解释
  * 200 OK
    > 表示码请求在服务端被正常处理完毕。
  * 204 No Content
    > 该状态码表示请求以被服务器处理完毕，但返回的响应报文中不含实体的主题部分。
  * 206 Partial Content
    > 该状态码表示客户端进行了范围请求，且服务端已成功处理了这部分请求。

### 常见的3XX状态码解释
  * 301 Moved Permanently
    > 永久性重定向。该状态码表示请求的资源已被分配了新的URI。
  * 302 Found 
    > 临时重定向。该状态码表示请求的资源已被分配了新的URI，希望用户（本次）能使用新的URI访问。
  * 303 See Other
    > 该状态码表示由于请求对应的资源存在着另一个URI，应使用GET方法定向请求资源。
  * 304 Not Modified
    > 该状态码表示客户端请求的资源没有更变，可以使用本地已经缓存的资源。
  * 307 Temporary Redirect
   > 临时重定向，该状态码与302 Found有着相同的含义，但不会从POST请求变成GET请求。

### 常见的4XX状态码解释
  > 4XX的响应结果表明客户端方是本次请求处理失败的原因所在。
  * 400 Bad Request
   > 该状态码表示请求报文中存在语法错误。
  * 401 unauthorized
   > 该状态码表示要请求的资源需要先通过HTTP认证的认证信息。
  * 403 Forbidden
   > 该状态码表示对请求资源的访问被服务器拒绝。
  * 404 Not Found
   > 该状态码表示请求资源不存在。
### 常见的5XX状态码解释 
  > 5XX的响应结果表明服务器本身发生了错误。
  * 500 Internal Server Error
   > 该状态码表明服务器在执行请求时发生了错误。
  * 503 Internal Server Error
   > 该状态码表明服务器暂时处于超负载或正在进行停机维护。
---

## 4）首部字段解释
  * HTTP首部字段由首部字段名和字段值构成的，中间用冒号“:”分割。格式如下：
    > 首部字段名: 字段值
  * HTTP首部字段根据实际用途可分为以下四种类型
    * 通用首部类型
      > 请求报文和响应报文都会用到的首部

      ![请求方法](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_CommonHeaderField.jpg)

    * 请求首部字段
      > 从客户端向服务器发送请求时使用的首部。 

      ![请求方法](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_RequestHeaderField.jpg)

    * 响应首部字段
      > 从服务器向客户端返回响应报文时使用的首部 

      ![请求方法](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_RespondHeaderField.jpg)

    * 实体首部字段
      > 针对请求报文和响应报文的实体部分使用的首部。 

      ![请求方法](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTP_BodyHeaderField.jpg)
---
## 5）基于HTTP的功能追加协议
### AJAX
  > AJAX技术主要用于更新局部页面。
  > 客户端定时异步请求到服务端，以获得局部数据，而后使用该局部数据来更新局部页面。
### Comet
  > Comet技术用于实现服务端主动推送功能。
  > 客户端向服务端请求资源时，如果服务端没有内容则会先将响应置于挂起状态，当服务端有内容更新时，再返回该响应。
### websocket
  > WebSocket协议用于建立基于HTTP协议的全双工通信。
  > WebSocket协议使用一次HTTP报文来建立连接，建立成功后，通信协议将从HTTP协议换成WebSocket协议。（使用“Upgrade:websocket”首部字段来通告服务器将通信协议切换成websocket协议）
  * WebSocket通信时序图：
    ![HTTPS通信时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTPS_WebSocketSequence.jpg)
---
## 6）HTTPS
### HTTP协议的不足：
  * 通信使用明文（不加密），内容容易被窃听。
  * 不验证通信方的身份，因此有可能遭遇伪装。
  * 无法证明报文的完整性，所以有可能已遭篡改。

### HTTPS使用SSL/TLS进行信息加密
  * HTTPS并非是应用层的一种新协议，只是HTTP通信接口部分用SSL和TLS协议替代。HTTPS其实就是身披SSL协议这层外壳的HTTP。
  * HTTPS会先使用SSL/TLS建立安全连接通道，而后在该通道上进行HTTP通信。
  * HTTP+加密+认证+完整性保护=HTTPS

### HTTPS通信过程
  1. 客户端通过发送Client Hello报文开始SSL通信。报文中包含客户端支持的SSL的指定版本、加密组件（Cipher Suite）列表（所使用的加密算法及密钥长度等）。
  2. 服务器可进行SSL通信时，会以Server Hello报文作为应答。和客户端一样，在报文中包含SSL版本以及加密组件。服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的。
  3. 之后服务器发送Certificate报文。报文中包含公开密钥证书。
  4. 最后服务器发送Server Hello Done报文通知客户端，最初阶段的SSL握手协商部分结束。（客户端收到公开密钥证书后，会访问证书颁发机构核实该证书）
  5. SSL第一次握手结束之后，客户端以Client Key Exchange报文作为回应。报文中包含通信加密中使用的一种被称为Pre-master secret的随机密码串。该报文已用步骤3中的公开密钥进行加密。
  6. 接着客户端继续发送Change Cipher Spec报文。该报文会提示服务器，在此报文之后的通信会采用Pre-master secret密钥加密。
  7. 客户端发送Finished报文。该报文包含连接至今全部报文的整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确解密该报文作为判定标准。
  8. 服务器同样发送Change Cipher Spec报文。
  9. 服务器同样发送Finished报文。
  10. 服务器和客户端的Finished报文交换完毕之后，SSL连接就算建立完成。当然，通信会受到SSL的保护。从此处开始进行应用层协议的通信，即发送HTTP请求。
  11. 应用层协议通信，即发送HTTP响应。在使用HTTP报文通信时，该报文会被进行对称性加密。发送数据时会附加一种叫做MAC（Message Authentication Code）的报文摘要。MAC能够查知报文是否遭到篡改，从而保护报文的完整性。
  12. 最后由客户端断开连接。断开连接时，发送close_notify报文。
  * HTTPS通信时序图：
    ![HTTPS通信时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTPS_Sequence1.jpg)

### HTTPS通信概括：
  1. 客户端与服务端之间协商决定加密组件。
  2. 服务端发送公开密钥证书给客户端，客户端访问该证书的证书颁发机构，确认该证书是否有效。
  3. 客户端生成pre-master secret（随机数，该随机数为对称加密的初始密钥）。客户端用服务端的公钥对pre-master secret加密（使用非对称性加密）后发送给服务端。
  4. 服务端用私钥解密后得到pre-master secret。
  5. 至此客户端和服务端可以进行安全的HTTP报文通信（双方的通信数据将进行对称性加密，pre-master secret将作为对称加密的初始密钥）。
  6. 通信结束，断开连接。
  * 时序图：
    ![HTTPS通信时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_HTTPS_Sequence2.jpg)

### HTTPS的缺点
  * 加密和解密处理会大量消耗CPU和内存等资源，导致处理速度变慢。
  * 需要购买证书，比使用HTTP付出更高的开销。
---

## 7）DNS服务器
> DNS是一套分布式的域名服务系统。每个DNS服务器上都存放着大量的机器名（域名）到IP地址的映射，并且是动态更新的。众多的网络客户端程序都使用DNS协议来向DNS服务器查询目标主机的IP地址。
  * Linux使用/etc/resolv.conf文件来存放DNS服务器的IP地址。
---

# 3. 加密
  * 对称加密
  * 非对称加密
## 1）对称性加密
  > 对称性加密体制也成为单钥或私密钥体制，其加密和解密密钥相同。对称密码算法按加密方法的不同可分流密码算法类和分组密码算法类。
### 流密码算法
  > 流密码也称为序列密码，是将明文消息按照字符逐位地加密，连续地处理输入的明文，即一次加密一个比特或一个字节。
### 分组密码算法
  > 分组密码是将明文按照分组分成固定长度的块，用同一密钥和算法对每个块加密，每个输入块加密后得到一个固定长度的密文输出块。
### 常见的对称密码算法：
  * DES
  * 3DES
  * IDEA
  * AES
  * RC5

### 对称密码算法优点：
* 算法简单
* 加密速度快
* 加密效率高
* 适合加密大量数据
* 明文长度与密文长度相等

## 2）非对称加密
  > 非对称加密体制又称双钥或公钥密码体制，其加密密钥和解密密钥不同，可以公开的密钥称为公钥，必须保密的密钥称为私钥。公钥密码算法计算速度较慢，加密数据的速率较低，通常用于密钥管理和数字签名。

### 常见的非对称加密算法：
  * RSA
  * ECC
  * ElGamal

## 3）哈希函数和数字签名
### 哈希算法
  > 哈希函数是进行消息认证的基本方法，其主要用途是消息完整性检测和数字签名。哈希函数接受一个消息做为输入，产生一个称为哈希值（也可称为散列值、消息摘要）的输出。哈希函数是将有限任意长度比特串映射为固定长度的串。常用的哈希算法为MD5和SHA。

### 消息鉴别码（MAC）
  > 利用密钥来生成一个固定长度的MAC值，并将该数据块附加在消息之后。消息和MAC一起发送给接收方。接收方对收到的消息用相同的密钥进行相同的计算得出新的MAC，并将接收的MAC与其计算出的MAC进行比较。诺接收到的MAC与计算的MAC相等，则接收方可以相信消息未被篡改。

### 数字签名
  > 数字签名是指附加在数据单元上的一些数据，或是对数据单元所做的密码变换，这种数据或变换能被数据单元的接收者用于确认数据单元来源和数据单元的完整性，防止被人伪造。RSA、ElGamal等非对称加密算法可用于实现数字签名。

  * **数字签名有以下特征：**
    * 不可伪造性：如果不知道签名者的私钥，敌手很难伪造一个合法的数字签名。
    * 不可否认性：对普通数字签名，任何人可用签名者公钥验证签名的有效性。由于签名的不可伪造性，签名者无法否认自己的签名。
    * 消息完整性：可以防止消息被篡改。
  
  * **RSA算法和哈希函数配合的数字签名过程：**
    ![HTTPS通信时序图](https://raw.githubusercontent.com/JinVei/Document/master/img/NetWork_Encryption_DigitalSignature.jpg)
---

### 参考：
1. TCP/IP部分
   * 《Linux高性能服务器编程》—游双
2. HTTP部分
   * 《图解HTTP》—上野宣
3. 加密部分
   * 《信息安全技术》—吴世忠；李斌；张晓菲；沈传宁；李淼；
---
