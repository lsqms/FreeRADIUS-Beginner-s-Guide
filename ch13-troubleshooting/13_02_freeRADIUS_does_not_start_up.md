# FreeRADIUS无法启动
所以你很想开始这个名为radiusd的程序。您已以root身份登录，在终端提示符下键入radiusd，按Enter键，您将获得以下内容：
`radiusd: command not found`
我知道这听起来很愚蠢，但请确保使用以下命令实际安装了FreeRADIUS：
`locate radius`
> 并非所有发行版都默认包含locate命令。在SUSE上，您可能首先必须通过运行以下命令来安装它：`zypper in findutils-locate`

如果您确定FreeRADIUS存在但未启动，请尝试识别服务器二进制文件。 在Ubuntu和Debian系统上，二进制文件称为freeradius而不是radiusd。 如果您有正确的二进制文件，请使用-X选项启动FreeRADIUS以显示可帮助您识别问题的调试消息。 某些发行版（如CentOS）不在root用户的路径中包含/usr/sbin目录。 然后，您必须在shell中输入绝对路径和二进制名称，以便FreeRADIUS启动。
以下列表提到了FreeRADIUS无法启动的常见原因：

+ 端口1812和1813已在使用中。
+ 配置存在问题。
+ 缺少模块或库。
+ FreeRADIUS连接到无法正常工作的外部组件。

## 谁在使用我的端口？
罪魁祸首通常是在启动期间由启动脚本启动的另一个FreeRADIUS实例。 当您尝试启动时，FreeRADIUS将以类似于以下内容的消息结束。 这意味着另一个程序正在使用FreeRADIUS也想使用的UDP端口：
```
Failed binding to authentication address * port 1812: Address already in use
/etc/freeradius/radiusd.conf[240]: Error binding to port for 0.0.0.0 port 1812
```
以下命令将显示计算机上的所有UDP侦听器：
＃> netstat -uanp
请注意，您必须位于root用户才能使用netstat命令的-p选项。 -p选项将显示使用列出的端口的进程名称。 在我的Ubuntu机器上返回以下内容：
```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address Foreign Address State	PID/Program name
udp 0 0 0.0.0.0:68 0.0.0.0:*	605/dhclient3
udp 0 0 0.0.0.0:1812 0.0.0.0:*	1554/freeradius
udp 0 0 0.0.0.0:1813 0.0.0.0:*	1554/freeradius
udp 0 0 0.0.0.0:1814 0.0.0.0:*	1554/freeradius
```

您将注意到FreeRADIUS也使用端口1814.此端口不会侦听请求，而是发送请求，例如，在代理期间，当FreeRADIUS充当领域的主服务器的客户端时。 要关闭现有的FreeRADIUS实例，请使用启动脚本及其stop选项或killall命令。 以下是CentOS上使用这两个选项的方法：
```
/etc/init.d/radiusd stop
killall radiusd
```
如果netstat命令的输出包含不需要的信息，则通过grep命令管道此输出以搜索某个短语非常方便。 以下命令仅列出freeradius进程使用的UDP端口：
`netstat –uanp | grep freeradius`

## 检查配置
FreeRADIUS有一个-C选项，用于检查配置。 使用-XC选项启动FreeRADIUS将报告配置中是否存在明显错误。 这种检查不是万无一失的，可能会发生FreeRADIUS通过此检查，但仍然无法启动。 但是，调试消息将在大多数情况下指向问题。
FreeRADIUS在配置文件中使用特殊关键字$ INCLUDE来包含其他文件但不检查以防止递归包含。 这可能导致FreeRADIUS以递归方式读取文件以将其包含在配置中并最终放弃。 如果发生这种情况，问题通常在于最后一个包含行。 以下是包含损坏的字典文件的系统的输出：
```
including dictionary file /etc/freeradius/dictionary
Errors reading dictionary: dict_init: /usr/share/freeradius/
dictionary[57]: Couldn't open dictionary "/usr/share/freeradius/
dictionary.compat": Too many open files
```

## 找到丢失的模块或库
找到一个缺少的模块并不总是简单地安装包含这个缺失模块的FreeRADIUS软件包。 由于FreeRADIUS允许我们通过为模块的后续实例提供名称来定义模块的各种实例，因此任何名称都可以出现在虚拟服务器部分中以表示模块实例。 我们不一定知道此实例的功能或该实例派生自的模块。

