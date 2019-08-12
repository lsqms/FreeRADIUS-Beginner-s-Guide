# 行动时刻 - 使用JRadius Simulator在FreeRADIUS上测试EAP
我们将首先为JRadius Simulator准备FreeRADIUS环境，然后配置JRadius Simulator以测试EAP身份验证。
## 准备FreeRADIUS
FreeRADIUS可以处理EAP之前最重要的事情就是什么。 当你无所作为时，EAP将发挥最佳作用。 FreeRADIUS作者确保默认配置支持EAP而无需任何调整。

我们只需要确保用户文件中有一个有效用户，并且发送EAP请求的NAS已在clients.conf文件中注册。
1. 编辑位于FreeRADIUS配置目录下的用户文件，并确保有一个alice条目：
`“alice”Cleartext-Password：=“passme”`
2. JRadius Simulator需要从具有GUI和Java的机器运行。记录这台机器的IP地址。该机器将充当FreeRADIUS服务器的NAS（客户端），并且必须在位于FreeRADIUS配置目录下的clients.conf文件中定义。这里我们假设它是192.168.1.101。请根据您的环境进行更改。将此计算机添加为客户端：
```
client jradius {
	ipaddr = 192.168.1.101
	require_message_authenticator =不
	secret = testing123
	nastype = other
}
```
3. 确保已启用内部隧道虚拟服务器，方法是确认它已在FreeRADIUS配置目录下的启用站点的子目录中列出。它默认启用，但如果未列出，请参阅第8章，虚拟服务器以了解如何启用它。
4. 在调试模式下重新启动FreeRADIUS以执行最新更改。
正如您所看到的那样，准备工作不包含任何EAP特定的设置，因为默认情况下一切都应该正常工作。

## 配置JRadius Simulator
如前所述，您需要在运行JRadius Simulator的计算机上安装Java运行时。 JRadius Simulator是一个GUI应用程序，还需要一个可以运行的窗口环境。 如果你运行Linux，它将需要一个桌面环境，如Gnome，KDE或XFCE。 您可以下载JRadius Simulator软件，解压缩并运行它。
或者，如果您的系统具有Java Web Start，则可以从浏览器中启动它。 以下分步过程在Ubuntu中完成：
1. 转到以下URL：http://coova.org/Download
2. 单击JRadius Minimal（客户端）链接开始下载。
3. 下载完成后，打开终端并导航到ZIP文件下载的位置。
4. 解压缩文件：`$> unzip jradius-client-1.1.4-release.zip`
5. 将目录更改为解压缩的jradius文件夹并运行以下命令：`$> sh simulator.sh`
以下屏幕截图显示了上一个命令的输出：

6. 为RADIUS服务器和共享密钥提供值。 单击Log RADIUS to Log选项卡以激活日志记录。
7. 选择“属性”选项卡，然后添加屏幕截图中显示的属性及其值。 您可能必须更改某些值以适合您的环境。 还要确保在AccessReq列中选中它们：
8. 现在可以通过单击RADIUS选项卡然后单击开始按钮开始测试。 您将能够在“日志”选项卡中看到反馈。
9. 浏览各种身份验证协议选项并测试每个选项。
除EAP-TLS之外的所有这些都应该通过身份验证请求。

### 刚刚发生了什么？
我们使用JRadius Simulator程序测试FreeRADIUS上的各种身份验证协议。 我们对以下EAP方法进行了部分测试：EAP-MD5，EAPMSCHAPv2，EAP-TLS，PEAP / MSCHAP和EAP-TTLS / PAP。
> 对于命令行迷，有一个名为eapol_test的JRadius Simulator的替代方案，它是一个包含在wpa_supplicant代码中的实用程序。。 您可以在此位置阅读有关eapol_test的更多信息：
http://hostap.epitest.fi/wpa_supplicant/devel/testing_tools.html

## 配置eap模块
FreeRADIUS中的eap模块与所有其他模块非常相似。 但是，eap模块的配置文件的位置是不同的。 它配置在名为eap.conf的单独文件中，该文件直接位于FreeRADIUS配置目录下，而不是位于modules子目录中。

