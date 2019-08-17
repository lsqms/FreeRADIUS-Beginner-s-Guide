# 行动时刻 - 使用首选的状态检查方式
以下步骤将演示如何执行状态检查：
1. 更新my-org.com和your-org.com的home_server定义以包含以下指令：`status_check = status-server` 
2. 要了解FreeRADIUS如何将 Status-Server 数据包发送到挂掉的服务器，只需关闭your-org.com的FreeRADIUS服务器，然后继续向my-org.com服务器发送bob@your-org.com的身份验证请求：
```
Marking home server 192.168.1.106 port 1812 as zombie (it looks
like it is dead).
Sending Status-Server of id 97 to 192.168.1.106 port 1812
Message-Authenticator := 0x00...
NAS-Identifier := "Status Check. Are you alive?"
```
3. 在调试模式下再次为your-org.com启动FreeRADIUS服务器，并查看它如何回答从my-org.com服务器发送给它的Status-Server数据包：
```
rad_recv: Status-Server packet from host 192.168.1.105 port 1814,
id=160, length=68
Message-Authenticator = 0x7b2f0a58666d532b2...
NAS-Identifier = "Status Check. Are you alive?"
Sending Access-Accept of id 160 to 192.168.1.105 port 1814
```
4. 在对Status-Server请求做出指定数量的响应后，my-org.com上的FreeRADIUS服务器会将your-org.com的主服务器标记为再次活动：
```
Received response to status check 16 (3 in current sequence)
Marking home server 192.168.1.106 port 1812 alive
```

这里我们结束了状态服务器的讨论。 这只是获得更多背景的一般介绍。 建议您阅读proxy.conf文件中的信息，以帮助您配置和微调用于故障转移或负载平衡的设置。
最后，请记住，在代理期间，home_server_pool会使用此故障转移和负载平衡。 FreeRADIUS本身还通过使用unlang提供故障转移和负载平衡功能。 关键字冗余，负载平衡和冗余负载平衡用于在FreeRADIUS内的不同模块之间创建故障转移和负载平衡配置。

## 代理计费要求
要了解当my-org.com上的FreeRADIUS服务器收到bob@your-org.com的计费请求时会发生什么，我们可以使用第6章“计费”中的两个文件来模拟计费：

+ 4088_06_acct_start.txt：此文件可用于模拟会话的开始。
+ 4088_06_acct_stop.txt：此文件可用于模拟会话结束。

修改这些文件并将User-Name ='alice'更改为User-Name ='bob@your-org.com'。






























