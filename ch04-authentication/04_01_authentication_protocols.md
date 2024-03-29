# 验证协议
本部分将为您提供三种常见身份验证协议的背景知识。 这些协议涉及提供用户名和密码。

测试身份验证时，radtest程序默认使用密码身份验证协议（PAP）。 PAP不是唯一的认证协议，但可能是最通用和广泛使用的协议。 您应该了解的身份验证协议是PAP，CHAP和MS-CHAP。 这些协议中的每一个都涉及用户名和密码。 可扩展的验证协议（EAP）协议在本书后面有自己的专用章节，并将向我们介绍更多的身份验证协议。

身份验证协议通常用于连接客户端和NAS的数据链路层。 只有在认证成功后才能建立网络层。 NAS充当代理，将来自用户的请求转发到RADIUS服务器。

>数据链路层和网络层是开放系统互连模型（OSI模型）内的层。关于这种模式的讨论几乎可以在任何关于网络的书籍中找到：
http://en.wikipedia.org/wiki/OSI_model

## PAP
PAP是用于在进行点对点连接时提供用户名和密码的第一个协议之一。 使用PAP，NAS获取PAP ID和密码，并将其作为用户名和用户密码发送到Access-Request数据包中。 与CHAP和MS-CHAP相比，PAP更简单，因为NAS只需向RADIUS服务器提供用户名和密码，然后对其进行检查。 此用户名和密码直接来自用户通过NAS到服务器的单个操作。

虽然PAP以明文形式传输密码，但使用它不应该总是不受欢迎。 此密码仅在用户和NAS之间以明文形式显示。 当NAS将请求转发给RADIUS服务器时，将加密用户的密码。

如果在安全隧道内使用PAP，则它与隧道一样安全。 这类似于您的信用卡详细信息在HTTPS连接中进行隧道传输并传送到安全Web服务器的情况

>HTTPS代表安全超文本传输协议，是一种使用安全套接字层/传输层安全（SSL/TLS）在不安全网络上创建安全通道的Web标准。 一旦建立了这个安全通道，我们就可以通过它传输敏感数据，如信用卡详细信息。 HTTPS每天用于通过Internet保护数百万个交易。

请参阅以下典型强制网络门户配置的示意图。
![schematic_of_a_typical_captive_portal_configuration](https://github.com/lsqms/FreeRADIUS/blob/master/image/ch04/schematic_of_a_typical_captive_portal_configuration.PNG?raw=true)

下表显示了PAP请求中涉及的RADIUS AVPs：

|AVP			|典型值									|
|:-:			|:-:									|
|User-Name		|alice									|
|User-Password	|\xbe\xd1}r\xc8vc/\x93*\x8f\xa0$\xa4gz	|

如您所见，在NAS和RADIUS服务器之间加密了User-Password的值。 如果用户的密码可以被第三方捕获，则将用户的密码从用户传输到NAS可能存在安全风险。

## CHAP

CHAP代表挑战握手身份验证协议，旨在改进PAP。它会阻止您传输明文密码。
CHAP是在拨号调制解调器受欢迎的时代创建的，并且对PAP的明文密码的关注度很高。

在建立到NAS的链路之后，NAS生成随机质询并将其发送给用户。然后，用户通过返回在身份证（与挑战一起发送），挑战和用户密码上计算的单向哈希来响应此挑战。然后，NAS使用用户的响应来创建Access-Request数据包，该数据包被发送到RADIUS服务器。根据RADIUS服务器的回复，NAS将向用户返回CHAP成功或CHAP失败。

NAS还可以以随机间隔请求通过向用户发送新挑战来重复认证过程。这是为什么它被认为比PAP更安全的另一个原因。

CHAP的一个主要缺点是虽然密码是加密传输的，但密码源必须是明文，以便FreeRADIUS执行密码验证。

FreeRADIUS常见问题解答讨论了传输明文密码与在服务器上以明文形式存储所有密码相比的危险。

下表显示了CHAP请求中涉及的RADIUS AVPs：

|AVP			|典型值									|
|:-:			|:-:									|
|User-Name		|alice									|
|User-Password	|4A2578ED8C1A747AFED86EB96F024ADFF8		|

## MS_CHAP

MS-CHAP是由Microsoft创建的挑战 - 握手认证协议。 有两个版本，MS-CHAP版本1和MS-CHAP版本2。

NAS发送的质询与标准CHAP质询数据包的格式相同。 这包括身份认同和任意挑战。 用户的响应在格式上也与标准CHAP响应数据包相同。 唯一不同的是Value字段的格式。 Value字段被子格式化以包含MS-CHAP特定字段。其中一个字段（NT-Response）包含非常特定加密格式的用户名和密码。NAS将使用用户的回复来创建Access-Request数据包，该数据包将发送到RADIUS服务器。 根据RADIUS服务器的回复，NAS将向用户返回成功数据包或失败数据包。

>RADIUS服务器不参与发送挑战。 如果您查看NAS和RADIUS服务器之间的RADIUS流量，则可以确认只有Access-Request后跟Access-Accept或Access-Reject。 向用户发送挑战并从她或他接收响应是发生在NAS和用户之间的。

MS-CHAP还有一些不属于CHAP的增强功能，例如用户更改其密码或包含更多描述性错误消息的功能。

该协议与LAN Manager和NT Password哈希紧密集成。 FreeRADIUS会将用户的明文密码转换为LM密码和NT密码，以确定MS-CHAP请求中出现的密码哈希值是否正确。 虽然MS-CHAP存在已知的弱点，但它仍然被广泛使用并且非常受欢迎。

>永远不要把话说绝了。如果您当前对RADIUS部署的要求不包括使用MS-CHAP，那么可以考虑有一天您可以使用它。最流行的EAP协议使用MS-CHAP。EAP在Wi-Fi身份验证中至关重要。

由于MS-CHAP是特定于供应商的，因此VSAs而不是AVPs是NAS和RADIUS服务器之间的访问请求的一部分。 它与用户名AVP一起使用。

|AVP				|典型值									|
|:-:				|:-:									|
|MS-CHAP-Challenge	|CF702D195889B225						|
|MS-CHAP-Response	|:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:e6:3e:d6:be:b8:90:b4:88:84:bc:b3:71:c0:ce:b8:d3:1d:1a:06:35:32:c5:f1:85		|

现在我们了解了有关身份验证协议的更多信息，让我们看看FreeRADIUS如何处理它们。