# 行动时刻 - 在两个组织之间配置代理
我们将从my-org.com的FreeRADIUS服务器开始：

1. 编辑位于FreeRADIUS配置目录下的用户文件，并确保以下alice条目存在：
```
"alice" Cleartext-Password := "passme"
Tunnel-Type = VLAN,
Tunnel-Medium-Type = IEEE-802,
Tunnel-Private-Group-ID = "100"
```
2. 编辑位于FreeRADIUS配置目录下的proxy.conf文件夹，并为your-org.com添加一个主服务器条目。 我们假设它的IP地址为192.168.1.106。
```
home_server hs_1_your-org.com {
	type = auth+acct
	ipaddr = 192.168.1.106
	port = 1812
	secret = testing123
}
```
3. 同时将一个home_server_pool部分添加到proxy.conf文件中，该文件包含上一步中定义的home_server：
```
home_server_pool pool_your-org.com {
	type = fail-over
	home_server = hs_1_your-org.com
}
```
4. 使用此池代理对your-org.com域的请求：
```
realm your-org.com {
	pool = pool_your-org.com
	NOSTRIP
}
```
5. 为my-org.com创建一个LOCAL域：
```
realm my-org.com {
}
```
6. 编辑位于FreeRADIUS配置目录下的clients.conf文件，以允许来自your-org.com RADIUS服务器的请求。 我们假设它的IP地址为192.168.1.106：
```
client your-org.com {
	ipaddr = 192.168.1.106
	secret = testing123
}
```
这样就完成了my-org.com RADIUS服务器所需的配置。
my-org.com RADIUS服务器现在将做两件事：

+ 将your-org.com域的请求转发到your-org.com RADIUS服务器
+ 接受来自your-org.com RADIUS服务器的请求

我们现在将以类似的方式配置your-org.com RADIUS服务器：
1. 编辑位于FreeRADIUS配置目录下的用户文件，并确保存在以下bob条目：
```
"bob" Cleartext-Password := "passbob"
Tunnel-Type = VLAN,
Tunnel-Medium-Type = IEEE-802,
Tunnel-Private-Group-ID = "55"
```
2. 编辑位于FreeRADIUS配置目录下的proxy.conf文件，并为my-org.com添加主服务器条目。 我们假设它的IP地址为192.168.1.105。
```
home_server hs_1_my-org.com {
	type = auth+acct
	ipaddr = 192.168.1.105
	port = 1812
	secret = testing123
}
```
3. 将一个home_server_pool部分添加到proxy.conf文件中，该文件包含上一步中定义的home_server：
```
home_server_pool pool_my-org.com {
	type = fail-over
	home_server = hs_1_my-org.com
}
```
4. 使用此池代理my-org.com域的请求：
```
realm my-org.com {
	pool = pool_my-org.com
	NOSTRIP
}
```
5. 为your-org.com创建一个LOCAL领域：
```
realm your-org.com {
}
```
6. 编辑位于FreeRADIUS配置目录下的clients.conf文件，以允许来自my-org.com RADIUS服务器的请求。 我们假设它的IP地址为192.168.1.105：
```
client my-org.com {
	ipaddr = 192.168.1.105
	secret = testing123
}
```

your-org.com RADIUS服务器现在将做两件事：
+ 将my-org.com域的请求转发到my-org.com RADIUS服务器
+ 接受来自my-org.com RADIUS服务器的请求
场景已经确定。 在调试模式下重新启动两个FreeRADIUS服务器并执行以下测试，同时仔细观察两个FreeRADIUS服务器上的调试输出。 我们将使用一个表来列出要执行的测试，要执行它的服务器以及将要测试的内容。
|User to authentcate|Do on RADIUS server|What is tested|
|:-|:-:|-:|
|alice|my-org.com|Local user without realm|
|alice@my-org.com|my-org.com|Local user with realm|
|bob@your-org.com|my-org.com|Remote user with realm|
|bob|your-org.com|Local user without realm|
|bob@your-org.com|your-org.com|Local user with realm|
|alice@my-org.com|your-org.com|Remote user with realm|

如果两个服务器都配置正确，则所有Access-Requests都应通过。 如果它不能按预期工作，请重新检查配置并按照调试输出尝试确定请求被拒绝的位置。

## 刚刚发生了什么？
我们配置了两台RADIUS服务器，每台服务器都具有以下功能：
+ 每个服务器都有自己的领域。
+ 每个服务器将为其他服务器上定义的用户转发传入请求（身份验证和记帐）到该服务器。

下面的部分将专门介绍代理身份验证请求时的要点。 此后的部分将讨论代理计费请求的重点。

## 代理身份验证请求
为了给我们的服务器提供另一个RADIUS服务器代理权限，我们只需将它作为我们服务器的一个客户端添加到clients.conf文件中。 正如我们所见，用户通过定义领域而组合在一起。 本节中的新部分是home_server和home_server_pool的定义。

+ home_server和home_server_pool用于定义各个域可以向其发送代理请求的外部RADIUS服务器。
+ 领域包含home_sever_pool。
+ home_server_pool又包含一个或多个home_server条目。

以下示意图显示了如何将域(realm)，home_server_pool和home_server部分用作创建许多布置的单独构建块。


## home_server
home_server部分定义了一个服务器，用于代理某些类型的请求。在我们的示例设置中，我们指定了auth + acct。本质上，它包含FreeRADIUS将用作指定家庭服务器的客户端的详细信息。它还可以包含可选指令，当home_server在home_server_pool中分组时，FreeRADIUS将使用这些指令来确定故障转移和负载平衡。一个home_server可以包含在一个或多个home_server_pool中，或者根本不包括在内。

