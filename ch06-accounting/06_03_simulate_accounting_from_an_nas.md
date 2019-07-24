# 行动时刻 - 模拟NAS的计费
在第3章，FreeRADIUS使用入门，我们介绍了radclient命令。

本部分创建三个可与radclient一起使用的文件，以模拟NAS通常发送到RADIUS服务器的计费数据包。

# 用于模拟的文件
三个文件中的AVP类似于从hostapd程序发送的AVP。
>hostapd是一个守护进程，用于控制Wi-Fi网络上的身份验证。它可以配置为与身份验证一起进行计费，通常在基于Atheros的Wi-Fi接入点上与OpenWRT一起安装。您还可以使用它将Wi-Fi NIC转换为Linux上的接入点。

当用户的会话启动时，NAS将通知RADIUS服务器。在会话期间，NAS可以将会话的更新发送到RADIUS服务器，然后当会话结束时，NAS也将通知RADIUS服务器。

这些计费数据包内部是Acct-Status-Type AVP，它将反映Start，Interim-Update和Stop的会话状态。这对应于我们将创建的三个文件。以下文件名为4088_06_acct_start.txt，将创建一个会话：
```
Packet-Type=4
Packet-Dst-Port=1813
Acct-Session-Id = "4D2BB8AC-00000098"
Acct-Status-Type = Start
Acct-Authentic = RADIUS
User-Name = "alice"
NAS-Port = 0
Called-Station-Id = "00-02-6F-AA-AA-AA:My Wireless"
Calling-Station-Id = "00-1C-B3-AA-AA-AA"
NAS-Port-Type = Wireless-802.11
Connect-Info = "CONNECT 48Mbps 802.11b"
```
以下文件名为4088_06_acct_interim-update.txt，将更新会话：
```
Packet-Type=4
Packet-Dst-Port=1813
Acct-Session-Id = "4D2BB8AC-00000098"
Acct-Status-Type = Interim-Update
Acct-Authentic = RADIUS
User-Name = "alice"
NAS-Port = 0
Called-Station-Id = "00-02-6F-AA-AA-AA:My Wireless"
Calling-Station-Id = "00-1C-B3-AA-AA-AA"
NAS-Port-Type = Wireless-802.11
Connect-Info = "CONNECT 48Mbps 802.11b"
Acct-Session-Time = 11
Acct-Input-Packets = 15
Acct-Output-Packets = 3
Acct-Input-Octets = 1407
Acct-Output-Octets = 467
```
最后，以下文件名为4088_06_acct_stop.txt，将结束会话：
```
Packet-Type=4
Packet-Dst-Port=1813
Acct-Session-Id = "4D2BB8AC-00000098"
Acct-Status-Type = Stop
Acct-Authentic = RADIUS
User-Name = "alice"
NAS-Port = 0
Called-Station-Id = "00-02-6F-AA-AA-AA:My Wireless"
Calling-Station-Id = "00-1C-B3-AA-AA-AA"
NAS-Port-Type = Wireless-802.11
Connect-Info = "CONNECT 48Mbps 802.11b"
Acct-Session-Time = 30
Acct-Input-Packets = 25
Acct-Output-Packets = 7
Acct-Input-Octets = 3407
Acct-Output-Octets = 867
Acct-Terminate-Cause = User-Request
```
通过将这三个文件与radclient程序一起使用，我们现在可以探索FreeRADIUS会计的各个方面。
# 开始一个会话
当NAS从RADIUS服务器收到访问接受数据包时，NAS会尝试将标识符字段与待处理的访问请求匹配。 如果找到匹配，并且NAS配置为计费，则NAS将向RADIUS服务器发送Code 4 RADIUS数据包（Accounting-Request）。 这标志着会议的开始。 让我们使用radclient从NAS模仿这个动作：

1. 在调试模式下启动FreeRADIUS。
2. 使用radclient命令和4088_06_acct_start。
txt文件向FreeRADIUS发送Accounting-Request：
`> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt`
3.观察FreeRADIUS和radclient命令的输出。
以下是来自radclient的反馈，通过返回Code 5（Accounting-Response）来请求成功：
`Received response ID 66, code 5, length = 20`
4.通过发出radwho命令确认alice的活动会话。
您必须是root用户才能发出此命令。
根据FreeRADIUS的编译方式和您使用的发行版，radwho命令可能会返回错误。如果是这种情况，请按照下一节修复它：
5.尽管是root用户，但您会收到以下错误：
`radwho: Error reading /var/log/radius/sradutmp: No such file or directory`
6. sradutmp文件不存在，因为在启用站点的/默认虚拟服务器的记帐部分内禁用了sradutmp模块。通过取消注释以下行来激活sradutmp：
`sradutmp`
7.再次以调试模式重新启动FreeRADIUS。像以前一样发出radclient和radwho命令。您现在应该看到如下内容：
```
radwho
Login Name What TTY When From Location
alice alice shell S0 Sun 16:34 127.0.0.1
```
alice的活跃会话现在通过radwho反映出来。接下来，我们将使用停止请求结束此活动会话。
# 结束会话
当用户注销或会话超时时，NAS将向RADIUS服务器发送停止请求，以便会计详细信息可以反映NAS上发生的事件。请按照以下步骤结束请求：

