# 行动时刻 - 使用control-socket和raddebug进行故障排除
如前所述，在Ubuntu上没有启用控制套接字虚拟服务器，在我们通过raddebug使用控制套接字之前，SUSE和CentOS上的默认安装需要进行一些调整。

## CentOS
在CentOS上执行以下操作以使raddebug可用：
1. 通过检查控制套接字(control-socket)虚拟服务器是否已在启用站点的目录下列出来确认它已启用：
`＃> ls /etc/raddb/sites-enabled`
2. 编辑控制套接字虚拟服务器文件，并确保在服务器部分底部将模式指定为mode = rw。
3. 确保radiusd用户属于root组：
`＃> /usr/sbin/usermod -a -G root radiusd`
4. 使用启动脚本重新启动FreeRADIUS：
`＃> /etc/init.d/radiusd restart`

## SUSE
在SUSE上执行以下操作以使raddebug可用：
1. 通过检查它是否列在sites-available目录下，确认控制套接字虚拟服务器已启用：
`＃> ls /etc/raddb/sites-enabled`
2. 编辑控制套接字虚拟服务器文件，并确保在服务器部分底部将模式指定为mode = rw。
3. SUSE默认以root身份运行FreeRADIUS，因此不需要将radiusd用户添加到根组以修复权限。 但是，如果您以用户radiusd而不是root身份运行FreeRADIUS，请记住将radiusd添加到root组。
4. 使用启动脚本重新启动FreeRADIUS。
`＃> /etc/init.d/freeradius restart`

## Ubuntu
在Ubuntu上执行以下操作以使raddebug可用：
1. 启用控制套接字虚拟服务器：
```
$> sudo su
#> cd /etc/freeradius/sites-enabled
#> ln -s ../sites-available/control-socket ./
```
2. 编辑控制套接字虚拟服务器文件，并确保在服务器部分底部将模式指定为mode = rw。
3. 确保freerad用户是根组的一部分：
`＃> usermod -a -G root freerad`
4. 使用启动脚本重新启动FreeRADIUS。
`＃> /etc/init.d/freeradius restart`

## 使用raddebug
raddebug程序在生产环境中真正节省了生命。让我们假设我们有一个生产环境，其接入点的IP为192.168.1.103，试图对用户进行身份验证。我们不知道共享的密钥是错的，但我们很快就会发现！
1. 确保您是FreeRADIUS服务器上的root用户，该服务器已按照前面所述激活和配置控制套接字。
为了使用raddebug，纯粹主义者可能会皱着眉头。然而，这证明在所有分布中给出的问题最少。
2. 从终端发出以下命令：
`＃> raddebug -t 300 -i 192.168.1.103`
3. （如果使用CentOS，则必须在raddebug命令之前使用以下命令以准备$ PATH
`variable : export PATH = $ PATH：/usr/sbin）`
4. 从IP地址为192.168.1.103的主机发送EAP身份验证请求，但使用错误的共享密钥。
5. 观察运行raddebug的终端上的输出：
```
Received Access-Request packet from host 192.168.1.103 port 41450, id=18, length=63
Cleaning up request 5 ID 18 with timestamp +253
```
6. 在FreeRADIUS日志文件上运行tail -f以观察日志文件的输出：
```
Thu May 19 21:20:47 2012 : Error: Received packet from 192.168.1.103 with invalid Message-Authenticator! (Shared secret is incorrect.) Dropping packet without response.
```
7. 修复共享密钥并执行另一个EAP身份验证请求。 raddebug的输出不应该显示合约：
```
Received Access-Request packet from host 192.168.1.103 port 34008, id=22, length=63
NAS-IP-Address = 192.168.1.103
User-Name = "alice"
EAP-Message = 0x0200000a01616c696365
```

## 刚刚发生了什么？
我们使用raddebug命令有选择地观察来自IP地址为192.168.1.103的客户端的请求的调试输出。

## 记住日志输出
raddebug命令仅报告调试消息，而不报告错误消息。 错误消息仍将记录到FreeRADIUS日志文件中。 与在调试模式下启动FreeRADIUS时相比，这是不同的。 当我们在调试模式下启动FreeRADIUS时，会在终端中报告错误消息以及调试消息。 出于这个原因，我们在日志文件中使用tail -f命令来查看那里报告的内容。

## 发现不匹配的共享密钥
当您查看使用PAP的Access-Request中的User-Password属性时，很容易发现不匹配的共享密钥。 此AVP的值将包含所有奇怪的字符，而不是用户的密码。 以下是调试输出中的一个示例：
`User-Password = "(*t\303v\230_\264\t;\211\221\343\024\343$"`

但是，在我们的练习中，我们使用的是EAP而不是PAP。 使用EAP，您必须采用不同的方法来检测错误的共享密钥。 客户端上最近的RADIUS实现包括请求中的Message-Authenticator AVP。 此AVP为RADIUS协议增加了额外的安全性，也是FreeRADIUS确认共享密钥是否正确的快捷方式。 如果FreeRADIUS从Message-Authenticator的值中获取共享密钥错误，它将只报告它并忽略该数据包。 这是我们在练习中经历的。
具有错误共享密钥的客户端实现（不包括Message-Authenticator AVP）在服务器上不使用PAP身份验证协议时检测起来非常困难。 FreeRADIUS配置目录下的clients.conf文件中的客户端定义可以使用以下指令强制使用Message-Authenticator：
`require_message_authenticator = yes`

> 有关Message-Authenticator AVP的更多信息以及对RADIUS协议的其他改进建议，您可以查看RFC 5080。
始终尝试使用包含Message-Authenticator AVP的RADIUS客户端。这使得RADIUS协议更加安全，并且可以轻松检测错误的共享密钥。另请注意，尽管FreeRADIUS支持最多31个字符的共享密钥，但并非所有客户端设备都支持该多个字符的共享密钥。当事情不能正常工作时，这些设备可能不会总是通知您这一限制，从而导致混淆。
## raddebug的选项
raddebug命令有两个方便的选项，如下所示：
<-u name：此选项允许您指定将用于过滤请求的用户名。仅显示具有User-Name == name的请求。
<-i ipv4-address：此选项允许您指定客户端的源IP地址以过滤请求。仅显示来自Packet-Src-IP-Address == ipv4-address的请求。
还有一个-c选项可用于创建更复杂的条件。这些条件是使用unlang语法创建的。

## Raddebug自动终止
raddebug手册页提到默认情况下会在十秒后终止。 这个持续时间因系统而异，通常要长得多。 在练习中，我们将持续时间增加到300秒（5分钟）。 该手册页还提到-t 0选项将让它永远运行。 但是，这不适用于我的任何服务器。

## 如果raddebug没有输出
如果您没有从raddebug获得任何输出，请确认运行FreeRADIUS服务器的用户，例如，radiusd是root组的成员，并且在使其成为root组的成员后重新启动了FreeRADIUS服务器。 以root身份运行FreeRADIUS时，不需要它。

> 定义客户端时的一些提示
> client.conf文件中定义的新客户端未加载SIGHUP信号。 你必须重新启动freeradius才能使用它们。如果在SQL表中定义客户端，则同样适用。 但是，有一个名为dynamic-clients的虚拟服务器可用作支持新客户端的引用，而无需重新启动FreeRADIUS。

如果您决定创建一个脚本，只要配置了新客户端，该脚本将自动重启FreeRADIUS，请记住在实际重新启动服务器之前进行配置检查（-C）。 如果引入导致服务器无法启动的配置错误，则无法执行此操作可能会出现问题。

下一节将介绍用户通过本节中讨论的RADIUS客户端进行身份验证时可能出现的问题。









