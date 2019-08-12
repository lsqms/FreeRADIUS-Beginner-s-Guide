# 总结
授权可以成为FreeRADIUS中最复杂的部分。通过充分利用所提供的资源，我们几乎可以克服所有可能的问题。
在本章中，我们介绍了：
+ 限制的应用：可以在RADIUS服务器或NAS设备上应用限制。
+ Unlang：Unlang是一种强大的处理语言，它允许我们操纵FreeRADIUS处理传入请求的方式。它具有可以控制请求流的条件检查。它还允许与某些模块（如sql模块）进行交互，以从SQL数据库获取结果。 Unlang使我们能够操作和添加将随Access-Accept数据包返回的AVP。任何想要在FreeRADIUS中创建灵活和版本配置的人都应该掌握unlang的使用。

随着本章关于授权的结束,我们现在已经完成了AAA框架的覆盖范围。本书的其余部分将重点介绍RADIUS的更高级主题，以及FreeRADIUS特有的主题。

快速测验 - 授权
1. 您正在NAS上实施限制，返回一个AVP，该AVP应该为用户强制执行带宽调整。 它似乎无法正常工作。 可能有什么不对？
2. 像任何铁杆IT老兄一样，你希望你的FreeRADIUS服务器超级快。不幸的是，您必须使用外部代码来获取属性的值。 这可以通过Bash或Perl完成。 哪个选项会产生最佳性能？
3. 您将在哪里定义将由unlang内部使用的属性？
4. 什么是存储内部属性的属性列表，以及如何在此列表中引用Auth-Type属性？
5. 您继承了FreeRADIUS部署，在浏览配置文件时，您会在policy.conf文件中遇到以下unlang代码：
```
rewrite_calling_station_id {
	if(request:Calling-Station-Id =~ /([0-9a-f]{2})[-:]?([0-9af]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})
	[-:]?([0-9a-f]{2})/i){
		update request {
			Calling-Station-Id := "%{1}-%{2}-%{3}-%{4}-%{5}-
			%{6}"
		}
	}
	else {
		noop
	}
}
```
这段代码有什么作用？
