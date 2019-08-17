# 行动时刻 - 模拟代理计费
在my-org.com的FreeRADIUS服务器上执行以下操作：
1. 将目录更改为my-org.com的FreeRADIUS服务器上用于模拟bob@your-org.com计费的文件所在的目录。
2. 确保FreeRADIUS在代表my-org.com和your-org.com的服务器上以调试模式运行。
3. 在my-org.com服务器上发出以下命令：
`$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt`
这是为了模拟会话的开始。
4. 在my-org.com服务器上发出以下命令以结束上一个会话：
`$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_stop.txt`
5. 观察两台服务器上的调试输出，以查看默认情况下如何进行计费请求的代理。

## 刚刚发生了什么？
我们模拟了一个典型的计费请求，该请求从一个RADIUS服务器代理到另一个组织的主服务器。

## 计费代理请求的流程
下图显示了代理到另一台服务器的请求与本地处理的请求之间的流量差异：

![Flow_of_an_accounting_proxy_request](https://github.com/lsqms/FreeRADIUS/blob/master/image/ch12/Flow_of_an_accounting_proxy_request.PNG?raw=true)

您会注意到默认情况下计费记录在两台服务器上。 您可以使用unlang创建if条件，以防止在转发服务器中记录计费数据。

服务器中断后更新计费记录
一个常见问题是如何处理对已关闭的主服务器的请求。 当主服务器关闭或响应太慢时，您将在FreeRADIUS日志文件中看到类似以下内容：
`Tue Jul 5 19:13:16 2012 : Error: Rejecting request 2310 due to lack of any response from home server your-org-1:1813`

FreeRADIUS在这种情况下使用的原则是在本地服务器上编写详细的日志文件，然后在该日志文件上使用带有侦听器的虚拟服务器，以便在主服务器再次启动时转发请求。
在FreeRADIUS的2.x版本发布之前，这种类型的功能不是核心的一部分，通常是在一个名为radrelay的程序的帮助下完成的。 现在内置了这个功能。FreeRADIUS目录下的sites-available文件夹下有四个示例虚拟服务器，它们显示了实现写入详细日志然后在这些日志文件上创建侦听器的功能的可能方法。  以下列表给出了虚拟服务器文件的名称及其功能的简要说明。

+ buffered-sql：将SQL中存储的长期记帐数据（慢）与RADIUS服务器运行时（快速）所需的“实时”信息分离。 它用于加快速度。
+ copy-acct-to-home-server：在负载均衡或故障转移服务器集之间启用信息复制。
+ decoupled-accounting：与buffered-sql配置类似。 创建一个虚拟服务器，用于将详细信息写入文件，并创建另一个虚拟服务器来侦听此文件。
+ robust-proxy-accounting：仅在对主服务器的代理请求失败时写入详细信息。 然后，这些失败请求的侦听器将尝试将它们转发到指定的主服务器。

## 试一试 - 实现强大的代理会计功能
以robust-proxy-accounting文件为例，修改自己的设置，使网络中断时更加可靠。




