为了防止将来出现这种情况，最好在所使用的模块的实例名称中提供一个使用哪个模块的提示。 默认情况下定义的attr_filter实例用是很好的例子，如attr_filter.access_reject。

如果不需要缺少模块，可以在配置文件中注释掉它。 调试输出指示哪个文件，部分以及列出缺失模块的行，例如：
```
/etc/freeradius/sites-enabled/default[160]: Failed to find module "frbg".
/etc/freeradius/sites-enabled/default[62]: Errors parsing authorize section.
```

如果需要丢失模块，请检查它是否未包含在尚未安装的其他FreeRADIUS软件包中。 最后，您可以尝试从源代码编译FreeRADIUS，确保在编译FreeRADIUS期间包含所有必需的开发库以创建此模块。 例如，较旧版本的Ubuntu不包括对EAP-TTLS的支持，这是您必须采取的路线，以便在FreeRADIUS中包含EAP-TTLS支持。
如果您使用configure，make，make install模式编译了FreeRADIUS，则在尝试启动FreeRADIUS时可能会出现以下错误：
```
radiusd: error while loading shared libraries:
libfreeradius-radius-2.1.10.so: cannot open shared object file: No such file or directory
```
这是因为操作系统还不知道新安装的库的存在和位置。 如果运行ldconfig命令，则应该修复它。

修复损坏的外部组件
有些模块依靠外部组件来完成部分工作。 FreeRADIUS在这些外部组件出现问题时的反应方式因模块而异。 让我们讨论三种可能性：
FreeRADIUS拒绝启动
perl模块调用外部Perl脚本。 在启动期间，此脚本与Perl运行时一起加载到内存中。 但是，如果外部Perl脚本包含错误，FreeRADIUS将无法启动。 外部Perl脚本中的错误位置将显示在调试输出中。
> FreeRADIUS在启动期间不测试Perl脚本的执行。 只有在调用perl模块来处理请求时才会执行Perl脚本。 在执行期间，Perl脚本也可能失败。

## 尽管显示错误消息，FreeRADIUS仍会运行
sql模块将在启动期间创建与数据库的连接。 如果数据库服务器关闭，FreeRADIUS仍将启动，但会在日志文件或调试输出中报告。 以下是日志文件中的一个片段，显示FreeRADIUS无法连接到MySQL数据库：
```
Tue May 17 19:21:02 2012 : Info: rlm_sql_mysql: Starting connect to MySQL server for #0
Tue May 17 19:21:02 2012 : Error: rlm_sql_mysql: Couldn't connect socket to MySQL server radius@localhost:radius
Tue May 17 19:21:02 2012 : Error: rlm_sql_mysql: Mysql error 'Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)'
Tue May 17 19:21:02 2012 : Error: rlm_sql (sql): Failed to connect DB handle #0
```
请务必定期检查生产系统的日志文件，以确定这些潜在问题。
当FreeRADIUS连接到外部服务器上的数据库时，请确保没有防火墙阻止从FreeRADIUS服务器访问数据库服务器。 MySQL等一些数据库还允许您指定主机以及数据库用户可以连接的用户名和密码。 这些问题可能很难追踪，因为在所有服务运行时一切似乎都可以。 不幸的是，一个小组件，如关闭防火墙上的所需端口，可以使FreeRADIUS处于中断状态。

## FreeRADIUS仅在回答请求时报告问题
ldap模块不会检查LDAP服务器在启动期间是否正常工作。 只有在调用ldap模块来处理请求时，才会发现此外部组件的任何问题。 然后，日志文件将报告失败，如下所示：
```
Tue May 17 22:59:36 2012 : Error: rlm_ldap: cn=binduser,ou=admins,ou=radius,dc=my-domain,dc=com bind to 127.0.0.1:389 failed: Can't contact LDAP server
Tue May 17 22:59:36 2012 : Error: rlm_ldap: (re)connection attempt failed
```
另请注意，外部组件可能随时出现故障。 为了最大限度地减少此类故障的影响，您可以使用FreeRADIUS包含的冗余功能作为unlang的一部分。

## 使用启动脚本
如果您确认FreeRADIUS可以从终端启动，也可以确保它使用启动脚本启动。 在FreeRADIUS日志文件上运行tail -f，同时通过启动脚本启动服务。
`＃> tail -f /var/log/radius/radius.log`
最后确保FreeRADIUS能够在重启后仍能正常工作。 请参阅第2章“安装”以了解如何在每个分发版本上激活启动脚本。

















