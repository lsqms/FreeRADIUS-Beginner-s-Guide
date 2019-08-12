# 行动时刻 - 使用正则表达式
Unlang允许在条件检查中进行正则表达式计算。这些通常是Posix正则表达式。运算符=〜和！〜与正则表达式相关联。为了简单的概念证明，我们将修改上一个练习：
1.编辑FreeRADIUS配置目录下的sites-available / default虚拟服务器，并在该部分顶部的post-auth部分中添加以下内容：
```
if(request:Framed-Protocol =~ /.*PP$/i){
	update reply {
		Reply-Message := "Regexp match for %{0}"
	}
}
```
2.在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证。
首先在radtest命令的末尾添加1，然后省略1。
请注意，将1添加到radtest命令时，正则表达式匹配如何更改Reply-Message的值。

## 刚刚发生了什么？
我们已经展示了unlang的正则表达式功能。请注意unlang中正则表达式的以下要点：
+ 运算符=〜和！〜与正则表达式一起使用。
+ 正则表达式在两个/字符之间指定。
+ 正则表达式允许您引用其中的变量，例如 `/^％{Framed-Protocol}$/i`。
+ 您可以在正则表达式的末尾添加可选的i字符，以使搜索大小写不敏感。
+ 如果匹配，则特殊变量％{0}保存在正则表达式中测试的变量的值。

这就结束了unlang的介绍 您现在应该知道unlang中使用的大多数构建块。 下一个部分将充分利用这些构建块来创建unlang的真实应用。