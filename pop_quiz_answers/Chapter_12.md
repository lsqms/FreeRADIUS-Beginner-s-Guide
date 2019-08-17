# 漫游和代理
1. 是的，在RADIUS服务器之间配置漫游不依赖于某些RADIUS服务器软件。 如果服务器软件符合RFC中的标准，它应该很容易。
2. 不，您可以通过your-org.com通知访问者他应该能够使用org.com SSID的配置文件进行简单连接，而无需进行任何更改。 对your-org.com的EAP请求将仅代理到your-org.com上的RADIUS服务器。
3. 动态VLAN分配很可能通过RADIUS服务器完成，该服务器返回特定的AVP以指定用户应该使用的VLAN 。 your.org的RADIUS服务器管理员可能忽略了为来自my-org.com访问者分配默认VLAN 。
4. 他们很可能将特殊的DEFAULT域配置为将来自未知领域的请求转发到my-org.com上的RADIUS服务器，而不是为my-org.com创建专用领域。