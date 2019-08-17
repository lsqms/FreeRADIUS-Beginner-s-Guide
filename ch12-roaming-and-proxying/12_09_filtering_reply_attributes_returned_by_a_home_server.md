# 行动时刻 - 过滤主服务器返回的回复属性
必须在my-org.com FreeRADIUS服务器上执行以下操作：
1. 编辑位于FreeRADIUS配置目录下的sites-enabled / default文件，并取消注释post-proxy部分下的attr_filter.post-proxy行：
```
# Uncomment the following line if you want to filter
# replies from remote proxies based on the rules defined
# in the 'attrs' file.
attr_filter.post-proxy
```
2. 编辑FreeRADIUS配置目录下的attrs文件，并在DEFAULT条目之前添加以下条目：
```
your-org.com
	Reply-Message =* ANY,
	Tunnel-Type := VLAN,
	Tunnel-Medium-Type := IEEE-802,
	Tunnel-Private-Group-Id := "100"
```
3. 在调试模式下重新启动FreeRADIUS服务器，并从my-org.com上的FreeRADIUS服务器测试bob@your-org.com的身份验证。
4. 现在，回复属性将始终包含以下内容，不论your-org.com的主服务器返回的AVP：
```
Tunnel-Type:0 = VLAN
Tunnel-Medium-Type:0 = IEEE-802
Tunnel-Private-Group-Id:0 = "100"
```
## 刚刚发生了什么？
我们已经为your-org.com的主服务器实现了回复属性的过滤器。为此，我们使用了rlm_attr_filter模块。该模块本身有大量文档，包括手册页（man rlm_attr_filter）和示例attrs文件。该模块的各种实例在位于FreeRADIUS配置目录下的modules / attr_filter文件中定义。
属性条目的格式为<attribute> <operator> <value>。选择运算符时，请查阅手册页。选择正确的运算符对于过滤器按预期工作至关重要。我们选择了：=运算符，它将覆盖现有属性（如果存在）或添加属性（如果它不存在）。假设我们使用了==运算符而不是：=，那么当来自主服务器的回复包含具有指定值的特定属性时，它将仅返回该特定属性。我们还使用了= * ANY模式作为Reply-Message AVP。这意味着Reply-Message的任何值都应该被转发。
在我们继续讨论计费请求的代理之前，我们将简要介绍一下home_server_pool中的故障转移和负载均衡配置。

## 主(home)服务器的状态
通过创建home_server_pool，我们可以在此池中指定一些主服务器。我们可以声明两种类型的池。一个是处理高负荷;另一个将处理网络中断。对于这两者，FreeRADIUS需要跟踪池内主服务器的运行状况。要指定FreeRADIUS如何检查家庭服务器的运行状况，我们在home_server声明中使用status_check指令。根据status_check的值，还有其他附加指令将影响（或微调）健康检查的执行方式。
status_check的三个可能值是：

+ None：虽然这是默认值，但它是最不受欢迎的。只能将它作为最后的手段。
+ Status-Server：这要求主服务器支持接收Status-Server数据包。在指定之前确认主服务器是否支持它。这是状态检查的首选方式。
+ Request：FreeRADIUS将向主服务器发送Access-Request或Authentication-Request数据包以检查其状态。如果主服务器不支持Status-Server数据包，请使用此选项。
虽然我们只为my-org.com和your-org.com指定了一个主服务器，但我们可以添加status_check = status-server指令来指定检查主服务器健康状况的首选方法。








































