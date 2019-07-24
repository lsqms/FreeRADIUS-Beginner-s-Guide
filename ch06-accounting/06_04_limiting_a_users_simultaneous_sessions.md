# 限制用户账户的同时使用
Isaac是一家无线互联网服务提供商（WISP），他的收入取决于拥有尽可能多的用户，因为他每月收费。 爱丽丝是他的客户。 她给住在隔壁的鲍勃提供了她的证书（账户密码），所以他们都在同一时间连接无线网络。 Isaac需要结束这一点，否则他将不得不关闭他的WISP并编写HTML代码作为食物！

# 行动时刻 - 限制用户账户的同时使用
在sites-enabled/default文件中定义的默认虚拟服务器含有如下会话定义部分：
```
Session database, used for checking Simultaneous-Use.
Either the radutmp or rlm_sql module can handle this.
The rlm_sql module is *much* faster
session {
	radutmp
	See "Simultaneous Use Checking Queries" in sql.conf
	sql
}
```

我们想要一个快速的系统，所以我们将使用推荐的sql。 要将SQL用作会话数据库，我们还需要使用sql进行记帐。

1. 确保您具有第5章中所述的有效SQL数据库配置。
2. 编辑已启用站点/默认文件，并进行以下更改。 我们不会将SQL用作用户存储，因此请在authorize部分注释掉sql。
3. 取消注释会计部分中的sql。 这将导致会计请求写入SQL数据库。
4. 在会话部分取消注释sql，并在会话部分中注释radutmp。 这将启用sql并禁用radutmp。
5. 我们还需要指定将要执行的SQL查询。 这是在sql/mysql/dialup.conf文件中。 取消注释以下部分：
```
simul_count_query = "SELECT COUNT(*) \
FROM ${acct_table1} \
WHERE username = '%{SQL-User-Name}' \
AND acctstoptime IS NULL"
```
6. 修改用户文件，以便alice仅限于一个会话：
`"alice" Cleartext-Password := "passme",Simultaneous-Use := 1`
7. 重新启动FreeRADIUS。 确保FreeRADIUS能够很好地连接到MySQL数据库。
8. 使用radclient和4088_06_acct_start.txt初始化alice的会话：
`$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt`
9. 确认alice有一个打开的会话：
```
radwho
Login Name What TTY When From Location
alice alice shell S0 Sun 22:25 127.0.0.1
```
10. 尝试使用radtest进行身份验证。 您现在应该收到Access-Reject数据包，因为已达到会话限制：
```
$>radtest alice passme 127.0.0.1 100 testing123
Sending Access-Request of id 177 to 127.0.0.1 port 1812
User-Name = "alice"
User-Password = "passme"
NAS-IP-Address = 127.0.0.2
NAS-Port = 100
rad_recv: Access-Reject packet from host 127.0.0.1 port 1812, id=177, length=68
Reply-Message = "\r\nYou are already logged in - access denied\r\n\n"
```
11. 使用radzap命令终止所有活动会话：
```
radzap -N 127.0.0.1 127.0.0.1 testing123
Received response ID 175, code 5, length = 20
```
12. 尝试现在作为alice进行身份验证。 您应该收到一个Access-Accept数据包：

## 刚刚发生了什么？
我们刚刚设法将Isaac从编码HTML中删除。 Alice和Bob也将无法再同时使用互联网。

我们来看一些技术方面。

## 会话部分
FreeRADIUS中的会话部分与授权和验证部分分开定义。 尽管如此，它在功能上是授权的一部分。 FreeRADIUS检查用户是否有权拥有多个会话，然后将用户的限制与当前活动会话数进行比较。 然后，此检查的结果会影响返回到访问请求。

会话部分使用会话数据库来确定用户具有的活动会话。必须使用两个可用会话数据库之一。 Radutmp比sql慢。当您使用sql时，还需要包含sql以进行计费。同样，当您将radutmp用于会话数据库时，还必须包含用于记帐的radutmp。

如果不使用同时使用检查，请通过注释掉radutmp和sql来保持会话部分为空。这将使FreeRADIUS表现更快。另一件需要考虑的事情是当你使用方法来检查同时使用时，尝试只使用一种方法并禁用另一种方法。

## 孤立会话的问题
当您限制用户的会话时，孤立会话现在可以强制阻止用户访问网络。如果NAS上的状态与FreeRADIUS服务器上记录的状态不一致，FreeRADIUS可能会认为用户在NAS知道beter时已登录。请记住这一点，尤其是当NAS关闭并启动时，不会发送Acct-Status-Type = Accounting-Off和Acct-Status-Type = Accounting-On。

## checkrad
会话部分以调用名为checkrad的程序为特色。这是一个Perl脚本，根据客户端的直接类型的值，可以直接联系NAS以确定用户是否已经连接，以及如何进行连接。

如果客户端的nastype = other，则checkrad将不执行任何操作，您可以从此调试输出中看到：

`checkrad: No NAS type, or type "other" not checking`

但是，即使nastype包含其他值，我的设置上的测试也会给出相同的消息。 YMMV，但请注意，因为这也在FreeRADIUS邮件列表中报告。

还要记住，checkrad脚本的执行可能会成为瓶颈，从而减慢身份验证时间。如果要禁用它，只需为客户端定义nastype = other。

我们已经将WISP从破产中拯救出来，是时候进一步限制用户的日常使用。
