
# 行动时间 - 使用FreeRADIUS对用户进行身份验证

我们继续前一章的练习，我们在用户文件中对用户进行了认证。我们现在将查看在调试模式下运行的FreeRADIUS服务器的输出，而不是查看radtest命令的反馈。

1. 确保您是root用户，以便编辑用户文件。

2. 将Alice定义为FreeRADIUS测试用户。在用户文件的顶部添加以下行。确保第二行由单个制表符缩进。
```
“alice”Cleartext-Password：=“passme”
	Reply-Message =“你好，％{用户名}”
```

3. 以调试模式启动FreeRADIUS服务器。通过启动脚本将其关闭，确保尚未运行实例。我们假设在这种情况下使用Ubuntu。
```
$> sudo su
＃> /etc/init.d/freeradius停止
＃> freeradius -X
```

4. 使用以下命令验证Alice：
```
$> radtest alice passme 127.0.0.1 100 testing123
```

5. FreeRADIUS的调试输出将显示Access-Request数据包如何到达以及FreeRADIUS服务器如何响应此请求。

## 刚刚发生了什么？

您已向FreeRADIUS发送了一个Access-Request数据包。 您已收到Access-Accept数据包。 没什么好激动的，但是观察FreeRADIUS的调试输出是非常有趣的。 这显示了返回Access-Accept数据包所涉及的内容。

>如果你想使用FreeRADIUS在调试模式下运行的终端，在保持活动状态的同时你可以使用Ctrl + Z暂停当前作业（radiusd -X），然后执行bg命令在后台运行它。 终端现在应该可供您使用。 要在前台再次运行FreeRADIUS，只需执行fg命令即可。

## 访问请求到达
当数据包到达FreeRADIUS服务器时，由以下部分指示：
```
rad_recv: Access-Request packet from host 127.0.0.1 port 48698, id=73,length=57
	User-Name = "alice"
	User-Password = "passme"
	NAS-IP-Address = 127.0.1.1
	NAS-Port = 100
```
我们看到传入的请求包含四个AVP。
虽然此处以明文形式显示AVP用户密码，但它未以明文形式传送到服务器。 FreeRADIUS使用共享密钥来加密和解密User-Password AVP的值。

## 授权
收到请求后，授权部分负责处理请求：
```
# Executing section authorize from file /etc/freeradius/sites-enabled/
default
+- entering group authorize {...}
++[preprocess] returns ok
++[chap] returns noop
++[mschap] returns noop
++[digest] returns noop
[suffix] No '@' in User-Name = "alice", looking up realm NULL
[suffix] No such realm "NULL"
++[suffix] returns noop
[eap] No EAP-Message, not doing EAP
++[eap] returns noop
[files] users: Matched entry alice at line 137
[files] expand: Hello, %{User-Name} -> Hello, alice
++[files] returns ok
++[expiration] returns noop
++[logintime] returns noop
++[pap] returns updated
Found Auth-Type = PAP
```

授权部分在虚拟服务器中定义。让我们首先看一下有关FreeRADIUS中虚拟服务器的一些要点：

+ 虚拟服务器在sites-available目录下定义，该目录位于FreeRADIUS的配置目录下。
+ 每个虚拟服务器由单个文本文件表示。
+ 通过创建从sites-available目录中的文件到具有相同名称的sites-enabled目录中的文件的软链接来激活虚拟服务器。
+ 此方法类似于Apache Web服务器使用的方法。
+ 名为default的虚拟服务器处理所有典型请求。
+ 虚拟服务器基本上就像拥有多个RADIUS服务器。一个虚拟服务器甚至可以将请求转发到另一个虚拟服务器。这使得FreeRADIUS安装功能非常强大。
+ 每个虚拟服务器，包括默认服务器，具有各种部分。虚拟服务器可以包含嵌套在虚拟服务器定义内的以下部分：监听，客户端，授权，身份验证，授权后，预代理，后代理，预备，计费和会话。
+ 访问请求首先由授权部分处理。

## 授权设置Auth-Type
当授权部分处理请求时，各种FreeRADIUS模块会查看Access-Request中包含的AVPs。 这些模块尝试确定用于验证用户的机制和模块。 在我们的示例中，授权部分将Auth-Type设置为PAP。
例如，如果访问请求包含MS-CHAP属性而不是用户密码，则mschap模块将检测到此并设置Auth-Type = MS-CHAP。
## 授权生效
授权部分可以基于对指定AVP的存在或值的决定来决定完全拒绝请求。 这将导致返回到客户端的Access-Reject数据包。 那时就没有必要进行身份验证。

## 认证
在设置Auth-Type的值后，请求将传递给authenticate部分：
```
# Executing group from file /etc/freeradius/sites-enabled/default
+- entering group PAP {...}
[pap] login attempt with password "passme"
[pap] Using clear text password "passme"
[pap] User authenticated successfully
++[pap] returns ok
```
在这里，我们看到authenticate部分中的pap子部分正在处理此请求并返回ok。
## 后验证
认证后部分是在认证之后完成的。您可以使用它来执行某些操作：
```
# Executing section post-auth from file /etc/freeradius/sites-enabled/
default
+- entering group post-auth {...}
++[exec] returns noop
```
## 结束
结果现在发送回客户端：
```
Sending Access-Accept of id 73 to 127.0.0.1 port 48698
Reply-Message = "Hello, alice"
Finished request 3.
```
## 结论
查看调试输出时请记住以下几点：

+ 授权，验证和后验证等主要部分以#Execution开头。
+ 这些部分还指示它们驻留在哪个虚拟服务器中。
+ 授权部分设置Auth-Type的值。这又决定了将使用身份验证部分中的哪个模块。
+ FreeRADIUS模块的调试输出可分为两种类型。它们是调试消息和返回值。
+ 调试消息以模块名称开头，例如
[files] users: Matched entry alice at line 137.
+ 返回值以++ [module_name]开头，例如
 ++[files] returns ok.

## 尝试看看 - 使用其他身份验证协议
从FreeRADIUS 2.1.10版开始，radtest客户端程序允许您指定要使用的身份验证协议。

如果您的FreeRADIUS安装比2.1.10更新，您可以使用-t选项指定chap和mschap并再次执行身份验证请求。请注意当使用其他身份验证协议时，FreeRADIUS的调试反馈现在是多么不同。

下一部分将介绍我们可以存储用户密码的不同格式。
