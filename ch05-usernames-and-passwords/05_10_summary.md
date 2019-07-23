# 摘要
这一章很大，涵盖了很多内容。 通过典型部署，您可能会使用一个或两个用户存储。  作为总结，让我们重新审视这里讨论的每个用户存储的重点。

## Linux系统用户
unix模块（rlm_unix）需要访问/etc/shadow文件才能读取用户的加密密码。 pap模块使用此加密密码来验证用户。 CHAP和MS-CHAP身份验证不起作用; 只有PAP验证才能与系统用户配合使用

## SQL数据库
FreeRADIUS支持各种SQL数据库。它通过通用SQL模块和数据库特定SQL模块的组合来实现。数据库纯粹用作数据存储，并保留与用户文件相同类型的数据。用户可以属于一个或多个组。这简化了管理。用户的User-Profile属性允许我们为用户分配配置文件。配置文件比向组添加用户更灵活。
## LDAP目录
LDAP可以通过两种方式使用：

+ 第一种方法是使用“bind as user”进行身份验证。所有LDAP服务器都支持这种方式，但限制我们使用PAP身份验证。
+  第二种方法是读取userPassword属性之类的属性，并允许其他模块将其用作“已知的正确密码”。如果所需属性可读且格式正确，则允许使用其他身份验证协议，如CHAP和MS-CHAP。不幸的是，这是一种安全风险，并且所有LDAP服务器都不支持。

## 活动目录

Active Directory集成取决于Samba的Winbind组件。当Winbind正确运行时，它使我们能够使用ntlm_auth二进制文件对域进行身份验证。 exec模块（rlm_exec）使用ntlm_auth二进制文件进行PAP身份验证，使用mschap模块（rlm_mschap）进行MS-CHAP身份验证。

到目前为止，所有章节都涵盖了身份验证的各个方面以及一些授权。在下一章中，我们将学习有关会计的所有知识。

## 流行测验 - 用户存储
1. 您继承了具有现有MySQL用户存储的FreeRADIUS服务器。之前的所有者没有使用radgroucheck和radgroupreply表。你想使用它们并进行测试运行，但似乎没有任何改变。问题是什么？
2. 在Ubuntu服务器上，您宁愿运行PostgreSQL而不是MySQL。您试图查看是否有任何可用于PostgresSQL的示例数据库结构，但只有mysql列在/ etc / freeradius / sql目录下。为什么是这样？
3. 您的经理询问您是否可以使用数据库而不是文本文件进行身份验证。他的问题在技术上是否正确？
4. 您有一个slapd服务器，它以明文形式存储userPassword属性。 如何在保留明文密码的同时使其更安全？
5. 有人告诉您，在将Novell eDirectory用作用户存储之前，需要在Novell eDirectory上启用通用密码。 这可能是真的吗？
6. 您已将FreeRADIUS服务器配置为对PAP和MS-CHAP使用ntlm_auth。 PAP工作正常，但MS-CHAP失败并显示“确保/ var / run / samba / winbindd_privileged上的权限设置正确”的消息。 我们该如何解决这个问题？
7. 您刚刚完成了本章中有关如何将Active Directory包含为用户存储的部分。 它就像有魔力。 在夜间出现重大电力故障; 现在重启后无所作为。 你应该在哪里开始排除故障？