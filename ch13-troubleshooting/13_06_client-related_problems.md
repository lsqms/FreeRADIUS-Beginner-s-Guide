# 与客户有关的问题
用户连接RADIUS客户端; RADIUS客户端又连接到RADIUS服务器。如果客户端似乎无法与服务器连接，请首先检查以下内容：
+ FreeRADIUS知道这个客户吗？检查FreeRADIUS日志文件中的以下行：
`Wed May 18 17:53:57 2012 : Error: Ignoring request to authentication address * port 1812 from unknown client 192.168.1.103 port 39881`
+ 客户端是否允许通过FreeRADIUS服务器上运行的防火墙？要检查防火墙规则，请使用以下命令（需要root访问权限）：
` / sbin / iptables -L -n`
如果这些初始检查通过，请在调试模式下运行FreeRADIUS以进行正确的故障排除。调试消息将显示何时收到请求及其处理方式。这些调试消息很冗长，包含大量细节，因此很容易理解。
不幸的是，当你在生产环境中运行FreeRADIUS时，首先停止FreeRADIUS服务器然后在调试模式下启动它以进行故障排除并不总是那么容易。您将遇到的第二个问题是识别来自问题客户端的请求，以及同样转到FreeRADIUS的所有其他请求。为了解决这两个问题，我们可以将控制套接字虚拟服务器与raddebug程序结合使用。
另一种选择是虚拟化。今天，大多数大企业正在转向虚拟化环境。这使得故障排除和测试新配置变得更加容易。生产虚拟服务器的副本可用于测试或配置更改。这最大限度地减少了生产环境的中断。

## 测试与RADIUS服务器的UDP连接
大多数人都熟悉使用telnet测试与指定TCP端口的连接。 不幸的是，我们不能在FreeRADIUS上使用telnet测试，因为它运行在UDP而不是TCP上。
使用netcat和nmap等程序测试UDP连接并不能真正清楚地表明是否可以连接到UDP端口。 使用诸如radtest或radclient之类的RADIUS客户端程序来测试UDP连接要高效得多。 这些客户端程序还将报告错误的共享密钥，如下所示：
```
rad_recv: Access-Reject packet from host 192.168.1.42 port 1812, id=62, length=34
rad_verify: Received Access-Reject packet from home server 192.168.1.42 port 1812 with invalid signature! (Shared secret is incorrect.)
```

控制套接字(control-socket )虚拟服务器
FreeRADIUS具有控制套接字虚拟服务器，允许您控制正在运行的服务器。默认情况下，此虚拟服务器在SUSE和CentOS上启用，但在Ubuntu上不启用。虚拟服务器内的注释如下：
高度实验！它不应该在生产环境中使用。
实际上存在第二个问题。事实上，这个虚拟服务器是如此方便，您只需修改这个服务器上的规则即可！此虚拟服务器允许以下程序连接到FreeRADIUS控件套接字：
+ radmin：这是一个FreeRADIUS服务器管理工​​具，它连接到正在运行的服务器的控制套接字，并为其提供命令行界面。
+ raddebug：这是一个围绕radmin的shell脚本包装器，它自动执行从正在运行的服务器获取调试输出的过程。与使用radiusd -X不同，它可以在不影响服务可用性的情况下实现此目的。
这两个程序都包含自己的手册页，以便更详细地说明如何使用它们。
在读/写模式下激活控制套接字虚拟服务器时附加了安全警告（raddebug和radmin需要）。对我来说，raddebug的便利性超过了额外的风险。






