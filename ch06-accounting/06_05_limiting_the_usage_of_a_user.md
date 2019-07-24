# 限制用户的使用
如本章开头所述，我们可以使用会计数据进行容量规划。本章的其余部分将介绍我们根据指定用户的现有会计数据限制用户的每日使用情况的情况。

## 每天30分钟
Isaac的WISP再次流行，当地的比萨店已经与他接洽，为他们的客户提供互联网接入。每个客户购买的每个披萨都可以免费上网30分钟。此免费互联网必须仅在一天内有效，并且应在购买比萨饼的当天22:00到期。 Isaac使用Coova Chilli和Mikrotk俘虏门户组合作为他的WISP。
## FreeRADIUS如何提供帮助
在介绍RADIUS协议的过程中，我们指出RADIUS服务器不能对用户施加限制。虽然RADIUS服务器将返回AVP以指示某些限制，但NAS负责强制它们。

常见的限制是数据或基于时间。 Session-Timeout返回AVP是基于时间的，并且被许多NAS设备理解。如果我们只想让用户的会话持续30分钟，只需在Access-Accept数据包中返回Session-Timeout = 1800。如果NAS支持Session-Timeout，它将在30分钟后终止用户的会话。

>查看NAS支持哪些AVP的最佳位置是搜索特定供应商或项目的网站。

除了一个问题，此方案工作正常。用户可以在超时后再次登录，再获得30分钟！

但是，有一种方法可以解决这个问题。 FreeRADIUS提供计数器模块。在本节中，我们将首先使用计数器模块（rlm_counter）执行基本计数器设置，以跟踪会话间的tme使用情况。然后，我们将使用sqlcounter模块（rlm_sqlcounter）替换计数器模块。

## 行动时刻 - 限制用户的使用
以下部分将演示如何限制Internet使用。

## 激活每日计数器
计数器模块默认定义了以下计数器：
```
counter daily {
	filename = ${db_dir}/db.daily
	key = User-Name
	count-attribute = Acct-Session-Time
	reset = daily
	counter-name = Daily-Session-Time
	check-name = Max-Daily-Session
	reply-name = Session-Timeout
	allowed-servicetype = Framed-User
	cache-size = 5000
}
```
修改计数器如下：

1. 编辑FreeRADIUS配置目录中的sites-enabled/default文件。 在授权和会计部分取消daily注释。 此外，在radius.conf文件的实例化部分取消daily注释，以确保计数器的正确即时。
2. Max-Daily-Session和Daily-Session-Time AVP 未列入任何dictonary。 编辑FreeRADIUS配置目录中的字典文件并添加它们：
```
ATTRIBUTE My-Local-String 3000 string
ATTRIBUTE My-Local-IPAddr 3001 ipaddr
ATTRIBUTE My-Local-Integer 3002 integer
ATTRIBUTE Daily-Session-Time 3000 integer
ATTRIBUTE Max-Daily-Session 3001 integer
```
3. 更改用户文件中alice的条目以反映以下内容：
```
"alice" Cleartext-Password := "passme", Max-Daily-Session := 1800
Reply-Message = "Hello, %{User-Name}"
```
4. 编辑FreeRADIUS配置目录中的modules/counter文件，并注释掉allowed-servicetype行：
`allowed-servicetype = Framed-User`
5. rlm_counter模块将创建一个数据库文件，以跟踪各种用户的计数器。它位于FreeRADIUS配置目录中。我们需要更改此目录的权限。我们假设每个发行版都有正常安装。
在CentOS和SUSE上：
`＃> chmod g + w / etc / raddb`
在Ubuntu上一切正常，无需进行任何更改。
6. 在调试模式下重新启动FreeRADIUS服务器。
7. 认证为爱丽丝。您应该收到Session-Timeout = 1800：
```
radtest alice passme 127.0.0.1 100 testing123
Sending Access-Request of id 30 to 127.0.0.1 port 1812
User-Name = "alice"
User-Password = "passme"
NAS-IP-Address = 127.0.0.1
NAS-Port = 100
rad_recv: Access-Accept packet from host 127.0.0.1 port 1812,
id=30, length=40
Reply-Message = "Hello, alice"
Session-Timeout = 1800
```
8. 使用radclient和我们在本章前面创建的4088_06_acct_start.txt文件发送计费开始请求。等待30秒或更长时间。使用radclient和4088_06_acct_stopt.txt文件发送计费停止请求。这将记录使用情况。
9. 再次作为爱丽丝进行身份验证。这次Session-Timeout现在应该是1770或更少而不是1800（1800 - 30）。

