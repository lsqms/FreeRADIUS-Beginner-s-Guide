# 定义和启用虚拟服务器
FreeRADIUS默认启用了两个虚拟服务器。 它们位于FreeRADIUS配置目录的已启用站点的子目录下。 他们是：
  default：该名称几乎说明了虚拟服务器的功能。 此虚拟服务器处理未明确指定由virtual_server指令处理的所有缺省请求。 到目前为止，我们一直使用这个虚拟服务器。
  inner-tunnel：此虚拟服务器用于某些隧道式EAP请求，如TTLS和PEAP。

这两个虚拟服务器允许FreeRADIUS处理正常的RADIUS身份验证请求（默认）以及开箱即用的EAP / TTLS和EAP / PEAP请求（内部隧道）。
如果查看位于FreeRADIUS配置目录下的eap.conf文件，您可以看到指定内部隧道虚拟服务器的两个EAP方法的配置。 以下是eap.conf文件的摘录：
```
eap {
...
ttls {
	{
		...
		virtual_server = inner-tunnel
	}
}
```
FreeRADIUS遵循与Apache相同的约定，其中虚拟服务器在站点可用(sites-available)目录下定义，并通过创建指向启用站点(sites-enabled )的目录的符号链接来激活。 这些文件的内容通常是一个命名的服务器部分，其名称对应于文件名。