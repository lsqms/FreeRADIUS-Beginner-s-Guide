# 行动时间 - 包含到期和行记录模块
Isaac怀疑该大学的一些学生试图非法获得Wi-Fi接入。 他想将所有失败的身份验证尝试记录到专用日志文件中。 当他参与其中时，他还希望为每个学生添加一个到期日期，以防止他们在学期结束后进入网络。 为了实现这一点，他利用了FreeRADIUS中的expiration和linelog模块。 让我们看看它是如何完成的：
1. 编辑FreeRADIUS配置目录下modules子目录下的linelog文件。 更改以下行：
`Access-Request = "Requested access: %{User-Name}"` 更改为：
```
Access-Request = "Request access: %{User-Name} %{UserPassword} from %{NAS-IP-Address} %{reply:Reply-Message}"
```
2. 编辑FreeRADIUS配置目录下的sites-enabled / default文件。 在post-auth部分中更改以下部分：
```
Post-Auth-Type REJECT {
	# log failed authentications in SQL, too.
	# sql
	attr_filter.access_reject
}
```
更改为：
```
Post-Auth-Type REJECT {
	# log failed authentications in SQL, too.
	# sql
	linelog
	attr_filter.access_reject
}
```
3. 确保FreeRADIUS配置目录下的sites-enabled / default文件在authorize部分下列出了expiration模块（默认包含它）。
4. 编辑FreeRADIUS配置目录下modules子目录中的expiration文件。 更改以下行：
`reply-message = "Password Has Expired\r\n"`
更改为：
`reply-message = "Dude, you are like sooooo expired\r\n"`
5. 编辑FreeRADIUS配置目录中的用户文件，并确保alice存在以下条目：
"alice" Cleartext-Password := "passme", Expiration := "4 May 2010"
6. 在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证：
$> radtest alice passme 127.0.0.1 100 testing123
7. 由于Expiration check属性的值，应拒绝该请求。
8. 此故障也应记录在/ var / log / freeradius / logread文件中（根据您的安装，名称和位置可能不同）。

## 刚刚发生了什么？
我们已经配置并使用了expiration和linelog模块。

## 配置模块
配置模块的约定是通过编辑FreeRADIUS配置目录中modules子目录下的配置文件。这些文件的名称与模块相同，并包含模块的单个部分。第一个实例是未命名的。它类似于编程语言中的匿名子例程。
```
expiration {
	reply-message = "Dude, your account is like in sooooo expired\r\n"
}
```
随后使用具有不同设置的相同模块可以通过在其单独文件中为模块创建命名部分来完成。以下是modules子目录中exp_professors文件的内容：
```
expiration exp_professors {
	reply-message = "Dear professor %{User-Name}, kindly contact helpdesk about your expired account\r\n"
}
```
这与拥有一个对象和一个对象的各种实例非常相似。每个实例都可以具有不同的属性，并且独立于其他实例。然后，每个模块实例都可以在定义的任何虚拟服务器中使用任意次数。这给我们留下了一个模块实例池和一个虚拟服务器池，我们可以混合使用它们来创建非常灵活的配置。

模块的原始配置文件包含许多在配置模块时派上用场的注释。
> 命名约定
> 创建命名部分时，最好在名称中包含模块的名称。对于sql，您可以使用sql_primary和sql_secondary。在我们的示例中，我们维护了exp前缀。这有助于了解当它包含在一个部分中时涉及哪种类型的命名模块。

并非所有模块都遵循配置位于modules子目录下的文件中的约定。 radiusd.conf文件有一个modules部分，列出了要包含的所有模块。您会注意到那里列出的sql和eap模块违反了这条规则。这两个模块的配置文件直接位于FreeRADIUS配置目录下。

每个配置文件也不总是遵循一个模块实例的约定。例如，modules目录下的域配置文件声明了一些域实例。您需要决定是将所有实例保存在一个配置文件中，还是将它们分开，以便每个实例都有自己的文件。

## 使用模块
在配置模块后，可以使用它。模块的使用方式和位置在模块之间差异很大。要了解如何使用过期模块，请参阅随FreeRADIUS安装的文档。
rlm_expiration文档指定以下内容：
```
Module to expire user accounts.
This module can be used to expire user accounts. Expired users receive
an Access-Reject on every authentication attempt. Expiration is based
on the Expiration attribute which should be present in the check item
list for the user we wish to perform expiration checks.
Expiration attribute format:
You can use Expiration := "23 Sep 2004" and the user will
no longer be able to connect at 00:00 (midnight) on September 23rd,
2004. If you want a certain time (other than midnight) you can do
use Expiration := "23 Sep 2004 12:00".
The nas will receive a Session-Timeout attribute calculated to kick
the user off when the Expiration time occurs.
Example entry (users files):
user1 Expiration := "23 Sep 2004"
```

> 并非所有模块都有关于如何使用它们的说明作为单独的文档。 有时模块的配置文件也会包含使用说明。 或者参考FreeRADIUS Wiki或使用Google。

模块也可以使用两对或更多对。 unlang语言通过使用冗余，负载平衡和冗余负载平衡关键字来配置冗余，负载平衡或两者的组合。 这通常与sql和ldap模块一起使用，并在authorize部分中配置。

请记住，如果以这种方式使用ldap模块，并且还将其与bind-as功能一起使用以对用户进行身份验证，则Auth-Type将设置为LDAP。 然后还应配置authenticate部分下的Auth-Type LDAP声明，例如：
```
Auth-Type LDAP {
	#ldap
	redundant-load-balance {
		ldap_this
		ldap_that
	}
}
```

## 可以包含模块的部分
模块只能在指定的部分中指定。要在部分中包含模块，请在该部分中添加模块名称。如果要使用模块的命名实例，只需包含此命名值而不是模块名称。如果我们考虑使用教授的到期模块实例，它将被简单地列为exp_professors而不是expiration。
模块可以包含在虚拟服务器的所有常用部分中。这些是授权，身份验证，会话，preacct(预结算)，计费，授权后，预代理和后代理部分。模块也可以包含在实例化和eap部分中。 instantiate是radiusd.conf文件中的一个特殊部分，用于确保模块在虚拟服务器内的部分调用之前加载。在部分中包含模块之前，请确保该模块已写入以便在指定的部分中使用。
我们现在知道哪些部分可以包含模块。让我们看看在到期模块的不同实例上的实际实现。