此练习显示了如何使用计数器模块来跟踪会话的时间使用情况。根据现有用途，它将返回指定的AVP。如果现有使用已达到触发点，则身份验证将失败。

## 在指定时间终止会话
第二个要求是互联网访问应该只在购买比萨饼的那天的22:00之前有效。为此，我们将使用WISPr-Session-Terminate-Time返回AVP。 Coo Chilli和Mikrotk都支持WISPr-Session-Terminate-Time。此AVP的值可以指定应终止用户连接的精确时间（到秒）。

>**创建Internet凭证**
>
>虽然这部分超出了本书的范围，但我建议使用SQL数据库。 然后，创建互联网优惠券的软件应根据披萨销售日确定WISPr-Session-Terminate-Time的价值。
>
>您可能想要使用Login-Time而不是WISPr-Session-Terminate-Time，但Login-Time将允许任何一天使用凭证。

对于这个概念证明，我们只需在用户文件中为alice添加一个reply属性。我们设想今天是2012年1月10日星期二，时区是UTC/GMT + 2小时。

1. 更改用户文件中alice的条目以反映以下内容：
```
"alice" Cleartext-Password := "passme", Max-Daily-Session :=
1800
Reply-Message = "Hello, %{User-Name}",
WISPr-Session-Terminate-Time = "2012-01-10T22:00:00+02:00"
```
2. 重新启动FreeRADIUS服务器。
3. 认证为爱丽丝。返回AVP应包括Session-Timeout和WISPr-Session-Terminate-Time：
```
rad_recv: Access-Accept packet from host 127.0.0.1 port 1812,
id=19, length=73
Reply-Message = "Hello, alice"
WISPr-Session-Terminate-Time = "2012-01-10T22:00:00+02:00"
Session-Timeout = 1770
```
4. 确保Coova Chilli和Mikrotik强制门户的时区正确无误。如果不正确，您可能会遇到错误的终止时间。

## 刚刚发生了什么？
这个练习证明了我们如何能够将客户在互联网上的每日总时间限制为30分钟。我们还确保他们在购买比萨饼的当天22:00之后不再连接。让我们来看一下计数器模块打勾的原因，还是计数？

## rlm_counter
计数器模块允许您定义各种计数器。包含的计数器是基于时间的，每天调用，因为它每天重置。计数器模块为定义的每个计数器创建自己的数据库。让我们来看看每日计数器样本的定义：
```
counter daily {
	filename = ${db_dir}/db.daily
	key = User-Name
	count-attribute = Acct-Session-Time
	reset = daily
	counter-name = Daily-Session-Time
	check-name = Max-Daily-Session
	reply-name = Session-Timeout
	allowed-servicetype = Framed-User
	cache-size = 5000
}
```

每个计数器部分包含各种指令，用于定义计数器的行为。
有四条指令构成了计数器的核心：

|指令|值|注释|
|:-|:-:|:-:|
|check-name|Max-Daily-Session|内部检查AVP以供用户指示允许; 例如：每日最多会话：= 1800请记住，此AVP通常是内部AVP，您可以在字典文件中明确定义，并且应该是3000到4000之间的值。|
|count-attribute|Acct-Session-Time|会计数据包中的AVP要保持计数。|
|reply-name|Session-Timeout|Session-Timeout = Max-Daily-ession minus Acct-Session-Time|
|reset|daily|要考虑的时间跨度。 值可以是每日，每周，每月或从不。|

>简而言之，计数器模块将在指定的重置周期内计算count-attribute的总使用量; 然后它会从check-name中减去这个值。 如果小于零则返回失败; 如果它大于零，它将返回reply-name的值。

您可以在计数器文件的注释中阅读有关其他指令及其用法的更多信息。

我们已经删除了allowed-servicetype指令，因为它限制计数器仅在Access-Request包含Service-Type = Framed-User时才有效。

要激活已定义的计数器，必须在“计费”部分和“授权”部分中指定。它也应该在radius.conf文件的instantiate部分中指定。这将确保正确启动计数器。

