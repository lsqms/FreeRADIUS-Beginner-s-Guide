
# 行动时刻 - 将FreeRADIUS连接到Active Directory

以下部分将演示如何将FreeRADIUS连接到Microsoft Active Directory。

## 安装Samba

确保在Linux服务器上安装了Samba和Winbind。下表可用作在本书中讨论的三个发行版中的每一个上安装Samba和Winbind的指南：
|发行版|用于安装Samba服务器的命令|
|:-|:-:|
|CentOS|yum install samba|
|SUSE|zypper install samba samba-winbind|
|Ubuntu|sudo apt-get install samba winbind|


配置Samba
本节应作为指导。 它假设一个名为fr.com的工作Active Directory域，我们将加入该域。 下表列出了设置的一些细节：

|设置/项|评论|
|:-|:-:|
|Actve Directory Domain|Name: fr.com
Domain Controller: dc.fr.com
Domain Controller IP: 192.168.1.250
DNS server for domain: 192.168.1.250|
|Linux /etc/resolv.conf fle|This fle is used to defne the DNS servers on Linux. It is
important to use the Actve Directory as nameserver and
also specify the domain and search components.
nameserver 192.168.1.250
domain fr.com
search fr.com|

Samba和Winbind是通过/etc/samba/smb.conf文件配置的。 以下可以作为示例：
```
[global]
workgroup = FR
realm = FR.COM
preferred master = no
server string = Ubuntu FreeRADIUS Test Machine
security = ADS
encrypt passwords = yes
log level = 3
log file = /var/log/samba/%m
max log size = 50
printcap name = cups
printing = cups
winbind enum users = Yes
winbind enum groups = Yes
winbind use default domain = Yes
winbind nested groups = Yes
winbind separator = +
idmap uid = 600-20000
idmap gid = 600-20000
template shell = /bin/bash
```

## 加入域
要将Samba服务器加入Active Directory域，请按照下列步骤操作：

1. 您需要域管理员的密码才能加入域：
`net ads join -U Administrator`
2. 确认域用户现在是否可用于Samba服务器。
以下命令应列出它们：
`wbinfo -u`
3. 使用wbinfo命令测试对域进行身份验证。 Billy是一个密码为passbilly的域用户：
`wbinfo -a billy％passbilly`
4. FreeRADIUS将使用ntlm_auth二进制文件来测试针对域的身份验证。使用以下命令验证billy：
`ntlm_auth --request-lm-key --domain = FR.COM --username = billy--password = passbilly`

加入Active Directory的Sometmes可能很麻烦。以下是一些要检查的事项：

+ 这两个系统的时间是一样的吗？ Kerberos不喜欢大的差异。
+ 尝试通过将系统时间与外部NTP服务器同步，或使用Linux计算机上设置的净时间来解决此问题。
+ DNS是否正常工作，并且两个系统是否可以通过主机名和FQDN相互ping通？
+ 一些可用的教程包括Kerberos的配置，但在我的服务器上它只是在没有任何额外配置的情况下工作。因人而异。
+ 确保smbd，nmbd和winbind服务正在运行，尤其是重启后。这将要求您确认启动脚本已激活。

作为最后的活动，我们需要为运行FreeRADIUS服务器的用户设置权限。不幸的是，这样做的方法在每个发行版上都是不同的。我们来看看每一个。

### CentOS
该目录是/var/cache/samba/winbindd_privileged。使用以下命令将组所有权授予radiusd组：
`chgrp radiusd /var/cache/samba/winbindd_privileged`
### SUSE
该目录是/var/lib/samba/winbindd_privileged。使用以下命令将组所有权授予radiusd组：
`chgrp radiusd /var/lib/samba/winbindd_privileged`
### Ubuntu
该目录是/ var/run/samba/winbindd_privileged。该目录由名为winbind_priv的组拥有。使用以下命令使freerad用户成为该组的成员：
`sudo adduser freerad winbindd_priv`

这完成了第一个主要活动。现在我们可以配置FreeRADIUS了。

>请记住，此部分是教科书示例。在现实世界中，加入域并不总是那么简单。这是因为并非所有活动目录都配置相同，并且并非所有网络看起来都相同。 Samba网站包含大量文档，可在事情无法正常工作时提供帮助。

在进一步继续进行之前，您必须能够使用ntlm_auth命令测试身份验证。

## FreeRADIUS和ntlm_auth 
FreeRADIUS可以通过两种方式使用ntlm_auth二进制文件：

+ 对于PAP身份验证，请包含一个调用ntlm_auth二进制文件的exec部分。
+ 对于MS-CHAP身份验证，请修改MS-CHAP模块的配置文件以指定ntlm_auth进行身份验证。

