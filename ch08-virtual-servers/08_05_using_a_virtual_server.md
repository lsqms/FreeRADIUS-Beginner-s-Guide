# 行动时刻 - 使用虚拟服务器
请按照以下步骤使虚拟服务器可用：
1.编辑FreeRADIUS配置目录中的radiusd.conf文件，并将以下内容添加到包含type = auth的listen部分（有两个listen部分，一个有type = auth，另一个有type = acct）：
virtual_server = always_accept。
2.在调试模式下重新启动FreeRADIUS。
3.尝试使用任何密码验证任何用户。您的请求应该每次都被接受。
4.当FreeRADIUS接受请求时，观察调试输出。
5.再次编辑radiusd.conf文件，但这次更改了
virtual_server指令，从virtual_server = always_accept到virtual_server = always_reject。
6.在调试模式下重新启动FreeRADIUS。
7.尝试验证是否使用任何密码指定任何用户。您的请求应该每次都被拒绝。
8.当FreeRADIUS拒绝请求时，观察调试输出。
9.完成本练习后，再次注释virtual_server指令。这将使FreeRADIUS服务器在练习之前保持原样。
## 刚刚发生了什么？
我们使用了本章第一次实际练习中创建的两个虚拟服务器来覆盖默认的虚拟服务器。首先接受所有认证请求，然后拒绝所有认证请求。
## 包括虚拟服务器
可以在listen或客户端部分中指定虚拟服务器的使用。 listen部分在radius.conf文件中定义，客户端部分包含在clinets.conf文件中。它是通过在这些部分中添加可选的virtual_server指令来指定的。当我们在listen部分中指定虚拟服务器时，它更通用，因为listen部分指定了客户端如何连接到FreeRADIUS的详细信息。这包括FreeRADIUS将从任何人接收的请求类型以及它为这些请求监听的IP地址和端口。

当我们在客户端部分中指定虚拟服务器时，它是特定的。 除非客户端使用指定的IP地址和共享密钥连接，否则将不使用虚拟服务器。
客户端部分定义客户端并将其添加到配置中。 您可以通过在localhost客户端中指定virtual_server指令来重复上一个练习
定义与我们在listen部分中指定的方式相同。 这将仅适用于来自localhost的请求，例如，当从运行FreeRADIUS服务器的同一台机器执行radtest时。

## 正确处理Post-Auth-Type
如果在拒绝身份验证请求时查看调试输出，您将看到以下警告：
```
Using Post-Auth-Type Reject
WARNING: Unknown value specified for Post-Auth-Type. Cannot perform
requested action.
Delaying reject of request 0 for 1 seconds
```
这是因为Post-Auth-Type具有未在虚拟服务器的post-auth部分中处理的值。 为此，请将始终拒绝的飞行更新为以下内容，警告将消失：
```
server always_reject {
	authorize {
		update control {
			Auth-Type := Reject
		}
	}
	post-auth {
		Post-Auth-Type REJECT {
			noop
		}
	}
}
```
### 注意类型属性
可以包含在虚拟服务器中的五个部分具有附带的特殊属性。 设置此特殊属性后，FreeRADIUS将在该部分中查找子部分以处理此属性的值。 该小节的格式如下：
```
<special attribute> <value> {
	...
}
```

如果将此特殊属性设置为某个值，则会导致FreeRADIUS查找具有该值的子节。 只会执行此子部分。 该部分内的其他所有内容都将被忽略。 如果未定义子部分，则会出现Post-Auth-Type = REJECT的警告。
下表列出了这些特殊属性及其应用的部分。 它还列出了它们的使用应用。 您会注意到它们都以“类型”一词结尾。

|Special atribute|Apply to secton|Practcal implementaton|
|:-|:-:|-:|
|Post-Auth-Type|post-auth|将故障记录到单独的数据库|
|Auth-Type|authenticate|如果ldap模块在LDAP目录中找到用户，请使用ldap进行身份验证|
|Authz-Type|authorize|根据用户的check属性使用不同的LDAP服务器|
|Acct-Type|accounting|根据用户的check属性指定应使用的计费数据库|
|Session-Type|session|指定每个用户应如何进行会话检查|

在我们的例子中，我们从未明确地将Post-Auth-Type设置为REJECT; 但是，当Auth-Type的值更改为Reject时，rlm_reject为我们设置Post-AuthType = REJECT的值。 这使我们能够在访问拒绝和访问接受之间进行区分，并且还可以不同地处理它们。

您还可以参考默认虚拟服务器中的身份验证部分。 authenticate部分使用Auth-Type属性，该属性由authorize部分内的模块设置。 下一个练习将是虚拟服务器的实际实现。