## home_server_pool
home_server_pool用于将一个或多个home_servers组合在一起。 home_server_pool中home_server的选择标准可以在故障转移模式（默认）或负载平衡模式下完成。确保只将相同类型的home_servers（例如 auth + acct）添加到池中，以确保它们都能够处理转发给它们的请求。
拥有这三个构建块可以以最小的努力为不同的安排提供极大的灵活性和可能性。FreeRADIUS配置目录下的proxy.conf文件中存在大量详细信息，包括示例配置，可帮助您创建备用配置。

proxy.conf文件中的注释还提到了使用authost，accthost和secret指令而不是home_server_pool指令为特定域定义主服务器的另一种方法。 这些指令是定义家庭服务器的旧方法的一部分，鼓励人们使用更新的方式来提高灵活性。

## 身份验证代理请求的流程图
下图显示了代理到另一台服务器的请求与本地处理的请求之间的流量差异。

如果我们在bob@your-org.com尝试验证时查看my-org.com RADIUS服务器上的调试输出，我们可以按照此流程进行操作。 我们来讨论一些要点。

### 后缀设置控制：Proxy-To-Realm
在授权部分，后缀模块确认bob@your-org.com属于your-org.com域并将控制：Proxy-To-Realm设置为your-org.com：
```
[suffix] Looking up realm "your-org.com" for User-Name = bob@your-org.com
[suffix] Found realm "your-org.com"
[suffix] Adding Realm = "your-org.com"
[suffix] Proxying request from user bob to realm your-org.com
[suffix] Preparing to proxy authentication request to realm "your-org.
com"
++[suffix] returns updated
```
您将看到输出仅表明它正在准备代理身份验证请求。 这是因为如果授权部分中的其他模块更改了control:Proxy-To-Realm的值，则仍可以更改或取消代理决策。

预代理(pre-proxy)部分
由于control：Proxy-To-Realm设置为your-org.com，请求没有流向authenticate部分，而是转到预代理( pre-proxy )部分。 但是，此部分为空，这就是调试消息中以下行的原因：
`WARNING: Empty section. Using default return values.`
预代理部分可以用作取消或更改代理请求的最后位置，方法是修改control：Proxy-To-Realm AVP值。 然后我们看到请求如何发送到your-org.com的主服务器。
代理后(Post-proxy)部分
当您从your-org.com的主服务器返回回复时，我们会看到此回复如何通过后代理部分传递。 此部分通常用于删除your-org.com的主服务器返回的AVP。 此后，请求通过post-auth部分，并将回复返回给原始客户端。
```
rad_recv: Access-Accept packet from host 192.168.1.106 port 1812, id=184,
length=40
Tunnel-Type:0 = VLAN
Tunnel-Medium-Type:0 = IEEE-802
Tunnel-Private-Group-Id:0 = "55"
Proxy-State = 0x3330
+- entering group post-proxy {…}
[eap] No pre-existing handler found
++[eap] returns noop
Found Auth-Type = Accept
Auth-Type = Accept, accepting the user
+- entering group post-auth {…}
++[exec] returns noop
Sending Access-Accept of id 30 to 127.0.0.1 port 57020
Tunnel-Type:0 = VLAN
Tunnel-Medium-Type:0 = IEEE-802
Tunnel-Private-Group-Id:0 = "55"
```

## EAP和动态VLAN
在上一节中，我们看到了从your-org.com主服务器返回到my-org.com RADIUS服务器客户端的三个属性。这些属性用于动态VLAN分配。动态VLAN分配在某些企业网络中完成，这些网络在LAN上使用802.1x或在其Wi-Fi网络上使用WPA-2。这有助于动态地将每个客户端放在指定的VLAN中。用户应该属于哪个VLAN的决定可以基于许多事情，例如特权（例如学生和教授）或设备类型（例如VOIP电话）。
注意Tunnel-Private-Group-Id的值。此属性的值是字符串而不是整数。VLAN除了数字之外还可以有名称。某些设备只接受VLAN名称，而其他设备需要VLAN编号。在分配值之前，请务必检查设备需要的设备。
your-org.com使用的VLAN编号在my-org.com上不一定具有相同的权限，甚至不一定由my-org.com实现。这就是为什么在下一节中我们将修改从your-org.com上的主服务器返回的属性，以便它们满足my-org.com网络上的要求。在修改属性之前，作为可选练习，您可以使用第10章EAP中讨论的JRadius Simulator程序来测试EAP请求的代理。

## 试一试 - 测试代理EAP身份验证
通过执行前面表中列出的同一组测试来测试my-org.com和your-org.com之间的EAP身份验证代理。您会注意到运行隧道EAP方法时将丢失回复AVP。要在这些EAP方法中返回回复AVP，请确保在位于FreeRADIUS配置目录下的eap.conf文件中的peap和ttls部分中更改以下指令：use_tunneled_reply = no 更改为： use_tunneled_reply = yes。
隧道EAP方法的代理从不将位于隧道内的用户详细信息和密码暴露给转发请求的RADIUS服务器。这比PAP等其他身份验证协议更安全。
## 删除和替换回复属性
由于您无法控制从外部主(home)服务器返回的AVP，因此最好在将这些属性及其值返回到我们的服务器后对其进行管理。在本练习中，我们将使用my-org.com上的my-org.com上的动态VLAN详细信息替换your-org.com返回的动态VLAN详细信息。





















