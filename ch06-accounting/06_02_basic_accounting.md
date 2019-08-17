# 基本计费
计费是指跟踪用户对NAS资源的消耗。 计费不仅包括以结算形式进行的成本回收。 它还可用于容量规划，生成趋势图，以及了解有关给定时间点的资源使用情况的更多信息。 在本章中，我们将了解如何在FreeRADIUS中完成计费。

FreeRADIUS是一个AAA服务器。 RADIUS中的AAA可以分为两个组件。一个组件包括授权和认证，它使用UDP端口1812.第二个组件是计费并使用UDP端口1813.这两个组件彼此独立地运行。 radiusd.conf文件中的不同监听部分确认了这一点。计费的监听部分如下：
```
This second "listen" section is for listening on the accounting port, too.
listen {
	ipaddr = *
	ipv6addr = ::
	port = 0
	type = acct
	interface = eth0
	clients = per_socket_clients
}
```
这个监听部分使FreeRADIUS监听计费请求。有关监听部分的更多信息，请参阅radiusd.conf中的注释。
请注意监听代码中的port = 0。当port指定为0时，FreeRADIUS将从/etc/services文件中读取端口的值。但是，您可以在启动期间通过传递-p <端口号>参数来覆盖此值，这将强制FreeRADIUS服务器仅侦听指定的端口。

> /etc/services文件用于将端口号和协议映射到服务名称。
> 
```
radius 1812/tcp
radius 1812/udp
radius-acct 1813/tcp radacct # Radius
Accounting
radius-acct 1813/udp radacct
```
> /etc/services文件将端口1645和1646称为old-radius和old-radacct。这些端口有时被其他RADIUS服务器使用。

上面的摘录表明FreeRADIUS默认能够处理计费请求。让我们看看计费是如何完成的。