1. 确保FreeRADIUS服务器在调试模式下运行。
2. 使用radclient命令和4088_06_acct_stop.txt文件向FreeRADIUS发送请求：
`$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_stop.txt`
3. 通过检查radwho的输出确认会话已关闭：
```radwho
Login Name What TTY When From Location
```
# 孤立会话
有时可能会发生NAS挂起。当NAS稍后重置时，FreeRADIUS上的计费信息仍会反映旧状态。您可以使用radzap命令关闭FreeRADIUS上的任何打开的计费记录。让我们立即看看radzap：

1. 使用radclient命令和4088_06_acct_start.txt文件在FreeRADIUS上启动会话：
`$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt`
2. 使用radwho命令确认打开的会话：
```
radwho
Login Name What TTY When From Location
alice alice shell S0 Sun 16:58 127.0.0.1
```
3. 现在，您可以使用以下命令从127.0.0.1中删除所有活动会话：
`radzap -N 127.0.0.1 127.0.0.1 testing123`
Radwho将显示现在没有活动会话。

如果您在发布radzap时查看FreeRADIUS服务器的反馈，您将看到它发送以下请求：
```
rad_recv: Accounting-Request packet from host 127.0.0.1 port 43629, id=195, length=38
Acct-Status-Type = Accounting-Off
NAS-IP-Address = 127.0.0.1
Acct-Delay-Time = 0
```
这使我们结束了基础计费的实践练习。接下来将详细讨论。

## 刚刚发生了什么？
我们使用radclient命令将会计请求发送到FreeRADIUS服务器，其方式类似于NAS的工作方式。我们还模拟了NAS挂起的情况，使FreeRADIUS上的记帐数据与NAS的状态不同步。在谈到我们刚刚完成的练习时，让我们看看一些兴趣点。

# 计费独立性
FreeRADIUS中的会计独立于授权和身份验证。它使用单独的端口，由客户端发送到服务器的Accounting-Request数据包组成。服务器使用Accounting-Response数据包进行响应以确认请求。

会计数据用于衡量网络上的使用情况。 NAS可以报告用户连接到网络的时间以及用户的数据使用情况。

会计记录不会反映用户在会话期间访问过的网站等详细信息。它们仅指示时间和数据使用情况。

# NAS：重要的AVP
虽然NAS发送的AVP有所不同，但是一些AVP很重要，应该存在于Accounting-Request中。
## Acct-Status-Type
此属性指示此Accounting-Request是标记用户服务（开始）还是结束（停止）的开始。每个选项都由一个特定的数字表示。

会话开始时，它将被指定为1（开始）。当会话的数据更新时，它将被指定为3（临时更新），当会话结束时，它将被指定为2（停止）。 Acct-Status-Type还可以包含Accounting-On（由7表示）和Accounting-Off（由8表示）等值，以关闭NAS的所有打开会话。 Acct-Status-Type的值决定了FreeRADIUS操纵用户会计数据的方式。

## Acct-Session-Id
Acct-Session-Id用于唯一标识用户的会话。这与组合使用Acct-Status-Type来记录用户会话的状态。当AcctStatus-Type的值更改为反映会话的状态（Start，Interim-Update或Stop）时，Acct-Session-Id的值在整个会话期间保持不变。

# AVP表示使用情况
下表显示了Accounting-Request数据包中的AVP，它反映了用户的使用情况：

|AVP|描述|
|:-|:-:|
|Acct-Session-Time|会话持续时间|
|Acct-Input-Octets|字节从用户发送到NAS|
|Acct-Output-Octets|字节从NAS发送给用户|

这三个AVP指示用户的时间和数据使用。它们仅存在于具有Acct-Status-Type = Interim-Update或Acct-Status-Type = Stop的数据包中。 具有Acct-Status-Type = Start的数据包不能包含它们。

