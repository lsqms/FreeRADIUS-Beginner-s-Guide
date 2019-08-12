# 行动时刻 - 设置变量的默认值

我们并不总是确定变量是否存在。 Unlang为我们提供了语法，以便在变量不存在时指定默认值。 我们将再次通过修改上一个练习来证明这一点。 我们将使用radtest首先在请求中包含Framed-Protocol = PPP，并将其排除在外。 如果Framed-Protocol AVP不存在，我们将返回一个默认字符串。

1. 编辑FreeRADIUS配置目录下的sites-available / default虚拟服务器，并在该部分顶部的post-auth部分中添加以下代码：
```
if(control:Auth-Type == 'PAP'){
	update reply {
		Reply-Message := "Framed protocol is:
		%{%{request:Framed-Protocol}:-Not in request}"
	}
}
```
2. 在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证。首先在radtest命令的末尾添加1，然后省略1。
3. 在radtest中添加1的Reply-Message的值应为：
`Reply-Message = "Framed protocol is: PPP"`
4. radtest中省略1的Reply-Message值应为：
`Reply-Message = "Framed protocol is: Not in request"`

## 刚刚发生了什么？
我们已经使用unlang的功能为变量分配默认值。
 变量引用中的:-字符序列是unlang的指示，当第一个变量不存在时，它应该尝试使用以下内容:- 字符序列。 请注意以下要点：
  如果:- 后跟一个不带引号的字符串，它将返回该字符串。
  如果:- 后跟对另一个变量的引用，您可以创建一个链以最终测试是否存在多个变量，例如：
％{％{request：Framed-Protocol}:- ％{request：NAS-Name}:- Default value}
  此语法称为条件语法，并在FreeRADIUS版本之间进行更改。  某些模块仍然使用较旧的语法，这将导致调试消息中出现警告。 LDAP模块就是这样一个例子。
您可以在ldap模块的配置文件中更改以下行：
`filter = "(uid=%{Stripped-User-Name:-%{User-Name}})"`
更改为
`filter = "(uid=%{%{Stripped-User-Name}:-%{User-Name}})"`
