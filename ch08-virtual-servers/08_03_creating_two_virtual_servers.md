# 行动时刻 - 创建两个虚拟服务器
在本练习中，我们将创建两个非常简单的虚拟服务器。 一个将接受所有身份验证请求，而另一个将拒绝所有身份验证请求。
1. 在FreeRADIUS配置目录内的sites-available目录下，使用以下内容创建名为always_accept的文件：
```
server always_accept {
	authorize {
		update control {
			Auth-Type := "Accept"
		}
	}
}
```
2. 在FreeRADIUS配置目录内的sites-available目录下，使用以下内容创建一个名为always_reject的文件：
```
server always_reject {
	authorize {
		update control {
			Auth-Type := "Reject"
		}
	}
}
```
3. 确保您位于FreeRADIUS配置目录中。 通过创建从启用站点的目录到site-available目录中刚刚创建的文件的符号链接来启用这些虚拟服务器：
```
ln -s ../sites-available/always_accept sites-enabled/always_
accept
ln -s ../sites-available/always_reject sites-enabled/always_
reject
```

### 刚刚发生了什么？
我们已经定义并启用了两个非常简单的虚拟服务器。 一个将始终通过身份验证请求，而另一个将始终拒绝身份验证请求。

## 可用的子部分
要创建虚拟服务器，我们使用命名服务器部分：
```
server <virtual server name> {
	...
}
```
然后在此服务器部分内添加各个子部分。 如果存在没有名称的服务器部分，则将其用作默认服务器部分。 如果在可以定义的部分（listen，client，home_server_pool，ttls和peap部分）中没有定义virtual_server指令，则将使用此默认服务器部分。
> 您可能已经观察到网站启用/默认文件甚至不包含服务器{...}部分。 将默认虚拟服务器包装在匿名服务器{...}部分中并不是绝对必要的。
> 但是，所有其他虚拟服务器都需要包装在命名服务器部分中。

将由虚拟服务器使用的子部分取决于发送到虚拟服务器的请求。下表列出了与请求相关的常见请求和子部分。

|Request|Secton|
|:-|:-:|
|Access-Request|授权，验证，会话，授权后|
|Accountng-Request|预先帐户，计费|

我们还可以使用预代理和后代理子部分。这是代理的一部分，并在本书后面介绍。还有两个特殊的子部分叫做listen和client。它们被称为特殊的，因为它们可以是FreeRADIUS的全局或虚拟服务器的本地服务器，具体取决于它们的定义位置。本章将介绍这两个小节。

我们创建的两个虚拟服务器完全负责授权部分中的Access-Request。不需要定义其他部分。在定义虚拟服务器时，请注意不要创建重复项。文件名与文件中定义的虚拟服务器之间没有直接关联。我们只使用它作为约定来保持文件名和虚拟服务器的名称相同。FreeRADIUS不关心文件名或文件中声明了多少个服务器部分。它甚至可以加载具有相同名称的多个服务器定义而不会出现错误。这可能会导致意外结果。
如果定义了额外的监听部分，还要确保它们连接到正确的接口并具有正确的IP地址，端口和类型。

## 启用和禁用虚拟服务器

请记住，启用虚拟服务器时，将在FreeRADIUS启动时加载该虚拟服务器使用的所有模块。虚拟服务器还可以引入额外的UDP端口，FreeRADIUS将监听请求。如果您有记忆和安全意识，那么禁用未使用的虚拟服务器是很好的做法。
我们的服务器已创建，愿意并已启用。我们带他们去试驾。








