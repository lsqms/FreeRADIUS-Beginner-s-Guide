# 摘要
我们在本章中学到了很多关于FreeRADIUS计费的知识。具体来说，我们已经介绍了：

+ 基本计费：我们了解到计费与认证和授权是分开的，并且在端口1813上运行。它由客户端发送Accounting-Request数据包和服务器回复Accounting-Response数据包组成。 Accounting-Request中的Acct-Status-Type AVP可以具有Start，Stop，Interim-Update，Accounting-Off或Accounting-On的值。
+ 恶意计费数据：这些也称为孤立会话，当FreeRADIUS服务器的会计数据不反映NAS上的actvites时发生。
radzap命令帮助我们控制这些数据。
+ 同时会话：用户的同时会话可能存在限制。FreeRADIUS中的会话部分指定应该引用的会话数据库。会话数据库从会计部分获取会话数据。为了限制同时进行的会话，我们使用内部的Simultaneous-Use AVP作为对用户的检查。
+ 计数器：FreeRADIUS有计数器来跟踪用户的总使用情况。这可用于限制用户具有网络访问的总时间。 rlm_counter模块为每个定义的计数器使用自己的私有数据库。 rlm_sqlcounter模块搭载到sql accounting数据库，这是更有效的。

我们还讨论了访问和管理MySQL数据库的计费数据以及与计费相关的常见问题的方法。
下一章将破解开放授权的外衣，向您展示FreeRADIUS可以为您提供多少功能。

## 快速测验 - 计费
1. Telco正在将RADIUS身份验证请求转发到您的RADIUS服务器。一切都很好。他们现在也能够将会计请求转发到您的RADIUS服务器，但不知何故，没有一个请求到达您的RADIUS服务器。哪里是排除故障的好地方？
2. 您已配置了同步会话限制，它就像有魔力。在夜间，一场激烈的暴雨摧毁了其中一座Wi-Fi塔。现在有些人抱怨他们无法连接，尽管他们有来自附近另一座塔楼的信号。可能有什么不对？
3. 您为WisPr-Session-TerminateTime生成具有特定值的凭证。你的一些强制门户网站似乎忽略了这个回复AVP，虽然供应商确实支持这个AVP。可能有什么不对？
4. 使用sqlcounter模块，您已创建计数器以限制用户的每日数据。不知何故，这个反击只是表现得很奇怪。您将reset指令更改为never，并且它变得稳定。你疯了吗？