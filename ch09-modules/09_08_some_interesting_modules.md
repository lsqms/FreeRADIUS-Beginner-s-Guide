# 一些有趣的模块
模块子目录下有很多文件。 有些是我们已经知道的模块的特殊命名部分，但其他部分是全新的。 不幸的是，并非所有FreeRADIUS安装都默认包含相同的数字，但最好知道哪些是可用的。
下表列出了一些模块：

|Filename|Module|Functon|
|:-|:-:|-:|
|detail|detail|在NAS和日历日特定的文件中详细记录活动。 如果考虑速度，请禁用此功能。 文件detail.log和detail.example.com包含备用配置。|
|mac2ip|passwd|将MAC地址映射到IP地址。|
|mac2vlan|passwd|将MAC地址映射到VLAN名称。|
|dynamic-clients|dynamic-clients |在FreeRADIUS服务器的客户端具有更改的IP地址时使用。|
|otp|otp|用于实现一次性密码|
|echo|exec|一个模板，它使用exec模块来调用echo外部程序。|
|perl|perl|与FreeRADIUS配置目录中的example.pl文件一起使用的模板。|
|jradius|jradius|一个允许您与Java代码连接的模块。 JRadius项目是Coova套件的一部分，它使用了这个模块。|








