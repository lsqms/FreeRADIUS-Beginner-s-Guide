# 预先定义的虚拟服务器
FreeRADIUS包括站点可用子目录下的虚拟服务器。有些可以按原样使用，而有些则是用于特殊要求的模板。以下是一些虚拟服务器：
+ buffered-sql：此虚拟服务器用于克服大型SQL数据库（type = detail）的速度限制。
+ copy-acct-to-home-server：此虚拟服务器可用作模板，用于在两个位置记录一个计费请求（type = detail）。
+ coa：用于处理coa（权限更改Change of Authority）和pod（断开数据包Packet of Disconnect）请求的模板（type = coa）。
+ decoupled-accounting：解耦计费的模板。工作原理与buffered-sql虚拟服务器（type = detail）相同。
+ status：从FreeRADIUS服务器获取状态信息的虚拟服务器（type = status）。

正如我们在本章开头所指出的那样，虚拟服务器的优势在于它们的灵活性。由于虚拟服务器非常灵活，因此在创建和实现虚拟服务器时没有固定的规则。最好的方法是进行实验以获得更多经验。 Perl的moto也适用于此。有多种方法可以做到这一点（TIMTOWTDI）。