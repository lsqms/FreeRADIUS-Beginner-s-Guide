# Active Directory作为用户存储
虽然Microsoft Active Directory包含LDAP服务器，但使用LDAP不包括MS-CHAP身份验证。将Active Directory用作用户存储可以使用PAP和MS-CHAP身份验证。

配置FreeRADIUS以将Active Directory用作用户存储包含两个主要的代理：

+ 配置Samba服务器并将其加入Active Directory域。
+ 配置FreeRADIUS以调用ntlm_auth二进制文件来验证用户身份。

Samba是适用于Linux和UNIX的标准Windows互操作性程序套件。这是一个非常成熟的项目，正在积极开发（http://www.samba.org/）。

在本练习中，我们将Samba服务器加入Active Directory域。此Samba服务器将显示为Active Directory的另一个Windows服务器。 Samba服务器包含一个名为Winbind的组件，用于解决统一登录问题（http://www.samba.org/samba/docs/man/Samba-HOWTO-Collection/winbind.html）。

我们将使用Winbind允许在Active Directory中定义的用户在Linux服务器上进行身份验证。
