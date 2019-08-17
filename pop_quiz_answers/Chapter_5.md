# 用户存储
1. sql.conf中的read_groups指令的值可能由前一个管理员设置为no;将其更改为yes将激活所有用户的组表读取。
2. 需要先安装freeradius-postgresql包。该软件包包含所需的设置文件以及PostgreSQL特定的FreeRADIUS模块。
3. 不，您不对SQL数据库或文本文件进行身份验证，而是使用它们来存储凭据。然后，验证模块使用存储在文本文件或SQL数据库中的数据进行密码验证。 （如果他是非技术人员只是告诉他没问题，就行了。）
4. 通过安全连接连接到服务器，并向目录添加访问控制以限制对userPassword属性的访问。
5. 不，这不是真的！您仍然可以使用“绑定为用户”方法，这将限制您使用PAP身份验证。nspmPassword属性在启用通用密码时可用，允许MS-CHAP身份验证，因为nspmPassword的格式允许FreeRADIUS以明文形式获取用户密码（请记住，您必须使用SSL/TLS连接到LDAP服务器）并且您绑定的用户需要有足够的权限来读取此属性）。
6. 确保运行FreeRADIUS的用户或组具有对此目录的读访问权。
7. 确认重启后所有服务都已启动，尤其是smbd，nmbd和winbind。