eap模块默认包含一些EAP方法。 这些方法在eap部分内显示为子部分。 eap部分不能为空。 它必须包括至少一种方法。 如果eap部分为空，则服务器将返回错误，因为它不知道在通信通道中使用哪种身份验证类型。 方法的子部分包含配置特定EAP方法的指令。 下表列出了它们以及简短的注释：
|EAP Method|Comment|
|:-|:-:|
|md5|由于安全性较弱，不推荐使用方法。 无需配置。 没有广泛支持。|
|leap|由思科开发，证明存在缺陷。 既不推荐也不广泛支持。|
|gtc|此方法只能在TTLS和PEAP等隧道方法中使用。 将向RADIUS服务器发送纯文本密码。 如果不支持TTLS/PAP的请求者支持，则可能很有用。|
|tls|可以单独使用，客户端将需要提供客户端证书进行身份验证。这允许相互识别。该方法还用作TTLS和PEAP方法的加密元素的基础。非常安全，但由于客户端证书管理，部署很困难。|
|ttls|创建一个安全隧道，以在其中传输第二个身份验证方法。当FreeRADIUS中的用户存储只能处理明文密码时，通常与PAP一起使用。非常受欢迎，默认情况下支持大多数请求者。很遗憾不支持Microsoft产品。非常安全。使用第二个虚拟服务器进行内部隧道请求。|
|peap|创建类似于TTLS的安全隧道。流行的内部方法是MSCHAPv2和GTC。使用第二个虚拟服务器进行内部隧道请求。遗憾的是，Microsoft产品不支持GTC内部方法。非常安全，非常受欢迎。|
|mschapv2|使用MSCHAP协议进行身份验证。虽然它是安全的，但它没有安全隧道。|

客户端将使用哪种EAP方法的选择主要由两个因素决定：
+ 用户存储支持的方法
+ 客户端支持的方法

## 用户存储
在第5章，用户名和密码的来源中，我们看到一些用户名和密码来源不支持涉及加密的身份验证方法。例如，如果您连接到LDAP服务器并且必须以用户身份进行绑定以确定提供的密码是否正确，则您只能使用可以使用的身份验证协议。在本地存储密码时，如果以加密形式存储密码，可能会严重限制可用的EAP方法。
您可以使用以下URL检查支持哪种加密格式
EAP方法：
http://deployingradius.com/documents/protocols/compatibility.html
http://deployingradius.com/documents/protocols/oracles.html
如上面的链接中所述，如果密码加密错误，则无法使用需要以明文形式存储或访问密码的EAP方法。
如果连接到Novell的eDirectory LDAP服务器，FreeRADIUS可以使用通用密码从服务器以明文形式获取用户密码。有关详细信息，请参阅ldap模块配置文件中的注释。

## 客户端上的EAP
如果Microsoft默认包括EAP-TTLS / PAP支持，或者甚至是来自思科PEAP合作伙伴的PEAP / GTC，那么选择使用哪种EAP方法会更加简单。不幸的是，这是现实世界，我们必须面对现实。许多FreeRADIUS EAP部署不支持MSCHAP身份验证，因为用户存储通常是必须绑定以进行身份​​验证的LDAP服务器。许多Eduroam部署就是这种情况。
要在运行Windows的计算机上支持EAP-TTLS / PAP，您可以默认安装另一个支持此标准的操作系统，或者只需安装一个向Windows添加支持的请求者。许多硬件供应商现在通过其Wi-Fi硬件提供专用请求者。这通常包括对额外EAP方法的支持。或者，您可以加载名为SecureW2的程序。 SecureW2最初是作为GPL许可的程序，增加了EAP-TTLS / PAP支持。随着时间的推移，功能和许可都会发生变化。您现在必须购买许可证才能使用最新版本。然而，有一些较旧的基于GPL的版本在互联网上流传，特别是来自Eduroam的一部分的大学。谷歌是你的朋友，他会为你找到它。
旧版本的SecureW2在64位Windows系统上存在预配置问题。 Windows 7和Windows Vista计算机上可能出现的另一个问题是SecureW2 TTLS 选项不可用作身份验证方法。要解决此问题，请运行services.msc程序并启动Wired Autoconfig服务。您还应该通过编辑属性并将“启动类型”更改为“自动”来使其永久化。
至于大多数其他操作系统，如Apple的iOS，Android，Blackberry OS，Symbian以及各种Linux，他们的请求者通常支持一种方法，允许FreeRADIUS使用LDAP并绑定为服务器的某人进行身份验证。
您现在应该更加熟悉在FreeRADIUS中使用EAP。下一节将介绍一些可以应用于EAP配置和实现的特定于生产的调整。

















