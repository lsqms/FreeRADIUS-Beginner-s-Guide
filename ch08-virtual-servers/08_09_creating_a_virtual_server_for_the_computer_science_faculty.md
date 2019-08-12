# 行动时刻 - 为计算机科学系创建一个虚拟服务器
Isaac发现，计算机科学系的教师通过默认设置来实现安全。 其RADIUS服务器使用端口2812进行身份验证，使用2813进行记帐。 它有一个用户文件，其中包含用户的entre详细信息。 RADIUS客户端仅向RADIUS服务器发送身份验证请求。 下表列出了重要信息：

|Informaton item|Detail|
|:-|:-:|
|User store|users file|
|Authentication port |2812|
|Accountng port (unused)|2813|
|Computer Science RADIUS server IP Address|10.10.0.100|
|RADIUS client IP Address (authentcaton only)|10.10.0.200|

## 合并实施
根据Isaac收集的信息，我们可以做以下事情：
+ 创建命名文件部分以处理用户文件
+ 使用本地侦听和客户端部分创建虚拟服务器
+ 合并这个新的虚拟服务器

我们来解决这些问题！

## 一个名为fles的部分
模块特定配置通过位于FreeRADIUS配置目录下的modules子目录中的文件完成。 文件模块的默认行为是在files文件中指定的。 默认情况下，files模块会提供用户文件以确定是否已定义用户。  源的特定文件是可配置的，由usersfile指令指定。
如果我们想要合并第二个用户文件，我们只需创建一个额外的命名文件部分。 原始文件部分未命名，因为它通常是配置中唯一的部分。 必须命名所有后续文件部分。
+ 在FreeRADIUS配置目录内的modules目录下，创建一个名为files_cs的文件，其中包含以下内容：
```
files files_cs{
	usersfile = ${confdir}/users_cs
	acctusersfile = ${confdir}/acct_users
	preproxy_usersfile = ${confdir}/preproxy_users
	compat = no
}
```
+ 使用以下内容在FreeRADIUS配置目录中创建名为users_cs的文件：
```
"bob" Cleartext-Password := "passbob"
	Reply-Message = "Hello, %{User-Name}"
```

## 计算机科学系的虚拟服务器
服务器部分允许我们将各种监听和客户端子部分声明为服务器部分的本地部分。 要在单个服务器部分中包含计算机科学系的配置，我们将使用这些子部分。
在FreeRADIUS配置目录下的sites-available子目录中创建一个名为faculty_cs的文件，其中包含以下内容：
```
server faculty_cs {
	listen {
		ipaddr = *
		port = 2812
		type = auth
	}
	client cs_vpn {
		ipaddr = 10.10.0.200
		secret = bigone
		require_message_authenticator = no
		nastype = other
	}
	client cs_troubleshoot {
		ipaddr = 127.0.0.1
		secret = bigone
		require_message_authenticator = no
		nastype = other
	}
	authorize {
		files_cs
		pap
	}
	authenticate {
		Auth-Type PAP {
			pap
		}
	}
}
```

确保您位于FreeRADIUS配置目录中。 通过创建指向启用站点的目录的符号链接来启用faculty_cs虚拟服务器：
`ln -s ../sites-available/faculty_cs sites-enabled/faculty_cs`


## 合并新的虚拟服务器
现在，我们应该准备好尝试新的虚拟服务器了：
1.在调试模式下重启FreeRADIUS。
2.尝试使用faculty_cs虚拟服务器使用以下命令验证bob身份：
$> radtest bob passbob 127.0.0.1:2812 100 bigone
3.确认在用户文件中定义并由默认虚拟服务器使用的Alice未在Faculty_C虚拟服务器上进行身份验证：
$> radtest alice passme 127.0.0.1:2812 100 bigone
### 刚刚发生了什么？
我们刚刚证明了使用虚拟服务器将不同的RADIUS服务器整合到一起是多么容易。
## 那些存储在SQL中的用户呢？
您可能会问的一个问题是：假设我们将用户数据存储在SQL数据库中，并且计算机科学教员的用户也存储在SQL数据库中，我们如何整合这个？这可以通过在sql.conf文件中定义多个sql实例来完成：
sql sql_canteen {
	..
}
sql sql_cs {
	..
}
然后在虚拟服务器中使用您想要的名称而不是sql。这与我们在文件模块中应用的原理相同。
## 当IP地址和端口发生冲突时
我们很幸运，因为计算机科学系的教师没有使用1812的默认端口进行身份验证。如果它也使用端口1812，我们将不得不通过其他方式将请求分配给其虚拟服务器。通常的方法是为FreeRADIUS服务器的网络接口分配第二个IP地址。以下是当多个IP地址用于虚拟服务器分配时要遵循的一些一般规则。
+ 使用ifconfig命令将第二个IP地址添加到网络接口。
`ifconfig eth0:0 10.10.0.100 netmask 255.255.255.0 up`
+ 更新指定listen=*to listen=<ip address>的所有其他侦听部分。您可能需要添加可选的监听部分，以包括127.0.0.1和网络接口的第一个IP地址，以使默认设置像以前一样工作。

## 本地监听和客户端部分
通过在虚拟服务器定义中指定监听和客户端部分，我们保持虚拟服务器的统一性。由于客户端和侦听部分已经是虚拟服务器的一部分，因此可以轻松地将虚拟服务器配置从一个物理服务器传输到另一个物理服务器。
由于本地侦听和客户端部分已经在虚拟服务器中，因此我们不能在其中使用virtual_server指令。客户定义是直接的。但是listen部分有一个带有一些有趣选项的类型指令。
## IPv6
FreeRADIUS支持侦听IPv6地址作为IPv4地址的替代方案。因为与更成熟的IPv4代码相比，FreeRADIUS服务器的这个组件更新，所以可能仍然存在错误修复和改进。建议您在使用IPv6时使用最新的可用版本。例如，随Ubuntu 8.04一起提供的FreeRADIUS 2.1.8版软件包有一个错误，阻止它监听IPv6地址。
随着越来越多的管理员使用IPv6地址，这些初始问题将得到解决。不幸的是，我们在ipv6中有一个鸡和蛋的场景，现在有许多供应商，因为他们首先等待ipv6的大量使用，然后再调整radius客户机代码以支持ipv6寻址。
## 监听部分→类型指令
到目前为止，我们已经在listen部分中使用了type指令的auth和acct选项。但是，该指令还有其他选项可供使用。其中一些用于更高级的配置，本书稍后将对此进行介绍。为了完整起见，我们将在此列出：
|Opton|Where used|
|:-|:-:|
|proxy|Proxy requests to other RADIUS servers|
|detail|High-performance deployments or with requests that have to be send to multple databases|
|status|To get stats from the FreeRADIUS server|
|coa|To forward disconnect requests to other RADIUS servers|

定义客户端时，无法在客户机定义内限制我们可以从客户机接收的请求类型。 限制是通过listen部分完成的。 Listen部分定义了我们将响应的请求类型。 我们只指定了auth。 如果我们还想响应计费请求，我们必须添加第二个type = acct的listen部分。
本章的下一部分将介绍一些预定义的虚拟服务器。 它们每个都包含一个本地监听部分，以满足特殊要求。
