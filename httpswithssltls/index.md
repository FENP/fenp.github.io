# 简要概述TCP/IP模型、SSL/TLS/HTTPS协议以及SSL证书


> 原文链接：[A brief overview of the TCP/IP model, SSL/TLS/HTTPS protocols and SSL certificates](https://medium.com/jspoint/a-brief-overview-of-the-tcp-ip-model-ssl-tls-https-protocols-and-ssl-certificates-d5a6269fe29e)
>
> 在本文中，我们将了解基于SSL/TLS协议的Web加密和Web安全如何工作。我们还将了解SSL证书的结构，以及如何生成一个供自己使用的自签名证书。
>
> 我现在正在写一篇关于如何用Go（golang）创建一个安全的HTTP服务器的文章。我想没有比这更好的机会来探讨这个关于Web安全的广泛话题了。
>
> 本文，我们将广泛讨论SSL/TLS通信、HTTP协议、因特网数据包、SSL证书和其他web/因特网的基本特征。虽然它们并没有在我们日常的网络开发生活中出现，但是我们绝对应该了解它们。

## 因特网是如何工作的？

我可以用一句话回答这个问题，或者说，我们也可以在这谈论几个小时。不过我想让事情变的简单易懂。

因特网就是分布在世界各地，将不同设备互相连接起来的电网线。即使您没有看到实体的电线，但是您依旧可以想象您的智能手机通过虚拟电线连接到WiFi路由器，让您与位于您几英里之外的谷歌服务器对话。

当您想通过因特网向和您的个人设备连接的设备发送一些**数据**时，您需要遵守一些规则。所有在因特网上传递的**数据**都是由**0**和**1**组成的**二进制数据**。

1. 首先，用于发送数据的应用程序需要与设备上的硬件通信，以便在因特网上发送数据。同样，设备上的硬件需要与在因特网上实际传输数据的设备（如**WiFi路由器**）进行通信。
2. 当您给一个人寄信时，您会在信上标明收信人的地址，这样信就能送到正确的人手中。同样地，当您在因特网上发送一些数据时，您需要知道最终接收这些数据的设备的地址。
3. 当一个网络应用程序在设备中启动（如**Web浏览器**或**Web服务器**）来传输或接收数据时，它将在一个[**端口**](https://en.wikipedia.org/wiki/Port_(computer_networking))上监听因特网数据，如**8080**。发送方需要知道接收方设备的端口，以便数据被正确的应用程序接收，而不是其他可能将其滥用的应用程序。

------

:bulb:*这些端口是由操作系统随机分配的，但是我们可以迫使操作系统将应用程序绑定到一个特定的端口上。*

------

4. 接收到数据的设备将无法理解它，除非它事先知道它在看什么。这个数据可能是一张图片、一首歌、一条简单的文本消息或者一个HTML文档。因此我们需要告诉接收者数据的类型，以便它可以正确理解数据。

这四个步骤描述了在因特网上进行**数据传输**的**通信模型**。其中每一步都由一个**协议**定义。根据协议的类型，每一步都会对数据进行相应的格式化，这样接收方就能够正确地提取数据了。

------

:bulb:*根据数据传输的用例，在一个特定的模型中可能有更多的步骤。您可以阅读[OSI模型](https://en.wikipedia.org/wiki/OSI_model#Layer_architecture)。任何人都可以用这个模型在因特网或局域网（LAN）上传输数据。*

------

如果我们仔细审视这些步骤，我们会发现它们形成了一个管道：来自一个步骤的数据被发送到另一个步骤进行下一步处理，直到它在因特网上传递。然而，**层**可能是描述这些步骤的最合适的词汇，我们马上就会知道**为什么**。

## TCP/IP通信模型

传输控制协议（**Transmission Control Protocol**， TCP）是我们前面看到的通信模型中的**传输层**协议，而因特网协议（**Internet Protocol**，IP）是**网络层**协议。

这些协议共同推动了大部分的因特网通信。此时此刻，您的浏览器正在使用 TCP/IP 模型从服务器加载此网页。因此，这些协议共同构成了因特网协议套件 （[**internet protocol suite**](https://en.wikipedia.org/wiki/Internet_protocol_suite)），如下图所示（图源[维基百科](https://en.wikipedia.org/wiki/Internet_protocol_suite)）。

![因特网协议套件1](HTTPwithSSLTLS/internet_protocol_suite1.png)

![因特网协议套件2](HTTPwithSSLTLS/internet_protocol_suite2.png)

在上述通信模型中（左侧），**应用层**从源（如内部存储或 RAM）获取数据并用特定协议（如[HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)）的一些**标头**将其包装。这会创建一个 HTTP 协议的数据包，并且可以被像 Web 浏览器一样理解它的应用程序读取。

这个**HTTP包**将会被发送到**传输层**。传输层使用 **TCP 协议标头**以及**源端口**和**目标端口**包装数据包。源端口是传输数据的应用程序的端口（如 Web 浏览器），目标端口是运行在接收端的应用程序接收数据的端口。 **TCP 包**被网络层接收，它用源和目标 IP 地址以及 **IP 协议标头**将其包装。源IP地址是发送方设备的IP地址，目标地址是接收方设备的IP地址。

------

:bulb:*IP 地址是分配给因特网上设备的唯一编号。您可以阅读有关  [**IPV4**](https://en.wikipedia.org/wiki/IPv4) 和  [**IPV6**](https://en.wikipedia.org/wiki/IPv6)  IP 地址格式获得更多信息。设备的 IP 地址可以在数据传输之前就知道，也可以使用 HTTP 标头中包含的[**域名**](https://en.wikipedia.org/wiki/Domain_Name_System)从[**DNS**](https://www.youtube.com/watch?v=mpQZVYPuDGU)服务器解析。*

------

一旦数据包被印上IP协议，它就成为了可以通过因特网传输的**网络包**。这是虚拟层的最后一步（*（指）由计算机程序处理（的步骤）*）。

最后的**链路层**是设备上的**物理层**。这是硬件部分，例如设备上的网卡（[NIC](https://en.wikipedia.org/wiki/Network_interface_controller)），它接收数据包并将源和目标MAC地址添加到数据包中。

------

:bulb:*媒体访问控制 （[MAC](https://en.wikipedia.org/wiki/MAC_address)） 地址是设备制造商提供的硬件部分的独特地址。使用 MAC 地址，两个设备之间的通信成为了可能*

------

一旦数据包被源和目标的MAC地址以及一些**数据传输层的标头**（例如以太网：[*Ethernet*](https://en.wikipedia.org/wiki/Ethernet)）标记，它就可以被发送到因特网通信设备，如WiFI路由器或者[卫星天线](https://en.wikipedia.org/wiki/Satellite_dish) 。它们将负责数据包在因特网上的传输。

一旦数据包被接收方（上图的右侧）收到，它将被像洋葱一样去皮，直到原始数据（HTML 文档）被恢复。

一次一层（地提取数据），通过查看该层的协议，数据可以在没有任何误解的情况下被成功提取。原始数据提取后，可被正确的应用程序使用。

例如，如果服务器使用HTTP协议发送一个HTML页面，Web浏览器完全能够理解 HTTP协议的数据包并从中提取 出HTML 页面。然后，浏览器将在屏幕上呈现这个HTML文档。

------

:bulb:*HTTP协议还可用于传输 HTML页面以外的数据。HTTP协议的`Content-Type`标头可以被浏览器用来理解内容的[MIME](https://en.wikipedia.org/wiki/MIME) 类型*

------

## HTTP协议

HTTP协议基于TCP/IP或者[UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol)/IP协议。在将任何数据发送到接收方之前，应该开启发送方和接收方之间的通信通道。这是仅使用 TCP/IP 完成的，没有任何应用层参与，如下图（图源[维基百科](https://en.wikipedia.org/wiki/Handshaking#TCP_three-way_handshake)）所示以及TCP三次握手（图源[维基百科](https://commons.wikimedia.org/wiki/File:TCP_Three-Way_Handshake.svg)）。

![三次握手1](HTTPwithSSLTLS/handshake1.gif)

![三次握手2](HTTPwithSSLTLS/handshake2.png)

首先，客户端发送一个空的数据包（（此处“空”是指）没有任何应用数据），其中TCP协议的[`SYN`](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure)（*synchronize*）标志被置为`1`。当这个数据包被接收方收到后，它知道客户端尝试建立一个连接（会话）。

------

:bulb:*应用数据是指包含在应用层中的数据。*

------

服务端会返回一个将`SYN`和`ACK`（*acknowledge*）标记置为`1`的空数据包。当客户端收到这个数据包时，它知道服务端作出了回应并且愿意接受请求。

然后客户端又会发送一个将`ACK`标记置为`1`的空数据包。一旦服务端收到这个数据包，一个TCP通信通道就开启了。

上述过程被称作[TCP三次握手](https://en.wikipedia.org/wiki/Handshaking#TCP_three-way_handshake)。一旦握手完成，一个TCP通信通道就被开启了，客户端或者服务端可以在这个连接上发送并接收应用数据，直到其中一方关闭连接。

让我们使用 [**Wireshark**](https://www.wireshark.org/)看看真实的握手过程。我将会在浏览器中打开[**http://info.cern.ch**](http://info.cern.ch/)（世界上第一个网站），Wireshark将会为我捕获所有网络数据包。

![HTTPGETRequest](HTTPwithSSLTLS/HTTPGETRequest.png)

正如您在上面截图中看到的那样，当我们想向`info.cern.ch`服务器发送HTTP GET请求时，首先发生的是TCP三次握手。

------

:bulb:*有很多很棒的东西是您无法在一个屏幕截图中看到的。我建议您在您的系统上安装Wireshark并尝试一下。您可以根据这篇精彩的[演讲](https://youtu.be/0X2BVwNX4ks?list=PLejHxQ6h_36UrBhQmbN5wSH8xgjDMhhhA&t=2158)使用和应用Wireshark中的过滤器来跟踪TCP/IP通信通道中的数据包。*

------

一旦连接建立，HTTP协议数据将作为应用数据，通过网络数据包发送到服务器。此数据包含纯文本格式的HTTP头。

------

:bulb:*然而，数据是使用二进制编码的。*

------

![HTTP protocol](HTTPwithSSLTLS/HTTP_protocol.png)

一旦服务器接收到该数据，服务器就可以通过将数据发送回客户端来确认请求。从上面的屏幕截图可以看到，服务器使用HTTP协议应用数据进行响应，该数据包含**响应头**和纯文本的**HTML文档**。

一旦服务器发送回所有数据（可能是一个数据包或多个数据包），客户机必须通过发送空数据包来确认接收，空数据包的`ACK`标志设置为`1`，以及确认收到的数据包的**序列号**。

------

:bulb:*您可以通过这个[视频](https://www.youtube.com/watch?v=BWILgDt6jz0)更好地了解序列号和确认号（ACK）的用途。*

------

一旦服务器没有更多的数据发送，它将发送一个`FIN`（finished）标志设置为`1`的空数据包，这个标记暗含了结束信息。客户端可以确认数据包并关闭连接，如下图所示（图源[维基百科](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)）。

![wavehand](HTTPwithSSLTLS/wavehand.png)

------

:bulb:*在[HTTP持久连接](https://en.wikipedia.org/wiki/HTTP_persistent_connection)（keep-alive）中，相同的TCP连接将会被用于请求其他资源。这更高效，因为我们不必一次又一次地进行相同的TCP三次握手。*

------

很多时候，客户端和服务器之间的通信并不顺畅。可能会丢失数据包，而且数据包可能以错误的顺序到达，因此需要重新传输数据包。

------

:bulb:*UDP协议在这方面有所不同。在UDP协议中，接收方不必发送接收确认数据包，并且数据包丢失或数据包顺序也不作任何严肃处理。您可以通过[这个视频](https://www.youtube.com/watch?v=Vdc8TCESIg8)了解TCP和UDP协议之间的差异。*

------

### 安全问题

HTTP是一个不安全的协议，因为HTTP协议以纯文本格式编码。因此任何中间人都可以侦听TCP通信并读取您通过网络传输的个人数据。

这就是为什么像谷歌这样的搜索引擎在搜索结果中给不安全的网站分配较低的索引。然而，使一个网站安全并不是一件容易的事。但你还是可以使用[Cloudflare](https://www.cloudflare.com/)这样的网站来使你的网站安全，而不必担心实施的问题。

## SSL/TLS概述

HTTPS是安全的超文本传输协议（**HyperText Transfer Protocol Secure**）的意思，但是这在某些方面具有误导性。HTTPS协议并不能单独对数据进行加密，事实上，它依赖于**SSL**或**TLS**协议层。

------

:bulb:*我打算用TLS协议来指代SSL和TLS协议，很快我们就会明白为什么SSL的名字在21世纪可能会产生误导。*

------

HTTP协议层和TLS协议层都是应用层的一部分。TLS层的作用是使用TLS握手（在TCP握手之后）与服务器建立安全连接，并使用与服务器协商的一些加密算法对HTTP数据进行加密。

最后的加密数据就成为了服务器将要接收的网络数据包的应用数据。任何中间人都可能获得该数据，但是他们无法理解数据，因为应用数据是加密的。这就防止了[中间人攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

当HTTP协议与TLS协议结合使用时，它被称为HTTPS协议。为了调用浏览器或程序使用TLS进行通信，我们通常在URL中使用`https:// `协议前缀。

要了解TLS的工作原理，我们首先需要了解加密的工作原理和不同种类的加密算法。

[加密](https://en.wikipedia.org/wiki/Encryption)是将一种格式的数据编码成另一种我们人类无法读取的格式的过程。为了加密一些数据，我们会使用不同的数学技巧和秘密参数。

其中一些参数是不能有意公开的。使用这些相同或相似的参数，我们可以破译被加密的数据。这种参数被统称为密钥（**key**）。如果密钥公开，任何有权访问的人都可以解密和读取我们的隐私数据。

### 非对称加密算法

在非对称加密算法中，我们有两个用于加密和解密的密钥。**公钥**用于加密数据并将其提供给公众，只有**私钥**（保密不公开）才能破译它。

------

:bulb:*这种非对称加密被称为[公钥加密法](https://en.wikipedia.org/wiki/Public-key_cryptography)。*

------

最流行的非对称加密算法是[RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem))（Rivest-Shamir-Adleman），因为它广泛用于网络上的密钥交换和数字签名验证。但是，现在浏览器正在采用一种更安全、更有效的[Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)密钥交换算法进行密钥交换。

------

:bulb:*RSA私钥也可用于加密数据，公钥可以解密由私钥加密的数据。这用于生成和验证加密数据的数字签名和SSL证书（稍后解释）。*

------

非对称加密算法通常比较慢，而且是CPU密集型（[为什么](https://crypto.stackexchange.com/questions/5782/why-is-asymmetric-cryptography-bad-for-huge-data)），密钥或数据的长度越长，加密或解密数据的时间就越长。

因此，公钥加密法不用于批量数据加密。相反，我们使用对称加密算法来加密或解密大批量的数据，这更快，更高效。而非对称加密仅仅是传输共享对称加密密钥的方法。

### 对称加密算法

[对称加密算法](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)使用**相同的密钥**加密和解密数据，这种密钥被称作共享密钥。这也是一种安全的算法，但是您不能将密钥公开。

对称密钥加密通常比公钥加密更快（[为什么？](https://www.quora.com/What-are-the-reasons-why-symmetric-encryption-works-fast)），可用于批量数据加密。因此它也被称为**块密码**（ [**block cipher**](https://en.wikipedia.org/wiki/Block_cipher)）。

对称加密算法主要用于在两个受信任方之间开启一个加密通道。只有通信双方才知道他们之间共享的数据，因为网络上没有其他人能够获得共享的对称密钥。

网络上最流行的对称加密算法之一是[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard.)（*Advanced Encryption Standard*）。

### SSL（安全套接字层*secure sockets layer*）协议

SSL协议最早由网景（[**Netscape**](https://en.wikipedia.org/wiki/Netscape)）浏览器团队设计，SSL2.0于1995年公开发布。它很快被带有安全改进的SSL 3.0所取代，但在2015年被IETF[废弃](https://tools.ietf.org/html/rfc7568)。

目前，SSL协议已被没落（某些变体除外），没有人使用它。 IETF于1999年推出了[TLS协议的第一个版本](https://tools.ietf.org/html/rfc2246)，该协议现在是网络上所有加密通信的标准。

------

:bulb:*当有人谈及SSL时，其实他们可能是在说TLS。甚至SSL证书事实上也是TLS证书。**SSL v3.1**或**SSL v4**只是**TLS1.0+**版本的别名。*

------

### TLS（传输层安全*transport layer security*）协议

TLS是对 SSL协议的改进。TLS1.0于1999年推出，经历了一些迭代。当前最受支持的TLS版本是[TLS1.2](https://tools.ietf.org/html/rfc5246)。

[TLS1.3](https://tools.ietf.org/html/rfc8446)于2018年推出，是对TLS1.2的一次重大革新，提高了效率和数据安全性，减少了建立安全TCP连接的握手请求次数。

TLS1.3仅支持Diffie-Hellman公钥加密算法的修改版本，用于在客户端和服务器之间共享对称密钥。它还放弃了对用于密钥交换的RSA算法的支持。

------

:bulb:*然而，不是所有的浏览器和服务器都支持TLS 1.3。因此，浏览器和服务器可能不兼容TLS1.3的通信，从而使用较低的TLS版本，如TLS1.2。*

------

## 使用TLS1.2的HTTPS握手

当我们发送带有`https://`协议前缀的HTTP请求时，首先使用我们之前看到的三次握手建立TCP连接。如下图（图源[维基百科](https://commons.wikimedia.org/wiki/File:Full_TLS_1.2_Handshake.svg)）中的蓝线所示。

![TLS1.2handshake](HTTPwithSSLTLS/TLS1.2handshake.png)

------

:bulb:*这次TCP层的目标端口是**443**，这是HTTPS协议的[默认端口](https://tools.ietf.org/html/rfc2818)。*

------

TCP连接建立后，TLS握手开始。首先，客户端发送一个空数据包，但具有TLS1.2 协议层。该层包含一些元数据和一个**客户端 Hello**消息。

![TLSClientHelloMessage1](HTTPwithSSLTLS/TLSClientHelloMessage1.png)

从上面的屏幕截图中可以看到，在**客户端 Hello**握手消息中，我们附上了我们的应用程序（如浏览器）目前支持的一些密码套件（**cipher suites**）列表，我们稍后再谈。

------

:bulb:*当发送**客户端 Hello**握手信息时，通信的应用程序开始可能首先尝试使用TLS协议的最高版本，如TLS 1.3，然后降级到服务器支持的合适版本。如果应用程序不支持服务器所要求的版本，它可能会放弃TCP连接并发出警告。*

------

![TLSClientHelloMessage2](HTTPwithSSLTLS/TLSClientHelloMessage2.png)

伴随密码套件列表，我们会发送一个 [**server name indication**](https://en.wikipedia.org/wiki/Server_Name_Indication)（SNI）消息，表明我们试图与之连接的主机名（域名）。服务器使用它发回与此主机名匹配的恰当的SSL证书。

![TLSServerHelloMessage](HTTPwithSSLTLS/TLSServerHelloMessage.png)

当接收到**客户端 Hello**包后，服务端使用包含**服务端 Hello**消息的空包进行响应。该数据包含有服务端从客户端提供的选项列表中选择的密码套件。然后，它会发送一个数据包，其中包含我们使用 SNI 值要求的域名的 SSL 证书。此时，服务端没有其他可发的，因此它发送了带有"**服务端 Hello 完成**"消息的数据包。

客户端收到SSL证书后，我们的应用程序会对其进行验证。该证书还包含一个由服务端选择的密码套件的公钥。我们将在SSL课程中讨论更多关于证书验证的过程。

密码套件包含一个加密算法列表和一个哈希函数，用于加密和验证在客户端和服务器之间传输的数据。一个有效的密码套件的简单例子如下：

`TLS_RSA_WITH_AES_256_CBC_SHA`

如果服务端选择上述密码套件就意味着服务端将使用RSA算法对加密批量数据的共享秘钥进行加密。客户端和服务端使用的批量加密算法是AES256 bit（[***CBC***](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation) 模式）。密码套件的最后一项是用于创建数字签名的单向散列函数。关于数字签名，我们稍后将讨论更多。

一旦客户端和服务器就密码套件达成一致，加密通信就可以开始了。到目前为止，我们所有的数据包都没有被加密，因为它们不包含任何应用数据，而且事先没有加密知识。客户端现在生成了一个用于批量数据加密的共享密钥。正如密码套件描述的那样，客户端将创建一个256位的AES对称-密钥算法的密钥，并与服务端共享。

![ClientKeyExchange](HTTPwithSSLTLS/ClientKeyExchange.png)

客户端使用从 SSL 证书中获取的服务器的 RSA 公钥，对共享密钥进行加密，并与消息**Client Key Exchange**一起发送给服务器。该数据包还包含**Change Cipher Spec**消息，表示客户端正在使用商定的密码规范（**agreed cipher spec**）进行数据加密。

在同一个数据包中，客户端发送一个**Finished**握手消息，表明来自客户端的TLS握手已经完成。这个消息会被共享对称密钥（AES）加密。一旦服务端接收到这个数据包，它会使用私钥对共享对称密钥解密。由于没有其他人可以获得私钥，因此在网络上无法读取此共享密钥。

通过查看Change Cipher Spec消息，服务器也会将其数据读/写模式更改为商定的密码规范。然后它将使用刚刚得到的共享密钥解密握手消息。由于握手消息是**Finished**，因此它会客户端返回一个包含**Change Cipher Spec**的数据包，这表明服务端正在使用商定的密码规范进行数据加密，并使用共享密钥加密**Finished**消息。

![ServerFinished](HTTPwithSSLTLS/ServerFinished.png)

一旦客户端收到这个数据包，TLS1.2握手就完成了，应用程序数据可以使用选定的批量数据加密算法（对称加密算法）进行加密/解密，并通过TCP通道传输。

------

:bulb:中间人可以篡改握手，但是由于共享密钥已于RSA加密交换，因此握手完全安全（*没理解* :confused:）。然而，如果被加密的Finished消息已经被篡改，TCP连接将被废弃并且无法通过同一管道进行进一步的通信。

------

### 会话密钥（Session Keys）

有时，（客户端）原始的共享密钥不会与服务器共享，无论RSA加密有多强（预先对其进行加密）。相反，（客户端）与服务器共享**预主密钥**（pre-master secret）而不是共享密钥。客户端和服务器根据TLS握手期间协商的算法，使用预主密钥得到相同的共享密钥。此共享密钥将在重用之前的TLS会话（恢复）时生成，而无需经过完整的握手。但是，如果服务器的私钥被泄露，则将无法防止中间人攻击。

## 使用TLS1.3的HTTPS握手

TLS1.3带来了许多改进。它放弃了对用于密钥交换的RSA加密的支持。如果服务器的私钥被泄露，任何有权访问该私钥的人都将能够解密客户端和服务器之间传输的所有消息。

### 安全

TLS1.3协议被强制使用**ECDHE**密钥交换算法。它基本算是一种采用椭圆曲线加密技术（[**Elliptic-curve**](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography) cryptography）的[**Diffie-Hellman**](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) (DH) 公钥算法。这两者一起形成了非常安全的ECDHE密码。

ECDHE的工作方式是在客户端和服务器端保留一个随机生成的私钥。客户端使用 DH算法私有参数、DH算法公共参数和椭圆曲线公共参数生成一个公钥。该公钥将与所有公共参数一起共享给服务器。服务器使用客户端的公钥和参数以及自己的DH算法私有参数，算出自己的公钥并与客户端共享。服务器根据这些结果生成共享密钥。使用服务器的公钥和之前的参数，客户端算出一个共享密钥。DH算法背后花哨的数学使得客户端和服务器可以生成相同的共享密钥。此共享密钥用于批量数据加密。

------

:bulb:我不打算解释ECDHE背后的数学原理，但您可以观看这个[精彩的视频](https://www.youtube.com/watch?v=M-0qt6tdHzk&t=2s)来了解DH算法的工作原理。

------

### 前向保密（Forward Secrecy）

与TLS1.2 类似，客户端和服务器端生成预主密钥（计算用于批量数据加密的最终会话密钥），而不是原始的共享密钥。

ECDHE 中的`E`代表临时参数。这意味着客户端和服务器使用 DH 算法生成的 DH 参数是暂时（**ephemeral**）的。除非重用 TLS 会话，否则每个新的TLS会话都会选择新参数并生成新的会话密钥。

由于此过程不涉及SSL私钥和公钥，所以它们对提取通过 TLS 1.3 通道传输的数据不是很有用。这使得TLS1.3通信具有完美的前向保密（[*PFS*](https://en.wikipedia.org/wiki/Forward_secrecy)）。

------

:bulb:如果私有参数、预主密钥或会话密钥泄露，那么攻击者也只能提取单个TLS会话的数据。

------

### 更短的握手时间（Shorter Handshake）

TLS 1.3不仅有完美的前向保密，而且可以使TLS握手非常短。

与TLS1.2一样，客户端发送**客户端Hello**消息，同时此数据包中还包含最终用于生成共享密钥的公共参数，如下图所示。

![TLS1.3ClientHelloMessage](HTTPwithSSLTLS/TLS1.3ClientHelloMessage.png)

在同一数据包中还包含客户端支持的密码套件列表，服务器将选择使用合适的ECDHE算法。

```shell
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
```

如果服务器选择上述算法，则使用RSA的ECDHE将被用于密钥交换。RSA加密仅用于数字签名（基本上使用SSL私钥加密）服务器发送的公钥（密钥交换过程），以便客户端验证它是否真的是由服务器发送的。

![TLS1.3ServerHelloMessage](HTTPwithSSLTLS/TLS1.3ServerHelloMessage.png)



## 未完，待续。。。

:sleepy:

## 其他可以参考的链接：

[HTTPS篇之SSL握手过程详解 | Razeen`s Blog](https://razeencheng.com/post/ssl-handshake-detail.html)

