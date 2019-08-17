# 总结
虽然RADIUS协议不需要字典，但它们是FreeRADIUS服务器的核心部分。以下是要记住的关于词典的重要要点列表：

+ 不要修改FreeRADIUS安装的预定义词典文件。
+ 请向NAS供应商咨询是否有任何新的支持的radius属性，以便更新字典。
+ 必须在预定义的字典文件之后获取更新的字典，以便拥有最新的受支持属性。
+ 字典通过将属性名映射到AVP类型号来为我们提供帮助。
+ integer类型的属性可以具有将某个字符串链接到整数值的值定义。

位于/usr/share/freeradius下的字典文件不会被FreeRADIUS自动使用，但必须在/usr/share/freeradius/dictionary文件或FreeRADIUS配置目录的字典文件中明确列出。

快速测验 - 词典

1. Isaac很惊讶地打电话给你。他遵循硬件供应商的指示，将最新的字典文件包含在FreeRADIUS中，现在FreeRADIUS服务器拒绝启动。有什么不对？
2. 您接手离开公司的人，现在负责管理和更新所有Linux服务器。更新Linux服务器后，FreeRADIUS拒绝启动。如果您尝试在调试模式下启动它，则会报告以下错误：
`/etc/raddb/users[11]: Parse error (reply) for entry bob: Invalid octet string "1" for attribute name "ChilliSpot-Max-TotalGigawords"`
可能是什么导致了这种情况发生？
3.您正在阅读FreeRADIUS初学者指南，并在第7章授权中看到作者使用名为FRBG-Reset-Type类型字符串的私有属性。但是，此属性只能是四个值之一（每日，每周，每月或从不）。有一个更好的方法吗？