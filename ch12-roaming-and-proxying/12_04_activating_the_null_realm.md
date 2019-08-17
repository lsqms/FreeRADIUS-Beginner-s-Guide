# 行动时刻 - 激活NULL领域
请按照以下步骤激活NULL域：
1. 编辑FreeRADIUS配置目录下的proxy.conf文件，并更改以下部分：
```
#realm NULL {
# authhost = radius.company.com:1600
# accthost = radius.company.com:1601
# secret = testing123
#}
```
更改为：
```
realm NULL {
# authhost = radius.company.com:1600
# accthost = radius.company.com:1601
# secret = testing123
}
```
2. 在调试模式下重新启动FreeRADIUS服务器并作为alice进行身份验证。 观察FreeRADIUS服务器的输出。 以下应该是输出的一部分：
```
[suffix] No '@' in User-Name = "alice", looking up realm NULL
[suffix] Found realm "NULL"
[suffix] Adding Stripped-User-Name = "alice"
[suffix] Adding Realm = "NULL"
[suffix] Authentication realm is LOCAL.
++[suffix] returns ok
```

## 刚刚发生了什么？
我们创建了特殊域NULL并观察了身份验证请求的调试输出的变化。

## 剥离用户名和领域
后缀模块在proxy.conf文件中找到了NULL域。如果找到一个领域，则根据User-Name的值添加两个新属性，该属性分为两个组件：

+ Stripped-User-Name：这只是没有@realm的用户名，例如alice
+ 领域(Realm)：在alice的情况下没有领域，后缀模块使用特殊领域NULL

后缀模块还决定如何处理请求并报告以下方式：
`[suffix] Authentication realm is LOCAL.`
后缀模块自动设置领域;但是，我们可以在授权阶段的任何时候决定通过简单地使用unlang并在控件属性列表中修改Proxy-To-Realm属性的值来更改它。
在这里，我们测试用户名AVP的值，当它是必需值时，它将在本地进行身份验证：
```
if(request:User-Name == 'my-org-test@your-org.com'){
	update control {
		Proxy-To-Realm := LOCAL
	}
}
```

这是基于某些属性取消或更改代理请求的便捷方法。

## 本地(LOCAL)领域
如前所述的LOCAL领域也是特殊领域之一。当我们定义NULL域时，我们没有指定authost，accthost或secret。如果未指定这些，则将使用特殊领域LOCAL。 LOCAL领域只是一种说“不代理，继续，谢谢”的方式。在proxy.conf文件中定义了一个领域LOCAL，但它更像是一个占位符，它永远不会被修改。即使从proxy.conf文件中删除它，LOCAL域仍然可用于后缀模块。
## 领域的动作
定义领域时，您可以指定应采取的操作。这是通过在领域定义中使用指令来确定的。有三种类型的操作：

+ 将此请求代理到在Internet上的另一个RADIUS服务器或服务器池。这将在我们将使用pool指令的章节中稍后介绍。
+ 使用virtual_server指令将此请求转发到本地虚拟服务器。这类似于将其转发到另一台RADIUS服务器。该请求也将通过预代理和代理后部分发送，但不是转到外部服务器而是转发到本地虚拟服务器。
+ 不代理此请求，请使用本地服务器。当领域定义不包含指定外部或虚拟服务器的任何指令时，将使用特殊领域LOCAL。我们用NULL领域做到了这一点。


