## PAP验证
FreeRADIUS在modules目录下包含一个名为ntlm_auth的文本文件。该文件包含一个名为ntlm_auth的exec部分：
```
exec ntlm_auth {
	wait = yes
	program =“/ path / to / ntlm_auth --request-nt-key --domain = MYDOMAIN --username =％{mschap：User-Name} --password =％{User-Password}”
}
```
exec部分的名称可以是任何名称，但命名为ntlm_auth有助于识别将由exec模块调用的程序。修改程序指令以反映您的设置：
`“/ usr / bin / ntlm_auth --request-nt-key --domain = FR.COM --username =％{mschap：User-Name} --password =％{User-Password}“ `
要配置FreeRADIUS以使用带有PAP的Active Directory用户存储，请执行以下操作：

1. 编辑已启用站点/默认文件，并在authorize→pap行下添加此部分：
```
if（！control：Auth-Type）{
	update control {
		Auth-Type =“ntlm_auth”
	}
}
```
此条目使用unlang，这在授权章节中有详细介绍。表达式表明如果上述模块都没有设置Auth-Type的值，则应将其设置为ntlm_auth。
2. 我们还需要将一个名为NTLM_AUTH的Auth-Type添加到sites-enabled/default文件的authenticate部分：
```
Auth-Type NTLM_AUTH {
	ntlm_auth
}
```
这将创建一个名为NTLM_AUTH的Auth-Type，它使用ntlm_auth exec部分来执行身份验证。

Auth-Type的值不区分大小写，它允许我们遵循约定将它以大写形式存在，尽管authorize部分中的unlang条目是小写的。

在调试模式下重新启动FreeRADIUS并尝试使用Active Directory用户进行身份验证。

反馈将指示如何设置Auth-Type并调用ntlm_auth exec部分来执行身份验证：
```
++? if (!control:Auth-Type)
? Evaluating !(control:Auth-Type) -> TRUE
++? if (!control:Auth-Type) -> TRUE
++- entering if (!control:Auth-Type) {...}
+++[control] returns noop
++- if (!control:Auth-Type) returns noop
Found Auth-Type = NTLM_AUTH
# Executing group from file /etc/raddb/sites-enabled/default
+- entering group NTLM_AUTH {...}
[ntlm_auth] expand: --username=%{mschap:User-Name} ->
--username=billy
[ntlm_auth] expand: --password=%{User-Password} ->
--password=passbilly
Exec-Program output: NT_STATUS_OK: Success (0x0)
Exec-Program-Wait: plaintext: NT_STATUS_OK: Success (0x0)
Exec-Program: returned: 0
++[ntlm_auth] returns ok
```

## MS-CHAP身份验证
在MS-CHAP身份验证期间，mschap模块提取用户提供的NT密码哈希。 为了验证它是否正确，模块可以执行以下三种操作之一：

+ 获取用户的Cleartext-Password并从中生成NT-Password哈希值，然后将其与MS-CHAP中的哈希值进行比较。
+ 如果用户的密码已存储为NT-Password AVP，请将其与MS-CHAP中的密码进行比较。
+ 配置mschap模块的ntlm_auth指令。 这将使用ntlm_auth二进制文件对Active Directory域进行身份验证。

前两章已经讨论了前两种方法。 配置ntlm_auth指令很简单。 只需按以下步骤操作：

1. 编辑modules / mschap文件并取消注释以下行：
`ntlm_auth = "/path/to/ntlm_auth --request-nt-key --username=%{mschap:User-Name} --challenge=%{mschap:Challenge:-00} –nt-response=%{mschap:NT-Response:-00}"`
2. 将/path/to/ntlm_auth替换为服务器上ntlm_auth的实际路径（/usr/bin/ ntlm_auth）。
3. 这将告诉mschap模块使用ntlm_auth而不是其他两种方法进行凭证验证。
4. 如果仍有其他用户要使用mschap模块进行身份验证，则必须确保它们具有检查属性MSCHAP-Use-NTLM-Auth：=No。此外，alice条目现在如下所示：
`“alice”Cleartext-Password：=“passme”，MS-CHAP-Use-NTLM-Auth：= No`
5. MS-CHAP-Use-NTLM-Auth是用于控制mschap模块行为的内部AVP。
6. 在调试模式下重新启动FreeRADIUS。
7. 尝试使用MS-CHAP协议与Active Directory用户进行身份验证。

> 版本2.1.10和FreeRADIUS向上允许您使用radtest（-t mschap）指定MS-CHAP身份验证。如果正在运行的FreeRADIUS服务器是早期版本，请在其他地方进行最新安装，并使用较新的radtest远程进行身份验证。另一种方法是使用JRadius Simulator程序（http://coova.org/JRadius/Simulator）。

