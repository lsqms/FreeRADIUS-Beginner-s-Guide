# 行动时刻 - 为您的组织创建RADIUS PKI
本书的目的不是取代现有的文档。 FreeRADIUS配置目录下的certs子目录中有一个很好的README文件。按照说明为您的组织创建一组新证书。
如果你有一个辅助FreeRADIUS服务器，你可以使用server.cnf文件;备份主FreeRADIUS服务器的配置并修改它以为辅助RADIUS服务器创建证书。注意不要覆盖主FreeRADIUS服务器的文件。

## 刚刚发生了什么？
我们为我们的组织创建了一个特定的PKI。 EAP请求者应使用CA来确认RADIUS服务器的有效性。
### 为什么要使用PKI？
使用EAP-TTLS或PEAP的每个客户端都必须将新创建的CA证书添加到请求者中的可用CA列表中。如果您未能这样做，则会产生巨大的安全风险。下图应该说明一切：

![client_add_newly_created_CA_certificate_to_the_list](https://github.com/lsqms/FreeRADIUS/blob/master/image/ch10/client_add_newly_created_CA_certificate_to_the_list.PNG?raw=true)

如果客户端未检查来自RADIUS服务器的证书的有效性，则任何人都可以创建恶意设置并从组织的用户那里获取一些密码。
README文件还建议不要使用根CA创建的证书，因为这可能会允许黑客从同一根CA获取证书，并在用户认为安全的情况下在部署中使用此证书。
现在PKI已准备好生产，tls，ttls和peap方法可以使用新创建的证书。 ttls和peap方法都要求正确配置tls方法，因为它们将其用作加密功能的基础。
## 将CA添加到客户端
一些请求者允许我们在配置EAP-TLS，EAP-TTLS或PEAP时从操作系统的可信CA列表中进行选择。不幸的是，将新创建的CA添加到该列表中有时候是一场战斗。下表列出了一些操作系统和有关在请求方中包含新CA的注意事项：
|Operatng system|Comment on CA in supplicant|
|:-|:-:|
|Android|从版本2.2开始，有关添加新CA成功的报道不一。 在我的手机上我可以添加CA，选择它，但是请求者拒绝承认RADIUS服务器证书的有效性。|
|Apple|Apple使用.mobileconfig文件轻松添加新的Wi-Fi配置文件。 此文件包含完整的Wi-Fi配置文件，包括CA，只需使用Safari Web浏览器下载文件即可添加该文件。|
|Blackberry|在旧型号上，您需要使用连接到手机的Windows应用程序添加CA. 较新的型号允许您使用浏览器添加CA。|
|Linux|大多数发行版中使用的NetworkManager小程序允许您选择手动选择CA证书。|
|Windows|Windows有多个可以添加根CA的地方。 有些仅将CA应用于用户，其他地方将CA应用于整个计算机。添加新的CA证书非常简单。 将其添加到正确的位置可能会更加困难。 Windows 7要求您在某些计算机上使用“以管理员身份运行”选项导入证书。|

由于操作系统市场是一个在短时间内发生升级，改进和新的参与者加入的快速发展的市场，因此最好在寻求CA时咨询一个好的搜索引擎。
eap.conf文件中ttls和peap方法的配置内部是一个名为virtual_server的指令。这将在下面讨论。
## 配置内隧道虚拟服务器
如果查看FreeRADIUS配置目录下的eap.conf文件，ttls和peap方法都包含virtual_server指令。该指令指向将处理内部EAP隧道的身份验证请求的虚拟服务器。默认情况下，内部隧道虚拟服务器用于此目的。此虚拟服务器独立于默认虚拟服务器。在默认虚拟服务器中包含sql或ldap等模块时请记住这一点。您还必须将其添加到内部隧道虚拟服务器，以便EAP-TTLS和PEAP使用相同的用户存储。
拥有一个处理内部隧道验证的虚拟服务器可以为FreeRADIUS增加灵活性。这使我们可以拥有两个不同的用户存储，甚至可以在内部隧道虚拟服务器中包含完全独立于默认虚拟服务器的unlang逻辑。


