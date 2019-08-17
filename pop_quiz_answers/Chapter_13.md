# 故障排除
1. 在Debian和Ubuntu上，当您安装标准的FreeRADIUS软件包时，FreeRADIUS服务器二进制文件称为freeradius而不是radiusd。
2. 您可以创建将使用较慢服务器的ldap模块的命名实例。 然后，您可以使用冗余部分替换authorize部分中的ldap条目，该冗余部分首先使用快速LDAP服务器列出模块，然后使用较慢的部分使用ldap模块实例。
```
#ldap
redundant {
	ldap
	ldap.slow1
	ldap.slow2
}
```
如果对LDAP使用“bind as”身份验证方法，则还需要将authenticate部分中的Auth-Type LDAP更改为以下内容：
```
Auth-Type LDAP {
	redundant {
		ldap
		ldap.slow1
		ldap.slow2
	}
}
```
3. Bob的机器上的请求者可能设计得很糟糕。 当他的密码在后端被更改时，他的请求者继续尝试通过发送以前的密码进行连接。 后端检测到潜在的入侵并锁定了帐户。 如果请求者写得很好，就会弹出一个对话框供Bob提供他的凭据。 如果这不是问题所在，则可能是Bob连接的接入点将从先前成功的会话存储的凭证转发到RADIUS服务器。
4. 如果人们没有遵循RFC 1855中规定的正确网络礼节，或者他们不遵守此URL中指定的邮件列表规则，就会发生这种情况：
http://freeradius.org/list/users.html