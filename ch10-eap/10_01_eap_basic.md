# EAP基础知识

EAP用于在允许用户访问网络之前对用户进行身份验证。 由于EAP是一个具有可扩展性的框架，因此它使用许多可用方法之一来对用户进行身份验证。 本节介绍了EAP如何工作的基本概念。 有关EAP的详细信息，请参见RFC 3748。我们将首先看看EAP的三个核心组件，并继续看看LAN上的典型EAP会话是什么样的。

## EAP组件
下图显示了EAP框架的各个组件：

![the_various_components_of_the_EAP_framework](https://github.com/lsqms/FreeRADIUS/blob/master/image/ch10/the_various_components_of_the_EAP_framework.PNG?raw=true)

EAP框架涉及三个主要组件。

### 认证
验证者是守门人。它控制谁有权访问网络以及谁被阻止。以下是验证者的一些示例：
+ 在LAN上支持802.1x的管理型交换机。
+ 包含WPA2-Enterprise Wi-Fi安全性的接入点。
+ 支持PPP EAP的远程访问服务器。目前可用的开源远程访问服务器包括OpenVPN，Poptop（PPTP），strongSwan和Openswan。

验证者是是将转发和翻译请求方和后端身份验证服务器之间的对话的辅助工具。这具有以下优点：
+ 使用中央服务器用于身份验证。
+ EAP成为一种可扩展协议，允许将新的身份验证方法引入后端或请求者，而无需更改身份验证器上的功能。

验证者不会决定允许或拒绝访问网络的人，而只是遵循来自后端验证服务器的指令。验证者使用TCP / IP与RADIUS服务器以及使用EAP通信与请求者通信。


### 请求者
请求者是客户端机器用于身份验证的软件。验证成功后，验证者授予客户端对网络的访问权限。 EAP仅仅是支持许多身份验证方法的框架。请求者同样支持各种身份验证方法，并将使用一种。遗憾的是，某些操作系统上的默认请求者可能不包括对部分所需EAP方法的支持。这可以通过安装第三方请求者来解决，该请求者包括对所需EAP方法的支持。

使用请求方的网络类型将确定请求方如何封装EAP数据包以便与验证方通信。在LAN上，它将封装在EAPOL（EAP Over LAN）协议中。在Wi-Fi网络上，它将封装在EAPOW（EAP Over Wireless）协议中。 EAP适用于数据链路层（第2层），不依赖于或使用任何TCP / IP协议。 TCP / IP是当今最流行的协议。但是，在客户端计算机上使用TCP / IP并不是成功进行身份验证的必要条件。
### 后端验证服务器
虽然身份验证器控制对网络的访问，但是后端身份验证服务器决定谁将被授予或拒绝访问网络。此服务器通常是RADIUS服务器，但EAP标准不会将其仅限于RADIUS。本章将向您展示如何将FreeRADIUS用于此类后端身份验证服务器。
现在我们更熟悉EAP协议的组件，现在是时候查看来自LAN上客户端的典型身份验证请求了。

### EAP对话
下图显示了在激活802.1x的LAN上对用户进行身份验证之前，三个EAP组件之间的对话。 此处描述的EAP方法是EAP-MD5。 由于此方法不是很安全，因此不建议在生产环境中使用。

使用EAP协议进行身份验证与典型的身份验证过程不同，在此过程中，客户端通过识别自身来迈出第一步。 使用EAP，身份验证器要求客户端将自己标识为第一步。

### EAPOL启动

由于验证者通常知道潜在用户何时连接到交换机，因此通常不使用EAPOL-Start消息。 但是，请求者可以通过发送EAPOL-Start数据包让验证者了解其存在。

![the_conversation_between_the_three_EAP_components](https://github.com/lsqms/FreeRADIUS/blob/master/image/ch10/the_conversation_between_the_three_EAP_components.PNG?raw=true)

一旦验证者知道新客户端已连接，它将首先要求客户端通过向客户端发送EAPOL-Packet来标识自己。

### EAPOL-Packet
EAPOL-Packet包含一个Code字段，它可以是四个代码之一，即：

|Name|Value|
|:-|:-:|
|EAP-Request|1|
|EAP-Response|2|
|EAP-Success|3|
|EAP-Failure|4|

应使用EAP-Response回答EAP-Request。 如果将其与RADIUS进行比较，则它类似于我们具有Access-request（1）或Access-Challenge（11）的Code字段，依此类推。

EAP-Request和EAP-Response数据包还包含Type字段。 下表列出了一些类型：
|Name|Value|
|:-|:-:|
|Identity|1|
|Notification|2|
|Nak|3|
|MD5-Challenge|4|

前三种类型有特殊用途。其余类型表示使用的身份验证方法。让我们看看EAPOL-Packet流：
1. 您可以在上图中看到后端认证服务器不参与向请求者发送EAPOL-Packet-> EAP-Request-> Identty。这是由Authenticator完成的。
2. 当Authenticator从Supplicant收到响应时，它会将此响应转发给Backend authenticaton服务器。 Authenticator通过将EAPOL-Packet-> EAP-Response-> Identty转换为包含EAP-Message AVP的RADIUS Access-Request数据包来完成此操作。
3. RADIUS服务器通过发送包含EAP-Message AVP的Access-Challenge数据包来响应Access-Request。该EAP-Message AVP包含用于向请求者发送MD5质询的数据。
4. Authenticator使用EAP-Message AVP的内容创建EAPOL-Packet-> EAP-Request-> MD5-Challenge并将其发送给请求者。
5. 请求方使用此质询(挑战)在EAPOL-Packet-> EAP-Response-> MD5-> Challenge数据包内发送包含用户加密密码的响应。
6. 该数据包再次转换为包含EAP-Message AVP的RADIUS访问请求数据包，并发送到后端身份验证服务器。
7. 如果正确提供了此密码，RADIUS服务器将使用包含EAP-Message AVP的Access-Accept数据包进行验证。
8. 验证器现在将打开交换机上的端口，以便其他流量可以开始通过。
9. Authenticator还将向Supplicant发送EAPOL-Packet-> EAP-Success数据包，以通知它成功。

需要记住的一点是，在初始数据包之后，客户端对后端身份验证服务器的响应仅由身份验证器转发给它。
Authenticator只是将数据包从EAP重新格式化为RADIUS，并将其路由到后端身份验证服务器。
这使我们结束了关于EAP的速成课程。下一节将把这种客观知识转化为主观体验。

















