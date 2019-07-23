# 系统用户

位于运行FreeRADIUS的服务器上的系统用户可以用作用户存储。
系统用户通常与/etc/password，/etc/shadow和/etc/group文件相关联。

Linux机器也可以使用其他的方法，如NIS和LDAP，这使得系统用户的位置更加集中。然而，本节将重点介绍在服务器上使用本地定义的系统用户。