您将注意到，每日计数器都列在可以在授权部分提供用户信息的所有模块之后。这将使这些模块有机会在每天执行之前设置检查名称。在我们的情况下，这是由文件模块在将Max-Daily-Session的值设置为1800时完成的。

记帐部分跟踪使用情况，并让计数器模块将使用记录到指定的数据库中。在授权期间，计数器将查询指定的数据库以确定回复名称AVP的值。如果使用率超过checkname AVP，则返回Access-Reject。

如果所有计数器都可以使用一个数据库，那么不是为每个定义的计数器建立数据库，而是更有效。在下一节中，我们将使用sqlcounter。 sqlcounter模块使用sql记帐数据库来确定计数器值，而不是定义了多少计数器。

## 试一试 - 使用单个数据库的各种计数器
我们现在看一下从单个数据库运行多个计数器。
## 使用rlm_sqlcounter
当我们配置FreeRADIUS来限制用户的会话时，我们在会计部分包含了sql。我们假设您仍然在默认虚拟服务器的记帐部分中包含sql。

在本练习中，我们构建了之前使用sql模块进行记帐的方法。我们使用MySQL作为数据库。 FreeRADIUS还支持PostgreSQL，Microsoft SQL Server和Oracle数据库，作为MySQL的替代品。这个sqlcounter模块应该与替代方案一样好用。

我们将使用等效的sqlcounter替换daily计数器：

1. sqlcounter在sql/mysql/counter.conf文件中包含一些预定义的计数器，该文件位于FreeRADIUS配置目录中。打开文件并确认已定义名为dailycounter的计数器。
2. 改变dailycounter的sql指令：
更改前:
```
query = "SELECT SUM(acctsessiontime - \
GREATEST((%b - UNIX_TIMESTAMP(acctstarttime)), 0)) \
FROM radacct WHERE username = '%{%k}' AND \
UNIX_TIMESTAMP(acctstarttime) + acctsessiontime > '%b'"
```
更改后:
```
query = "SELECT IFNULL(SUM(acctsessiontime - \
GREATEST((%b - UNIX_TIMESTAMP(acctstarttime)), 0)),0) \
FROM radacct WHERE username = '%{%k}' AND \
UNIX_TIMESTAMP(acctstarttime) + acctsessiontime > '%b'"
```
3. 编辑FreeRADIUS配置目录中的sites-enabled/default文件。 通过注释将每日在授权和会计部分中删除。
4. 在授权部分的每日注释下方添加dailycounter。
5. 确认sql包含在计费部分中。
6. 清理MySQL数据库中的radacct表：
```
mysq -u root -p radius
delete from radacct;
```
7. 在调试模式下重新启动FreeRADIUS。
8. 认证为爱丽丝。 您应该收到Session-Timeout = 1800：
```
$>radtest alice passme 127.0.0.1 100 testing123
Sending Access-Request of id 181 to 127.0.0.1 port 1812
User-Name = "alice"
User-Password = "passme"
NAS-IP-Address = 127.0.0.1
NAS-Port = 100
rad_recv: Access-Accept packet from host 127.0.0.1 port 1812,
id=181, length=73
Reply-Message = "Hello, alice"
WISPr-Session-Terminate-Time = "2012-01-10T22:00:00+02:00"
Session-Timeout = 1800
```
9. 使用radclient和我们在本章前面创建的4088_06_acct_start.txt文件发送记帐开始请求。 使用4088_06_acct_interim-update.txt文件进行interimupdate跟进（使用sql 计费，您不必在两者之间等待）。
10. 再次作为Alice进行身份验证。 Session-Timeout的值现在应该是1789（1800-11）。
11. 使用radclient和我们在本章前面创建的4088_06_acct_stop.txt文件发送一个计费停止请求。
12. 再次以爱丽丝进行身份验证。这次Session-Timeout的值应该是1770（1800-30）。

现在耗尽用户的可用时间：

1. 编辑4088_06_acct_stop.txt文件，并将Acct-Session-Time的值更改为2000。
2. 使用分别具有4088_06_acct_start.txt和4088_06_acct_stop.txt的radclient发送记帐开始和记帐停止请求。
3. 再次验证为爱丽丝。这次验证失败，因为可用时间耗尽：
```
rad_recv: Access-Reject packet from host 127.0.0.1 port 1812,
id=143, length=84
Reply-Message = "Hello, alice"
Reply-Message = "Your maximum monthly usage time has been
reached"
```
4. 回复消息可能会产生误导，因为它说的是每月使用而不是每天使用;尽管如此，请求仍被拒绝。

