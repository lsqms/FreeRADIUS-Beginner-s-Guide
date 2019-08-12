# 行动时刻 - 禁用未使用的EAP方法
我们的组织决定支持两种隧道式EAP方法（PEAP和EAP-TTLS）。 我们将禁用其他方法并将默认EAP方法设置为PEAP：
1. 编辑位于FreeRADIUS配置目录下的eap.conf文件。 通过完全注释掉它们来禁用以下方法：md5，leap，gtc和mschapv2。
2. 更改default_eap_type指令：
d`efault_eap_type = md5` 
更改为：
`default_eap_type = peap`
3. 在调试模式下重新启动FreeRADIUS，并检查已禁用的EAP方法是否不再可用。 以下是我们尝试EAP-MD5时FreeRADIUS的调试输出。 它确认不再支持EAP Type 4（MD5）：
```
+- entering group authenticate {...}
[eap] Request found, released from the list
[eap] EAP NAK
[eap] NAK asked for unsupported type 4
[eap] No common EAP types found.
[eap] Failed in EAP select
++[eap] returns invalid
Failed to authenticate the user.
```
## 刚刚发生了什么？
我们更改了FreeRADIUS在识别后要求的默认EAP方法。我们还禁用了未使用的EAP方法。在eap.conf文件中，我们没有注释掉tls方法，因为ttls和peap方法都使用它来创建安全隧道。
禁用不支持的EAP方法并将默认EAP方法设置为最常用的EAP方法将减少身份验证器与RADIUS服务器之间的网络流量。这反过来应该导致更好的表现。也可以通过禁用它们来停止使用安全性较低的EAP方法。
### 消息身份验证
作为安全性的最后一点，我们将考虑对来自NAS的每个请求强制执行消息验证器AVP。 FreeRADIUS配置目录下的clients.conf文件中的每个客户端定义都有一个require_message_authenticator指令。如果将其设置为yes，它将拒绝来自指定设备的不包含Message-Authenticator AVP的请求，或者Message-Authenticator的值不正确的请求。
Message-Authenticator的值是通过在RADIUS数据包上生成HMAC-MD5校验和来创建的。此属性旨在阻止攻击者设置“流氓”NAS并对RADIUS服务器执行在线词典攻击的尝试。当攻击者截获包含（例如）chap挑战和响应的数据包，并对这些数据包执行字典攻击时，它不会对“离线”攻击提供有效保护。当服务器收到请求时，它会计算Message-Authenticator的假定值的值，并将其与收到的值进行比较。如果值不匹配，则会以静默方式丢弃该请求。这仅在身份验证请求时完成。消息验证器AVP不包含在计费数据包中。
遗憾的是，如果将NAS详细信息存储在SQL数据库而不是clients.conf文件中，则SQL表（nas）不提供设置此指令。

这使我们结束了对FreeRADIUS中EAP的讨论。现在是时候看一些要记住的关键点了。