# NAS：包含的其他AVP
包含在Accounting-Request中的AVP取决于Acct-Status-Type的值。启动会话时，无需发送指示时间和数据使用情况的AVP。这些AVP仅包含在后续请求中。

Stop请求通常会指示终止原因：
Acct-Terminate-Cause = User-Request

包含的AVP也取决于NAS。有时NAS不包括所需的AVP（Hostapd），有时它会交换输入和输出（Chillispot）。

您永远无法确定NAS将为服务器带来什么。因此，最好首先测试并查看包含哪些AVP，以及客户端是否需要额外的配置才能使会计按预期工作。

>有时会出现一个名为Acct-Delay-Time的AVP。 RADIUS服务器可以使用此AVP的值来调整记录会话详细信息时的开始和停止时间。当NAS很难将会计请求发送到RADIUS服务器并且必须重新发送请求时，通常会出现这种情况。如果Acct-Delay-Time的值很大，你应该调查为什么会这样。

# FreeRADIUS：预核算部分
当FreeRADIUS收到Accounting-Request时，它首先被传递到preacct部分。此部分在虚拟服务器的文件中定义。像大多数FreeRADIUS配置一样，默认配置可以工作正常，但如果你想为用户操纵AVP值，就可以在这里进行处理。 preacct部分内的注释表明了本节可以做什么。

一个有趣的模块是预处理模块（rlm_preprocess）。该模块在需要时恢复请求的AVP的完整性。在我们的示例中，它添加了NASIP-Address AVP，因为它丢失了。根据RFC 2866，此AVP必须位于Accounting-Request中。

RADIUS计费请求中必须存在NAS-IP-Address或NAS-Identifier。

acct_unique条目通过确定Acct-Unique-Session-ID的值来确保每个请求都具有半唯一标识符。

# 领域
当您将计费请求转发到另一台RADIUS服务器时，preacct部分也非常重要。后缀模块（领域模块的实例）用于识别和触发这种流量的路由。还列出了IPASS和ntdomain，但已注释掉，它们都是领域模块的实例。后缀，IPASS和ntdomain都在用户名AVP中寻找一个独特的模式来确定请求的领域。第12章“漫游和代理”将深入介绍如何将流量转发到其他RADIUS服务器。

# 设置Acct-Type
您也可以使用preacct部分中的files模块。该模块将用于设置Acct-Type内部AVP。 Acct-Type AVP用于通过强制它由模块的不同实例处理来分离会计部分内的会计流量。这与在授权部分中可以设置Auth-Type内部AVP以便指定要在身份验证部分中使用哪种身份验证方法的原理相同。您可以在以下URL上阅读有关此功能的使用的更多信息：
http://freeradius.org/radiusd/doc/Acct-Type

# FreeRADIUS：计费部分
在preacct部分处理了Accounting-Request后，它将被传递到计费部分。这也在虚拟服务器的文件中定义。这也是是我们已激活的sradutmp模块的已部分。计费部分执行计费数据的实际记录。有多种方法可以做到这一点。默认情况下，它将使用详细信息模块记录为文本文件。但是，我们也可以指定它应该使用SQL模块记录到SQL数据库中。此部分还可用于记录用户的使用情况（每日模块）。然后可以使用该用法来确定授权结果。

建议您阅读会计部分中的注释，以了解所包含的模块的功能。
# 最大限度地减少孤立会话
当NAS挂起时，它无法向RADIUS服务器发送任何请求。重新启动NAS后，应发送Acct-Status-Type = Accounting-On。在正常关闭时，它应发送Acct-Status-Type = Accounting-Off。这使计费记录更加健壮和可靠。 radzap命令模拟NAS的正常关闭，用于关闭孤立会话。
#radwho
根据FreeRADIUS的编译方式，radwho命令可能会在FreeRADIUS日志目录中存在sradutmp文件。我们必须在计费部分启用sradutmp才能存在此文件。这是因为radutmp文件中的敏感信息，并在第3章，FreeRADIUS入门中已进行过介绍。
#radzap
像radwho这样的radzap命令也可能需要在FreeRADIUS日志目录中存在sradutmp文件。它有各种开关，但并非所有开关都可以工作。在切换仍在工作的NAS时，请小心使用radzap，例如，使用带有-N <NAS IP Address>开关的radzap。这可能导致NAS请求FreeRADIUS更新已关闭的会话。当您使用sql进行计费时尤其如此。
有了这个，我们总结了基本会计部分。下一节将介绍限制用户会话的方法。