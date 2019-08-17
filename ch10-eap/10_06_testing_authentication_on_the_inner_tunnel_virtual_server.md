# 操作时间 - 在内部隧道虚拟服务器上测试身份验证
默认情况下，内部隧道虚拟服务器具有侦听部分，该部分侦听IP地址127.0.0.1和端口18120以进行身份验证请求。 这可用于测试虚拟服务器对身份验证请求的响应方式。

1. 确认已启用内部隧道虚拟服务器（列在启用站点的目录下），并且它包含以下监听部分。 默认情况下应包括此项。
listen {
	ipaddr = 127.0.0.1
	port = 18120
	type = auth
}
2. 在调试模式下重新启动FreeRADIUS。
3. 使用以下命令在内部隧道虚拟服务器上测试身份验证：
`radtest alice passme 127.0.0.1:18120 100 testing123`
4. 您应该通过查看调试输出中的反馈来查看内部隧道虚拟服务器：
```
server inner-tunnel {
+- entering group authorize {...}
++[chap] returns noop
```

## 刚刚发生了什么？
我们已经使用内部隧道虚拟服务器中定义的listen部分来测试虚拟服务器如何响应身份验证请求。

## 内在和外在身份之间的区别
像EAP-TTLS和PEAP这样的隧道EAP方法包含两个身份，如下所示：
+ 一个被称为外部身份，对外界是可见的。这意味着当RADIUS数据包被嗅探到时，您可以看到此值。
+ 另一个称为内部身份，在检测身份验证者和RADIUS服务器之间流动的RADIUS数据包时无法跟踪。
这些标识被指定为用户名AVP的值。外部身份对于做出代理决策很重要，这意味着用户名AVP的领域部分必须是正确的，以便请求到达正确的目的地。
FreeRADIUS中提供的虚拟服务器功能允许我们通过两个不同的虚拟服务器处理外部和内部身份。默认情况下，外部标识将由默认虚拟服务器处理，内部标识由内部隧道虚拟服务器处理。这使我们能够为内部和外部身份创建和管理两个完全独立的策略。

## 试一试 - 使用JRadius模拟器测试两个身份
在本练习中，我们将使用JRadius Simulator来使用两个不同的身份，以便在内部身份和外部身份之间进行区分：
1. JRadius Simulator在“属性”选项卡上有一列，允许您指定隧道的属性（TunnelReq）。 更新属性以显示以下内容：

![Update_the_Atributes](https://github.com/lsqms/FreeRADIUS/blob/master/image/ch10/Update_the_Atributes%20.PNG?raw=true)

2. 选择RADIUS选项卡，选择EAP-TTLS / PAP作为身份验证协议。 然后单击“开始”以执行身份验证测试。
3. 观察FreeRADIUS的调试输出，并在反馈中查看以下内容：
```
Sending Access-Accept of id 197 to 192.168.1.101 port 47083
MS-MPPE-Recv-Key = 0x825059e7df2f0dc.....
MS-MPPE-Send-Key = 0x89258e9f9267997ec59a1d5a5.....
EAP-Message = 0x03070004
Message-Authenticator = 0x00000000000......
User-Name = "bob@fake-realm.com"
```

RFC标准规定，在发送到RADIUS服务器的计费数据包中，NAS应使用Access-Accept数据包中指定的用户名AVP的值。
FreeRADIUS的默认值是将外部标识作为User-Name AVP的值返回。 当我们想要执行准确的记账时，这给我们留下了潜在的问题，因为用户可以指定任何东西作为我们刚才看到的外部标识的值。

我们可以使用unlang来纠正这个：
1. 编辑位于FreeRADIUS配置目录下的sites-enabled / inner-tunnel文件，并在post-auth部分的顶部添加以下部分：
```
if(outer.request:User-Name != "%{request:User-Name}"){
	update reply {
		User-Name := "%{request:User-Name}"
	}
}
```
2. 编辑位于FreeRADIUS配置目录下的eap.conf文件，并在ttls和peap方法下更改以下指令：
`use_tunneled_reply = no`更改为`use_tunneled_reply = yes`

3. 在调试模式下重新启动FreeRADIUS，并使用JRadius Simulator再次测试EAP-TTLS / PAP身份验证。调试反馈现在应该返回alice而不是bob@fake-realm.com作为用户名AVP的返回值：
```
Sending Access-Accept of id 239 to 192.168.1.101 port 55543
User-Name = "alice"
...
```

## 刚刚发生了什么？
我们通过在FreeRADIUS的Access-Accept回复中返回内部身份而不是外部身份作为用户名AVP的值来设法改善记账数据的真实性。
Unlang使我们有机会从用于内部请求的虚拟服务器获取外部请求中的属性。我们使用外部前缀以获取外部请求的属性列表。然后，我们可以将外部请求中的属性值与内部请求的属性值进行比较。
如果我们希望内部隧道虚拟服务器的回复属性直接到默认虚拟服务器，我们必须指示eap方法这样做。我们通过将use_tunneled_reply指令设置为yes来完成此操作。
请务必检查接收具有内部标识的Access-Accept的NAS是否支持用户名回复AVP。有关于RFC的错误实现的报告。此问题的解决方法可能是将内部标识与外部标识进行比较，如果请求不相同则拒绝该请求。但是，这需要额外的用户教育。

## 外部标识的命名约定
一个好的做法是教育用户提供他们的电子邮件地址作为外部身份。但是，这可能是潜在的安全风险。当您的组织成为像Eduroam这样的RADIUS服务器的分层网络的一部分时，这些请求将通过可能滥用外部身份的第三方服务器。然后，更通用的身份可能是更好的。因此，您宁愿使用eduroam@frbg.com而不是alice@frbg.com。没有固定的规则，但请注意。
如果您是RADIUS服务器的分层网络的一部分，则只有对EAP-TTLS和PEAP的RADIUS代理请求可以完全隐藏来自执行代理的第三方RADIUS服务器的用户身份和密码。
如果您的FreeRADIUS服务器响应代理请求并且您对安全性非常偏执，请考虑将返回到第三方RADIUS服务器的Access-Accept消息中的User-Name AVP的值更改为通用。
要继续这个偏执的注释，下一节将向您展示如何禁用未使用的EAP方法。