我们设法用sqlconter模块替换计数器模块。让我们来看一些要记住的要点。
## 重置计数器
reset指令有五个可供选择的可能性。这些可能性是每小时，每天，每周，每月，从不。这些值是基于日历的。这意味着每月重置将在本月的第一天发生。我们还有机会以num [hdwm]的形式定义我们自己的重置值，其中每个字母代表一个重置选项。如果我们再指定reset = 6h，计数器将每隔六小时重置一次，从00:00开始。
## SQL模块实例
我们可以声明sql模块的各种实例。默认情况下，第一个实例称为sql。但是，我们可以创建其他的，每个连接到不同的数据库甚至是不同的服务器。使用sqlmod-inst = sql，我们指示sqlcounter使用默认的sql模块实例。如果我们使用了一个额外的sql实例来进行记帐，那么可以将sql模块实例指定为sqlmod-inst的值。

## 查询中的特殊变量
您将注意到查询列出了一个特殊的单字符变量（％k）来表示键的属性。这意味着如果key = User-Name，那么我们将使用User-Name替换％k。还有％b，用复位周期的开始和％e代替，％e用复位周期的结束代替。这些单字符变量对于sqlcounter模块是唯一的，并且与已存在的变量相加。变量将在下一章深入介绍。

## 空帐户记录
我们必须修改计数器的SQL查询以处理NULL值。如果未指定此参数，则当SQL数据库中没有针对尝试进行身份验证的用户的记帐记录时，查询将返回NULL。这反过来将导致不返回Session-Timeout应答属性。
请记住将IFNULL与其他预定义计数器一起使用，以及定义自己的计数器时。如果你忘记了这一点，它会咬你！
##每天重置的计数器
每日计数器在午夜重置。爱丽丝在午夜前20分钟连接会发生什么？爱丽丝将获得今天剩下的20分钟，并且还将限制在第二天30分钟。要防止会话在任何时间持续超过30分钟，请使用Login-Time内部AVP以确保用户的会话在午夜之前到期：
`"alice" Cleartext-Password := "passme", Max-Daily-Session :=
1800,Login-Time := 'Al0001-2359'`
如果没有登录时间AVP，我们将在23:30之后进行身份验证时获得以下回复：
`Session-Timeout= 3417`
使用Login-Time AVP，用户将获得以下内容：
`Session-Timeout = 1440`
Session-Timeout值超过30分钟的事实可能令人困惑，但是这个人只是在今天30分钟的剩余时间加上明天的30分钟。

## 计算八位字节
您可能想要使用如下查询为八位字节创建计数器：
`query = "SELECT IFNULL(SUM(acctinputoctets - GREATEST((%b - UNIX_TIMESTAMP(acctstarttime)), 0)),0)+ IFNULL(SUM(acctoutputoctets -GREATEST((%b - UNIX_TIMESTAMP(acctstarttime)), 0)),0) FROM radacct WHERE username='%{%k}' AND UNIX_TIMESTAMP(acctstarttime) + acctsessiontime > '%b'"`
不要这样做！我重申不要这样做！ check-name和reply-name指令的AVP必须是基于时间的。当reset指令定义为never时，它可以按预期工作，但有一天你会将它更改为每日或每月并被销毁。

事情中断的原因是（如上一节所示）如果今天的剩余秒数小于为用户定义的Max-Daily-Session，则计数器返回今天的余数加上明天的Max-Daily-Session。因此，它假设这些AVP是基于时间的AVP。

在计数器定义中使用其他AVP不会改变sqlcounter模块的行为。它假定这些AVP是基于时间的。

我们将讨论基于本书后面的累计使用来控制用户数据的替代方法。

>如果您想了解此限制的详细信息，可以在以下邮件列表中阅读更多相关信息：
>
>http://www.mail-archive.com/freeradius-users@lists.freeradius.org/msg49267.html

现在已经配置了FreeRADIUS的计费部分，现在是时候看看我们应该做些什么来确保计费数据得到很好的维护。
