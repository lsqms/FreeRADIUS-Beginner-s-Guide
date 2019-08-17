# 生产中的EAP
EAP在默认的FreeRADIUS安装中开箱即用。但是，也有一些要点需要注意或更改以适应您的环境。在本节中，我们将介绍以下几点：
+ 适当的公钥基础设施（PKI）的重要性
+ 配置内部隧道虚拟服务器
+ 内部和外部隧道识别的问题
+ 禁用未使用的EAP方法
## 公共密钥基础设施简介
公钥基础结构主要用于两件事：
+ 验证某人的身份
+ 通过不安全的连接交换安全数据
为了确保某人是他们声称的人，我们使用证书颁发机构（CA）。 CA将颁发和签署数字证书。我们可以使用受信任的第三方（TTP）来签发和签署数字证书，或者我们可以成为我们自己的CA。这些证书绑定到标识，并且只应由该标识使用。
当我们处理身份时，我们可以通过与CA核实身份是否确实是其声称的身份来验证其有效性。这对于WPA2-Enterprise安全性非常重要，因为任何人都可以设置接入点并通告特定的SSID。为了验证接入点的有效性，我们需要有一个PKI并使用它。
## 创建PKI
当FreeRADIUS第一次启动时，它通过使用示例配置来设置PKI。这些证书可以在测试期间使用，但是一旦您想要转移到生产环境，您应该创建一个新的集合，以反映您的组织在证书中的值。