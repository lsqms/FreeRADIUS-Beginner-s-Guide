# 行动时刻 - 调查FreeRADIUS中的默认领域
在下面的练习中，我们将了解领域的各个方面。 我们将从FreeRADIUS的干净安装开始，然后进行更改以了解它如何处理各种领域配置：

> 确保您具有FreeRADIUS服务器的默认安装。

1. 编辑位于FreeRADIUS配置目录下的用户文件，并确保以下alice条目存在：
`"alice" Cleartext-Password := "passme"`
2. 在调试模式下重新启动FreeRADIUS服务器并作为alice进行身份验证。 观察FreeRADIUS服务器的输出。 以下应该是输出的一部分：
```
[suffix] No '@' in User-Name = "alice", looking up realm NULL
[suffix] No such realm "NULL"
++[suffix] returns noop
```

## 刚刚发生了什么？
我们对用户文件进行了正常的身份验证 - 这里没什么新东西。 但是，我们将关注于启用站点/默认虚拟(sites-enabled/default )服务器的授权部分内的模块。 该模块将自身标识为调试中的后缀模块
消息。

## 后缀模块
后缀模块是领域模块的实例（rlm_realm）。 该模块在FreeRADIUS配置目录下的modules / realm文件中定义：
```
# 'username@realm'
#
realm suffix {
	format = suffix
	delimiter = "@"
}
```

指示该模块检查用户名AVP是否为username@realm形式。 在modules/realm文件中还定义了realm模块的其他实例：

+ IPASS查找IPASS/username@realm形式的用户名。
+ realmpercent查找username%realm的用户名。
+ realmntdomain查找domain\user格式的用户名。

## NULL领域
FreeRADIUS中有三个具有特殊含义的领域。 它们是NULL，LOCAL和DEFAULT。 我们可以从调试输出中看到后缀模块实际上正在查找名为NULL的域上的信息。 因为它无法在NULL领域找到任何信息，所以它只是返回noop。
```
[suffix] No '@' in User-Name = "alice", looking up realm NULL
[suffix] No such realm "NULL"
++[suffix] returns noop
```
后缀模块将任何不包含后缀的用户名分组到NULL域中。这意味着任何没有域的请求都将自动处于NULL域。由于alice不包含@realm，后缀模块试图获取有关NULL域的信息，但无法得到任何并返回noop。

## 启用领域模块的实例
正如我们所见，后缀只是领域模块的一个实例。此实例在授权（Access-Request）期间以及preacct（Accounting-Request）期间使用。默认情况下仅启用后缀，但我们可以启用任何其他已定义的实例，例如IPASS。我们甚至可以声明我们自己的领域模块的命名实例，然后使用它。但是，默认值在大多数环境中都可以使用。
在下一个练习中，我们将看到后缀模块在哪里查找不存在的名为空的领域。

## 定义NULL域
领域在位于FreeRADIUS配置目录下的proxy.conf文件中定义。继续前面的练习，我们将创建我们的第一个领域。