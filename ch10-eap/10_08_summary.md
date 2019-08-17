# 总结
以下是我们在本章中提到的一些要点：
+ EAP是一个以可扩展性为核心特征的框架。这允许引入新的EAP方法而无需对验证器进行任何更改。
+ EAP允许我们在使用EAP-TTLS或PEAP时通过第三方RADIUS服务器代理请求，而不会泄露用户的用户名和密码。
+ 隧道EAP方法有两个身份，可以相互比较。
+ 建议使用和分发专用的自签名CA以获得最大安全性。教育用户安装并指定在请求者配置中使用自签名CA.
+ 在向RADIUS服务器发送计费详细信息时，验证者将使用Access-Accept中返回的用户名AVP的值。
+ 在测试各种EAP方法时，JRadius Simulator程序非常方便。

## 快速测验 -  EAP
1. 您刚刚安装了FreeRADIUS，在使用强制门户作为客户端进行初始测试后，您希望通过将接入点（AP）配置为客户端来测试EAP。您的同事会向您发送一些用于使EAP在FreeRADIUS上工作的URL。接下来你应该怎么做？
2. 强制网络门户允许来自用户文件和公司LDAP服务器的用户，但EAP-TTLS / PAP配置的请求者仅允许访问用户文件中的用户。哪里出错了？
3. 您指定的LDAP配置以用户身份绑定以验证用户名和密码。你可以用PEAP / MSCHAPv2吗？
4. 您的朋友有一台LDAP服务器，它是Novell电子目录的一部分。他说，他们很高兴地使用它和freeradius来验证使用PEAP / MSCHAHv2的请求者。这是谎言吗？
5. 在iPhone和iPad用户设备上获取包括CA的Wi-Fi配置文件的最简单，最有效的方法是什么？