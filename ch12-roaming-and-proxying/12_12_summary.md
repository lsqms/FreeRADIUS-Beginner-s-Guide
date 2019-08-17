# 总结
对于任何必须集成到更大的RADIUS服务器网络中的FreeRADIUS部署而言，领域的创建和代理的设置都是自然而然的过程。让我们来看看本章要记住的重点：

+ 领域在proxy.conf文件中定义，用于确定是否必须将请求转发到外部主服务器。
+ LOCAL，NULL和DEFAULT是特殊领域。 LOCAL始终存在，用于取消代理。 NULL用于在没有领域的情况下对用户名进行分组，DEFAULT用于对来自未知领域的用户名进行分组。
+ 对于接收转发请求的外部主服务器，代理请求的服务器必须被定义为主服务器上的客户端。
+ 使用nostrip选项定义的域(realms)将导致后缀模块不将Stripped-User-Name AVP添加到请求中。通常在将请求转发到外部主服务器时选择nostrip选项。
+ 当我们将请求代理到其他RADIUS服务器时，从这些服务器过滤回复AVP很重要。
+ 当我们希望转发计费数据时，我们可以利用FreeRADIUS中的集成radrelay功能来创建一个能够处理网络中断的强大服务器。

## 快速测试 - 漫游和代理
1. 您在一家名为my-org.com的公司工作，该公司刚刚协商了一项允许在my-org.com和另一家名为your-org.com的公司之间漫游的协议。您使用FreeRADIUS，your-org.com使用Radiator。尽管有不同的RADIUS服务器软件，你能配置漫游吗？
2. 在使用公共SSID为org.com的Wi-Fi网络上使用EAP配置和测试漫游后，来自your-org.com的用户访问my-org.com。他想知道my-org.com上使用的EAP方法，并且还想加载my-org.com的CA以连接到org.com SSID。这是必需的吗？
3. your-org.com已升级其网络并正在实施动态VLAN。从那时起，当my-org.com的用户访问他们时，他们无法访问Internet。有什么不对？
4. 第三家公司正在加入漫游协议。在他们配置了FreeRADIUS服务器之后，您会看到许多请求也转发到您的FreeRADIUS以用于其他领域。你怀疑他们在他们那边做了什么？