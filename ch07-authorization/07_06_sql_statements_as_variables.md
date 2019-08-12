# 行动时间 -  SQL语句作为变量

unlang的一个非常强大的功能是它允许您通过sql模块执行SQL查询。查询实际上是一个变量，此查询的返回值是变量的值。我们现在将修改上一个练习以从数据库中获取时间并将其添加到Reply-Message值。
要执行SQL查询，您需要包含并配置FreeRADIUS以使用sql模块。 sql模块还需要在至少一个部分中使用，例如，授权或记帐部分。
1.在FreeRADIUS下编辑sites-available/default虚拟服务器
配置目录并在该部分顶部的post-auth部分中添加以下内容：
```
if(control:Auth-Type == 'PAP'){
	update reply {
		Reply-Message := "We are using %{control:Auth-Type}
		authentication and the time in the database is now %{sql:SELECT
		curtime();}"
	}
}
```
2.在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证。
指定的Reply-Message应该包含在回复中，并且应该返回在数据库中的时间。

## 刚刚发生了什么？

我们使用SQL语句作为变量，并将此语句的结果作为此变量的值返回。
以下是FreeRADIUS服务器的调试输出，指示SQL语句的执行方式：
```
++? if (control:Auth-Type == 'PAP')
? Evaluating (control:Auth-Type == 'PAP') -> TRUE
++? if (control:Auth-Type == 'PAP') -> TRUE
++- entering if (control:Auth-Type == 'PAP') {...}
sql_xlat
expand: %{User-Name} -> alice
sql_set_user escaped user --> 'alice'
expand: SELECT curtime(); -> SELECT curtime();
rlm_sql (sql): Reserving sql socket id: 3
sql_xlat finished
rlm_sql (sql): Released sql socket id: 3
expand: We are using %{control:Auth-Type} authentication and the
time in the database is now %{sql:SELECT curtime();} -> We are using PAP
authentication and the time in the database is now 17:48:54
+++[reply] returns noop
++- if (control:Auth-Type == 'PAP') returns noop
```
正如您所看到的，SQL查询的处理方式与属性的处理方式大致相同。关于SQL语句作为变量的以下几点很容易记住：
 SQL查询应返回单个值。此值是分配给SQL语句变量的值。
 SQL查询可以引用查询本身内的属性。如果我们想获得当前用户的总使用量，我们可以使用以下行：
`“The total octets is: ％{sql：SELECT IFNULL（SUM（AcctInputOctets + AcctOutputOctets），0）FROM radacct WHERE UserName ='％{UserName}';}”`
 带引号或带反引号的字符串可以包含SQL查询和属性的组合。这些将被扩展以返回结果。
 如果在FreeRADIUS中未使用sql模块，则SQL查询扩展的结果将为空字符串，并且调试消息将显示错误：
`WARNING: Unknown module "sql" in string expansion "%{sql:SELECT
curdate();}"`
扩展的字符串最长可达8000个字符。这为相当复杂的SQL语句留下了足够的空间。