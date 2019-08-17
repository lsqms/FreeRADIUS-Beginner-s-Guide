# 行动时刻 - 执行基线速度测试
以下步骤将演示如何执行速度测试：
1. 使用第10章EAP作为指南安装和配置JRadius Simulator。
2. 通过增加请求线程和每线程请求的值来测试FreeRADIUS的响应时间，观察FreeRADIUS达到饱和点的值。
3. 测试在FreeRADIUS服务器上完成典型事务的持续时间。 例如，如果您是Eduroam的一部分，您可以记录您支持的各种EAP方法的持续时间。 您还可以测试计费请求的速度。

## 刚刚发生了什么？
您已通过使用JRadius Simulator程序对FreeRADIUS执行了基线速度测试。
FreeRADIUS在隔离中表现良好。 但是，当FreeRADIUS使用外部组件或服务器来处理请求时，由于请求的同步性，性能可能会降低。 下一个部分将帮助您最大限度地提高FreeRADIUS服务器本身以及外部组件（如文件，LDAP和SQL服务器）的性能。

## 调整FreeRADIUS的性能
此部分中的项目列表取自以下URL：http://freeradius.org/radiusd/doc/tuning_guide
该URL提供了一个方便的清单，您可以使用它来提高FreeRADIUS的性能。

## 主服务器
+ LDAP或SQL等可扩展的身份验证机制与大量用户和/或大量请求相关。
+ 在所有FreeRADIUS日志文件上启用noatime，或者更好地在FreeRADIUS日志目录中启用noatime。您可以使用/etc/fstab文件中的noatime挂载选项挂载整个文件系统，或者如果您有ext2类型的文件系统，则可以使用chattr命令添加A属性。
```
＃> chattr -R + A /var/log/freeradius/
＃> lsattr /var/log/freeradius/
```
+ noatime将禁用上次读取文件的记录，从而提高性能。
+ 不要使用detail和radwtmp（文件）模块。他们会减慢你的计费速度。但是，可以在另一个设置中使用，以取消对SQL记帐的耦合，这反过来可以加快速度。
+ 使用users文件仅设置默认配置文件。不要在那里放置任何用户。保持尽可能小。始终在users文件中设置默认属性，不要使用默认值填充LDAP / SQL中的用户条目。通常，LDAP / SQL用户配置文件应仅包含默认配置文件未提供的用户属性。
+ 调整线程池参数以匹配您的大小要求。将max_requests_per_server设置为零以避免服务器线程重新启动。
+ 增加网络访问服务器（NAS）中的超时（10秒）和重试（5-7）以进行计费。 这样你就不会丢失任何见覅诶信息。 如果你使用Mikrotk，它肯定会增加超时值，因为默认值只有100微秒。
+ 使用调整良好的快速以太网连接以最大限度地减少延迟。
+ 确保操作系统始终安装了最新的修补程序。

还有一些专门针对某些模块的提示，可用于加快速度。。

## LDAP模块
+ 调整FreeRADIUS配置目录下的modules/ldap文件中的ldap_connections_number，使其大于同时进行的用户身份验证请求的平均数。
+ 在LDAP服务器上，尝试最大化缓存。特别是，始终启用uid属性（相等索引(equality index)）和cn属性（相等索引 -  cn属性用于搜索组(equality index – the cn attribute is used to search for groups)）的索引。使LDAP服务器条目/目录缓存内存大小尽可能大。通常，尝试为LDAP服务器分配尽可能多的内存。
+ 在LDAP中放置默认配置文件。用户条目应仅包含非标准值，以便保持较小并最大化缓存用户默认/常规配置文件的收益。
## SQL模块
+ 调整FreeRADIUS配置目录下的sql.conf文件中的num_sql_socks，使其大于同时进行身份验证/记帐请求的平均数。
+ 在会话部分使用sql模块而不是radutmp模块。它工作得更快。
+ 为Username和AcctStopTime属性创建多列索引，尤其是在使用sql进行双重登录检测时。在MySQL shell中，您可以输入以下内容来执行此操作：
```
mysql> use radius;
mysql> ALTER TABLE radacct ADD INDEX myIndex（username，acctstoptime）;
```
+ 如果您使用的是MySQL并且进行了大量的计费工作，请尝试使用InnoDB作为radacct表而不是MyISAM。您可以使用MySQL shell中的以下命令来确定当前引擎：
`mysql> show table status from radius LIKE 'radacct';`
+ 要更改MySQL表的引擎，请发出以下命令：
```
mysql> use radius;
mysql> alter table radacct ENGINE = InnoDB;
```
+ 在accounting_stop查询中添加Acct-Unique-Session-Id。 特别是如果你有很多访问服务器或你的NAS不发送非常随机的Session-Ids。 这样，您将始终有一个候选行要搜索，而不是所有具有相同Acct-Session-Id的行。
+ 使用MySQL中的EXPLAIN语句来评估FreeRADIUS使用的SELECT语句，以帮助创建索引。
现在我们的服务器已经过微调，我们可以通过使用unlang内置的冗余和负载平衡功能使其更加可靠和快速。

## 冗余和负载平衡
以下是unlang提供的用于创建冗余，负载平衡或两者组合的关键字列表：
+ redundant：在虚拟服务器的授权或记帐部分内指定冗余部分。此部分只能包含模块列表。如果列表中的模块失败，则将尝试列表中的下一个模块，直到一个模块通过。
+ load-balance：负载平衡部分通常也在虚拟服务器的授权或记帐部分内指定。与冗余部分一样，它也只能包含模块列表。但是，模块必须是相同类型（例如：ldap或sql）才能使负载平衡公平地工作。为了处理请求，随机选择列表中的模块。
+ redundant-load-balance：冗余和负载均衡的组合。
大多数企业都有多个LDAP服务器。当您使用ldap模块时，通过使用冗余和负载平衡功能来进行更可靠的部署是有意义的。授权部分的以下代码段应该说明一切：
```
redundant-load-balance {
	ldap1 # 50%, unless ldap2 is down, then 100%
	ldap2 # 50%, unless ldap1 is down, then 100%
}
```

> FreeRADIUS Wiki包含两个页面，显示了更复杂的冗余和负载平衡可能性。 以下是链接：
> http://wiki.freeradius.org/Fail-over
> http://wiki.freeradius.org/Load_balancing


我们无法控制的事情
遗憾的是，某些RADIUS部署包括不受我们控制的网络部分。 如果客户端和FreeRADIUS服务器之间的连接不可靠或缓慢，那么FreeRADIUS服务器以超高速后端运行的事实将没有任何区别。 使用ping和traceroute等网络故障排除命令将是一个良好的开端，可以确定客户端向服务器发送请求以及响应这些请求的服务器是否存在延迟问题。 如果您无法对延迟做任何事情，可以考虑增加服务器或客户端配置中的一些超时。
拥有一个工作和快速的系统并不能保证FreeRADIUS永远不会崩溃。 下一节将讨论一种让FreeRADIUS在发生故障宕机后恢复的方法。










































