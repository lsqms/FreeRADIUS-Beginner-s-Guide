# EAP
1. 忽略这些链接；为了理智起见，您甚至可以删除它们。EAP在新安装中工作正常。EAP配置的更改越少越好。
2. 在验证用户身份时，EAP-TTLS / PAP方法使用内部隧道虚拟服务器而不是默认虚拟服务器。确保还指定在内部隧道虚拟服务器中使用ldap模块。这些虚拟服务器彼此独立。
3. 不，当您以用户身份绑定时，您需要将用户的明文密码发送到LDAP服务器。当您使用PEAP / MSCHAPv2时，无法从事务中获取明文密码。
4. 这里没有谎言！通用密码功能允许ldap模块从LDAP服务器以明文形式获取密码。要获取此密码，需要遵循一些规则。与LDAP服务器的连接必须是一个安全连接，特殊的特权用户绑定它以运行查询。还必须在ldap配置中指定password_attribute。有关更多详细信息，请参阅ldap配置文件。
5. 使用iPhone Configuration Utlity创建.mobileconfig文件。从Web服务器分发此文件。