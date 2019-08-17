# 行动时刻 - 定义领域

以下步骤演示了如何定义领域：

1. 将以下域添加到位于FreeRADIUS配置目录下的proxy.conf文件中：
```
realm my-org.com {
}
```
2. 以调试模式重新启动FreeRADIUS服务器，并以alice@my-org.com身份进行验证。 观察FreeRADIUS服务器的输出。 以下应该是输出的一部分：
```
[suffix] Looking up realm "my-org.com" for User-Name = "alice@myorg.com"
[suffix] Found realm "my-org.com"
[suffix] Adding Stripped-User-Name = "alice"
[suffix] Adding Realm = "my-org.com"
[suffix] Authentication realm is LOCAL.
++[suffix] returns ok
```
3.编辑my-org.com域以包含nostrip指令：
```
realm my-org.com {
	NOSTRIP
}
```
4.以调试模式重新启动FreeRADIUS服务器，并以alice@my-org.com身份进行身份验证。 观察FreeRADIUS服务器的输出。 身份验证应该失败，以下内容应该是输出的一部分：
```
[suffix] Looking up realm "my-org.com" for User-Name = "alice@myorg.com"
[suffix] Found realm "my-org.com"
[suffix] Adding Realm = "my-org.com"
[suffix] Authentication realm is LOCAL.
++[suffix] returns ok
```
## 刚刚发生了什么？
我们定义了一个真实领域，并研究了在领域定义中使用nostrip指令时的结果。
领域模块（后缀是实例）将在proxy.conf文件中查找领域。 如果找到并且定义中没有nostrip选项，它将添加Stripped-User-Name和Realm属性。 但是，如果领域定义中有一个nostrip选项，它只会添加Realm属性。
与身份验证相关的模块（如文件模块）检查用户是否存在Stripped-User-Name属性。 如果找到一个，它们将使用该值而不是User-Name属性的值来查找有效用户。
当我们使用nostrip选项时，没有添加Stripped-User-Name属性，User-Name是alice@my-org.com。 这就是身份验证失败的原因。

## 拒绝没有领域的用户名
两个组织之间存在漫游时的典型要求是阻止用户在没有域名的情况下使用其用户名。 如果不这样做，可能会导致用户名alice在my-org.com上运行，但不能在your-org.com上运行。 强制用户名格式为alice@my-org.com将确保其在两个组织中都有效。 下一个练习将向您展示如何执行此操作。











