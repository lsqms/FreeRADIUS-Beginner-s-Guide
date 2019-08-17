# 行动时刻 - 拒绝没有领域的请求
以下步骤将演示如何拒绝没有领域的请求：
1. 编辑FreeRADIUS配置目录下的proxy.conf文件，并确保my-org.com域没有nostrip指令（它包含在上一个练习中）。
2. 编辑已sites-enabled/default(启用站点的/默认文件)，并在授权部分中的后缀条目之后添加以下unlang代码。 这将拒绝任何没有领域的用户名请求：
```
if( request:Realm == NULL ){
	update reply {
		Reply-Message := "Username should be in format username@domain"
	}
	reject
}
```
3. 在调试模式下重新启动FreeRADIUS服务器并尝试作为alice进行身份验证。 身份验证请求应该失败。
4. 认证为alice@my-org.com。 请求应该通过。

## 刚刚发生了什么？
我们设法拒绝用户名不包含域的身份验证请求。
我们必须在后缀模块之后放置unlang代码，因为它将Realm设置为NULL。 然后，我们对请求属性列表中的Realm属性的值执行简单检查。 如果它为NULL，我们将使用相关消息拒绝身份验证请求。。

## 默认领域
在本练习开始时，我们说realms模块（例如后缀）使用了三个特殊的realms：

+ NULL域（如果已定义）用于任何用户名中没有域的用户。 Stripped-User-Name属性设置为与User-Name属性相同的值。 Realm属性将设置为NULL。
+ LOCAL域是一个始终存在的领域，如果控制：Proxy-To-Realm被指定为LOCAL，则不会发生代理。当定义领域时，在领域定义内没有任何外部服务器或虚拟服务器时，也会使用LOCAL领域。 LOCAL领域的另一个用途是取消代理请求并在本地处理请求。
+ DEFAULT域（如果已定义）用于包含未知领域的任何请求。 DEFAULT域定义几乎总是包含nostrip选项，以帮助上游服务器在域之间进行区分。当您将请求转发到Eduroam服务器等上游服务器时，通常会使用此方法。简而言之，它匹配所有未接收的领域。有关说明，请参见下图：

从图中我们可以看到，具有未知领域的用户通过后缀模块分组到DEFAULT域中。 从那里，请求通常在上游转发。 此原则类似于TCP / IP协议的默认网关。

> 小心创建无限循环
> 当两个组织在它们之间配置漫游时，人们常常犯的错误是my-org.com只是将未知用户代理到your-org.com。 然后，your-org.com将其服务器配置为简单地将未知用户代理到my-org.com。 这显然会创造一个无限循环！
> 记下这一点，小心！

## 在结束时
这就把我们带到了第一节关于领域的结尾。关于后缀模块的工作需要记住三个要点：

+ 它根据proxy.conf文件中的预定义域标识用户的域，并相应地设置控件：Proxy-To-Realm值。
+ 它添加了一个请求：Realm属性，如果用户是预定义领域的一部分。这包括特殊域NULL和DEFAULT。
+ 如果用户的预定义领域不包含nostrip选项，则后缀模块将添加请求：Stripped-User-Name属性。

> 小心旧文档
> 您可能会遇到文档，指示您在领域文件中定义领域。它还可以讨论可以在领域定义中使用的诸如notrealm和提示之类的选项。领域(realms)文件和这些选项不再使用。我们现在使用proxy.conf文件来定义领域。阅读proxy.conf文件中的注释，以发现当前允许的选项。
在下一节中，我们将看到如何向领域添加指令将使其转发请求。






























