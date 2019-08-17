# 漫游 - 概述
漫游允许您在各个地方使用相同的凭据来获取Internet访问权限。 今天有两种非常常见的漫游用法。 让我们看看它们是如何工作的。

## ISP与电信公司之间的协议
考虑下图：

+ Alice是my-isp.com的客户。 My-isp.com没有自己的基础设施。
+ 但是，my-isp.com与当地的电信公司(Telco)签订了协议。 Telco允许my-isp.com的客户使用Telco的DSL Concentrator(集中器)设备连接到Internet。
+ 本地Telco的DSL集中器将首先将认证请求转发给Telco RADIUS服务器。
+ 由于领域（@ my-isp.com），这些请求将被代理到my-isp.com RADIUS服务器。
+ 服务中的本地Telco RADIUS服务器成为my-isp.com RADIUS服务器的另一个客户端。

这样做的一些优点是：

+ my-isp.com不需要自己的基础设施。
+ my-isp.com可与其他基础设施提供商签订多项独立协议，例如移动电话运营商或WISP。

危险包括：

+ my-isp.com依赖第三方来提供部分服务。

## 两个组织之间的协议
为了证明这一点，我们将使用两个图表。 第一个是alice@my-org.com访问your-org.com：

+ 这种类型的漫游用于Eduroam。 两个组织都安装了具有通用SSID的Wi-Fi接入点（在我们的例子中为org.com）。
+ 当alice@my-org.com访问your-org.com时，她只需连接到org.com SSID即可。 这个SSID在my-org.com和your-org.com上是相同的。
+ 然后，Wi-Fi AP将其身份验证请求转发给your-org.com RADIUS服务器。 此RADIUS服务器发现alice@my-org.com是my-org.com的用户，并将请求代理到my-org.com RADIUS服务器。
+ your-org.com中的RADIUS服务器成为my-org.com RADIUS服务器的另一个客户端。

下图显示了当bob@your-org.com访问my-org.com时会发生什么：

+ bob@your-org.com已配置为连接到your-org.com的org.com SSID。 一旦他访问my-org.com，org.com SSID就准备就绪并等待他连接。
+ Wi-Fi AP将Bob的身份验证请求转发给my-org.com RADIUS服务器。 此RADIUS服务器看到bob@your-org.com是your-org.com的用户， 就会将请求代理到your-org.com RADIUS服务器。
+ my-org.com中的RADIUS服务器成为your-org.com RADIUS服务器的另一个客户端。

这样做的好处是：

+ 易连接，提高生产力

面临的一些危险是：

+ 容易滥用网络资源
+ 允许另一个组织处理您组织中用户的用户名和/或凭据可能带来的安全风险

现在我们已经了解了漫游的概况，现在是时候让我们的手忙碌起来了，看看如何在FreeRADIUS中完成代理以便允许漫游。 本章的其余部分将分为两部分。 我们从领域开始，在领域之后我们讨论代理。