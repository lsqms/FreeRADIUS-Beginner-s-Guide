# 行动时刻 - 将FreeRADIUS连接到LDAP

以下部分将向您展示如何将FreeRADIUS连接到LDAP。

##安装slapd
确保在Linux服务器上安装了slapd。下表可用作在本书中讨论的三个发行版中的每一个上安装slapd的指南：

|发行版|用于安装MySQL服务器的命令|
|:-|:-:|
|CentOS|yum install openldap-servers openldap-clients|
|SUSE|zypper install openldap2 openldap2-client|
|Ubuntu|sudo apt-get install slapd ldap-utils|

安装slapd后我们需要配置它。

## 配置slapd
要使slapd启动并运行，我们将使用最低限度的slapd.conf文件。 这仅用于演示目的; 不要在生产环境中使用它。

slapd的正确配置超出了本书的范围。 本章仅帮助配置一个非常基本的slapd LDAP服务器。
### CentOS
请按照以下步骤在CentOS上配置slapd：

1. 备份原始的slapd.conf文件：
`cp /etc/openldap/slapd.conf /etc/openldap/slapd.conf.orig`
2. 编辑slapd.conf的内容，使其包含以下内容：
```
include /etc/openldap/schema/core.schema
include /etc/openldap/schema/cosine.schema
include /etc/openldap/schema/inetorgperson.schema
include /etc/openldap/schema/nis.schema
pidfile /var/run/openldap/slapd.pid
argsfile /var/run/openldap/slapd.args
database bdb
suffix "dc=my-domain,dc=com"checkpoint 1024 5
cachesize 10000
rootdn "cn=Manager,dc=my-domain,dc=com"
rootpw secret
directory /var/lib/ldap
```
3. 使用以下命令启动slapd：
/etc/init.d/ldap start
4. 通过观察此命令的输出确保slapd正在运行：
ps aux | grep slapd
5. 使用以下命令确保slapd在重新启动后启动：
/sbin/chkconfig ldap on

### SUSE
请按照以下步骤在SUSE上配置slapd：

1. 备份原始的slapd.conf文件：
`cp /etc/openldap/slapd.conf /etc/openldap/slapd.conf.orig`
2. 编辑slapd.conf的内容以包含以下内容：
```
include /etc/openldap/schema/core.schema
include /etc/openldap/schema/cosine.schema
include /etc/openldap/schema/inetorgperson.schema
include /etc/openldap/schema/nis.schema
pidfile /var/run/slapd/slapd.pid
argsfile /var/run/slapd/slapd.args
database bdb
suffix "dc=my-domain,dc=com"
checkpoint 1024 5
cachesize 10000
rootdn "cn=Manager,dc=my-domain,dc=com"
rootpw secret
directory /var/lib/ldap
```
3. 使用以下命令启动slapd：
`/etc/init.d/ldap start`
4. 通过观察以下命令的输出确保slapd正在运行：
`ps aux | grep slapd`
5. 使用以下命令确保slapd在重新启动后启动：
`/sbin/chkconfig -a ldap`

### Ubuntu
版本2.3及更高版本的slapd引入了一种处理配置的新方法。 使用配置文件（slapd.conf）的旧方法现在替换为允许您在服务器运行时调整和配置服务器的替代方法。 然后，这些设置成为服务器的一部分，无需重新启动。

Ubuntu采取了plunge，是第一批采用这种配置slapd的新方法的发行版之一。 幸运的是，我们仍然可以使用更简单的slapd.conf文件。 为了在三个发行版中保持统一，我们将Ubuntu的slapd配置恢复为使用slapd.conf。

1. 要恢复到slapd.conf，请编辑/ etc / default / slapd文件并将SLAPD_CONF =条目更改为SLAPD_CONF = / etc / ldap / slapd.conf。
2. 在/ etc / ldap目录中创建一个名为slapd.conf的文件，其中包含以下内容：
```
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/inetorgperson.schema
include /etc/ldap/schema/nis.schema
pidfile /var/run/slapd/slapd.pid
argsfile /var/run/slapd/slapd.args
modulepath /usr/lib/ldap
moduleload back_bdb.la
database bdb
suffix "dc=my-domain,dc=com"
checkpoint 1024 5
cachesize 10000
rootdn "cn=Manager,dc=my-domain,dc=com"
rootpw secret
directory /var/lib/ldap
```
3. 确保此文件归openldap用户和组所有：
`sudo chown openldap。 /etc/ldap/slapd.conf`
4. 使用以下命令启动slapd：
`sudo /etc/init.d/slapd start`
5. 通过观察以下命令的输出确保slapd正在运行：
`ps aux | grep slapd`
6. 使用以下命令确保slapd在重新启动后启动：
`sudo update-rc.d slapd enable`

让我们看一下slapd.conf中的一些重要配置项：

+ 包含行用于获取模式定义。 （我们将在下一部分中介绍模式。）
+ 我们还指定了一个pidfile。该文件包含slapd的进程ID。
+ argsfile指定包含slapd参数的文件。
+ 数据库指定我们将用于slapd的后端。我们使用Berkeley DB（bdb）。还有其他可用，但我们将使用最常见的。
+ cachesize和checkpoint条目将应用于数据库。 cachesize指定内存缓存的大小（我们指定了1000个条目，这是默认值）。检查点指定两个值。 1024是千字节值，5是分钟值。无论哪个，第一个将导致数据库通过其缓存缓存并写入磁盘。
+ 后缀指定数据库中的所有条目将包含的公共根。 DN以“dc = my-omain，dc = com”结尾的查询将传递给此后端。
+ rootdn和rootpw就像是管理员用户的密码。此用户不会受到配置中指定的任何访问控制或限制。

我们现在应该配置并运行一个基本的slapd服务器。本节的其余部分将在各个发行版中很常见。

## 添加radiusProfile架构

必须定义LDAP中使用的对象类，以便LDAP知道对象类可以包含的结构和属性。 LDAP中提到的对象类与面向对象编程（OOP）中使用的对象类无关。 LDAP目录中的每个条目都应属于至少一个对象类。条目的对象类将指示条目应具有的属性以及条目可具有的那些属性。一个条目可以属于许多对象类。

对象类在.schema文本文件中定义，位于schema目录下。它们的方式类似于FreeRADIUS使用的字典文件。您还可以定义和包含自己的对象类。

Schema文件也需要包含在slapd.conf文件中，以便slapd了解它们。请参阅示例slapd.conf文件以找到您的分布式上的架构目录。

FreeRADIUS包含radiusProfile对象类的模式文件。这必须包含在slapd.conf文件中，并且必须重新启动slapd。当您包含此文件时，请注意以下两点：

+ 每个分布上该文件的位置不同。
+ 该文件名为openldap.schema，但OpenLDAP中已存在名为openldap.schema的模式文件。将freeRADIUS的openldap.schema文件复制到slapd架构目录时，将其重命名为freeradius.schema。

下表可用于在每个发行版上找到openldap.schema文件：
|发行版|openldap.schema的位置|
|:-|:-:|
|CentOS|/usr/share/doc/freeradius-<version>/examples/openldap.schema|
|SUSE|/usr/share/doc/packages/freeradius-server-doc/examples/openldap.schema|
|Ubuntu|/usr/share/doc/freeradius/examples/openldap.schema|

我们以Ubuntu为例：

1. 将openldap.schema文件作为freeradius.schema复制到/etc/ldap：
`sudo cp /usr/share/doc/freeradius/examples/openldap.schema /etc/ldap/schema/freeradius.schema`
2. 编辑slapd.conf文件以包含freeradius.schama文件：
`include /etc/ldap/schema/freeradius.schema`
3. 使用以下命令重新启动slapd：
`sudo /etc/init.d/slapd restart`

在slapd.conf中包含freeradius.schema文件允许我们将RADIUS属性存储在LDAP目录中。然后FreeRADIUS可以使用这些属性。

>在LDAP目录中，不必具有radiusProfile架构扩展。它允许强大的配置，特别是在FreeRADIUS的授权部分。

## 填充LDAP目录
LDAP具有称为LDAP数据交换格式（LDIF）的标准格式，用于添加或修改目录的数据。维基百科将LDIF描述如下：
`http://en.wikipedia.org/wiki/LDAP_Data_Interchange_Format`
>LDIF是标准纯文本数据交换格式，用于表示LDAP目录内容和更新请求。 LDIF将目录内容作为一组记录传送，每个对象（或条目）记录一条记录。它表示更新请求，例如添加，修改，删除和重命名，作为一组记录，每个更新请求一条记录。

我们将使用此LDIF格式为一些用户创建以下结构：
该图形显示了具有以下内容的树结构：

+ 树的根是名为My Domain Inc.的组织。它属于dcObject和组织对象类。
+ 到树的根是一个名为radius的组织单元，它包含三个子组织单元：用户，配置文件和管理员。组织单位就像一个文件夹;它属于organizationalunit对象类。
+ 用户组织单位包含三个用户：student1，student2和student3。这些用户属于person和radiusProfile对象类。
+ 配置文件组织单元包含两个用户模板：学生和教师。这些模板也属于person和radiusProfile对象类，但它们将充当用户组织单位下的用户可以属于的组。
+ 管理员组织单位包含FreeRADIUS将用于绑定到LDAP目录的用户。可以微调此用户的权限以获得最大安全性。此用户不需要属于radiusProfile对象类。

以下是LDIF文件的内容：
```
dn: dc=my-domain,dc=com
dc: my-domain
description: Tutorial for FreeRADIUS
objectClass: dcObject
objectClass: organization
o: My Domain Inc
dn: ou=radius,dc=my-domain,dc=com
objectclass: organizationalunit
ou: radius
dn: ou=profiles,ou=radius,dc=my-domain,dc=com
objectclass: organizationalunit
ou: profiles
dn: ou=users,ou=radius,dc=my-domain,dc=com
objectclass: organizationalunit
ou: users
dn: ou=admins,ou=radius,dc=my-domain,dc=com
objectclass: organizationalunit
ou: admins
dn: cn=students,ou=profiles,ou=radius,dc=my-domain,dc=com
objectclass: radiusProfile
objectClass: person
cn: students
sn: students
radiusSessionTimeout: 900
radiusReplyItem: ChilliSpot-Bandwidth-Max-Up = "393216"
radiusReplyItem: ChilliSpot-Bandwidth-Max-Down = "393216"
radiusCheckItem: ChilliSpot-Version == "1.0"
radiusReplyMessage: "Good day student"
dn: cn=teachers,ou=profiles,ou=radius,dc=my-domain,dc=com
objectclass: radiusProfile
objectClass: person
cn: teachers
sn: teachers
radiusSessionTimeout: 3600
radiusReplyItem: ChilliSpot-Bandwidth-Max-Up = "1048576"
radiusReplyItem: ChilliSpot-Bandwidth-Max-Down = "1048576"
radiusCheckItem: ChilliSpot-Version == "2.0"
radiusReplyMessage: "Good day teacher"
dn: cn=student1,ou=users,ou=radius,dc=my-domain,dc=com
objectclass: radiusProfile
objectClass: person
cn: student1
sn: student1
userPassword: student1
description: Test user with cleartext password student1
radiusGroupName: students
dn: cn=student2,ou=users,ou=radius,dc=my-domain,dc=com
objectclass: radiusProfile
objectClass: person
cn: student2
sn: student2
userPassword: {CRYPT}saCsqST0rezXE
description: Test user with CRYPT password student2
radiusGroupName: students
radiusGroupName: teachers
dn: cn=student3,ou=users,ou=radius,dc=my-domain,dc=com
objectclass: radiusProfile
objectClass: person
cn: student3
sn: student3
userPassword: {SHA}Mr5L7b06hTlQOpu75y+dhJVq/6E=
description: Test user with SHA password student3
radiusGroupName: students
radiusGroupName: disabled
dn: cn=binduser,ou=admins,ou=radius,dc=my-domain,dc=com
objectclass: person
sn: freeradius
cn: binduser
userPassword: binduser
```

执行以下操作以填充目录：

1. 创建一个名为4088_05_ldap.ldif的文件，其中包含上面的LDIF文本作为其内容。 此文件的最佳位置可能是您的主目录。
2. 使用以下命令将其添加到LDAP目录：
`ldapadd -x -D'cn = Manager，dc = my-domain，dc = com'-w secret -f 4088_05_ldap.ldif`

现在，LDAP服务器的所有准备工作都已完成。 接下来的步骤将准备FreeRADIUS将此目录用作用户存储。

## 安装FreeRADIUS的LDAP包
CentOS和Ubuntu有单独的FreeRADIUS包，其中包含ldap模块（rlm_ldap）。 使用下表作为安装指南。

|发行版|用于安装FreeRADIUS的LDAP包的命令|
|:-|:-:|
|CentOS|yum install freeradius2-ldap
yum --nogpgcheck install freeradiusldap-2.1.10-1.i386.rpm
(if built from source)|
|Ubuntu|sudo apt-get install freeradius-ldap
sudo dpkg -i freeradius-ldap_2.x.y+git_i386.deb
(if built from source)|

SUSE已将rlm_ldap模块作为freeradius-server软件包的一部分包含在内。

## 配置ldap模块
要配置ldap模块，您必须编辑modules目录下的ldap配置文件。 更改以下指令：
|指令|值|
|:-|:-:|
|server|127.0.0.1|
|identty|cn=binduser,ou=admins,ou=radius,dc=my-domain,dc=com|
|password|binduser|
|basedn|ou=users,ou=radius,dc=my-domain,dc=com|
|filter|(cn=%{%{Stripped-User-Name}:-%{User-Name}})|

值必须使用引号，例如：
`server =“127.0.0.1”`
不要更改文件的其余部分，默认值将正常工作。

## 测试LDAP用户存储

实际情况非常接近，我们只需要在虚拟服务器的授权和身份验证部分中包含LDAP并对其进行测试：

1. 编辑FreeRADIUS配置目录中的站点启用/默认文件。 在授权部分下，取消注释ldap。
2. 在验证部分取消注释：
```
Auth-Type LDAP {
	LDAP
}
```
3. 在调试模式下重新启动FreeRADIUS。
4. 尝试使用radtest程序作为student1进行身份验证：
`radtest student1 student1 127.0.0.1 100 testing123`
5. 观察FreeRADIUS服务器的调试输出。

虽然需要一些准备，但我们最终在FreeRADIUS中使用了LDAP用户存储。

## 刚刚发生了什么？
FreeRADIUS的ldap模块已连接到slapd以授权和验证student1。
在重新访问FreeRADIUS服务器的调试输出时，让我们看一些有趣的观点。
## 绑定为用户
ldap模块的配置导致以下情况发生：

+ LDAP包含在虚拟服务器的授权部分中。这导致ldap模块使用我们在ldap配置文件中指定的标识绑定到slapd。
+ 查询slapd以检查student1是否存在。此查询是通过使用ldap配置文件中指定的basedn值和过滤器值来制定的。
+ 查询成功了。这导致ldap模块：
	+ 在检查项列表中添加Ldap-UserDn内部属性
	+ 设置Auth-Type = LDAP
	+ 返回OK
+ 当FreeRADIUS进入认证部分时，它使用LDAP来执行身份验证。通过尝试使用Ldap-UserDn属性和Access-Request中提供的密码进行绑定来完成身份验证。
+ 如果绑定成功，则返回Access-Accept数据包。
+ 如果绑定不成功，则返回Access-Reject数据包。

>总结过程：授权部分搜索用户，找到它，添加Ldap-UserDn检查属性，并更改Auth-Type = LDAP。 认证部分使用Ldap-UserDn和Access-Request数据包中提供的密码绑定到LDAP服务器。

上面的过程看起来很简单，但是如果你不熟悉基础知识的话，它很容易被破坏。 FreeRADIUS配置中的黄金法则是尽可能改变。

>小心旧文档！不要使用旧的或不可靠的源来配置FreeRADIUS和LDAP。如果文档指示您将非协议整数翻译更改或添加到字典文件中，那么您就处于危险之中！

与系统用户存储一样，只有PAP认证协议才有效。 CHAP和MS-CHAP不起作用。

## LDAP的高级用法
ldap模块的配置使其作为用户绑定以验证凭据。以用户身份绑定对于典型的身份验证方案就足够了，并且应该适用于大多数LDAP服务器。

但是，我们可以做更多的事情，特别是当LDAP服务器具有radiusProfile架构扩展时。如果用户属于radiusProfile对象类，我们可以为LDAP目录中的指定用户指定AVP检查或回复属性。这类似于将其存储在用户文件或SQL数据库中。

此外，如果LDAP服务器以明文形式存储userPassword属性，我们甚至可以使用LDAP服务器，其方式与用户文件或SQL数据库大致相同。这不一定要求用户属于radiusProfile对象，并且可配置为指定应将哪个LDAP属性映射到userPassword AVP。

## 试一试 - 探索LDAP的高级使用
在编写slapd期间，我们包含了radiusProfile对象类的模式文件。这扩展了服务器的架构。 radiusProfile对象类允许在LDAP对象中包含检查和回复AVP。

遗憾的是，这不是直接的。 RADIUS AVP的名称与LDAP属性的名称不匹配。要将一个映射到另一个，FreeRADIUS使用ldap.attrmap文件。在此文件中，您可以看到RADIUS AVP及其相应的LDAP属性名称。让我们看一下文件中的一行：
`checkItem Auth-Type radiusAuthType`
这指定RADIUS Auth-Type AVP（用作检查项）映射到LDAP radiusAuthType属性。

并非所有RADIUS AVP都列在此文件中。但是，该文件还列出了特殊的radiusProfile属性radiusCheckItem和radiusReplyItem。这两个LDAP属性允许您指定属性映射中未指定的任何其他RADIUS AVP。在我们的LDIF中，我们使用这些属性来指定一些AVP。以下是我们如何指定值为2.0的ChilliSpot-Version AVP：
`radiusCheckItem：ChilliSpot-Version ==“2.0”`
## Ldap-Group和User-Profle AVP
Ldap-Group内部AVP用于指定组检查。我们将在用户文件中指定它，尽管它也可以在其他模块中指定。

用户配置文件内部AVP可以包含DN而不是其值的普通文本字符串。发生这种情况时，会导致ldap模块在授权期间查询LDAP目录中的DN。查询返回的radiusCheckItems和radiusReplyItems将用于创建用户的配置文件。

Ldap-Group和User-Profile通常配对在一起。首先进行LDAP搜索以检查用户是否属于Ldap-Group。如果为true，则指定指定的用户配置文件。如果不为true，则不分配指定的用户配置文件。
让我们来利用它：

1. 编辑用户文件并在底部添加以下内容：
```
DEFAULT Ldap-Group == disabled, Auth-Type := Reject
Reply-Message = "Account disabled"
DEFAULT Ldap-Group == teachers, User-Profile := "cn=teachers,ou
=profiles,ou=radius,dc=my-domain,dc=com"
Fall-Through = no
DEFAULT Ldap-Group == students, User-Profile := "cn=students,ou
=profiles,ou=radius,dc=my-domain,dc=com"
Fall-Through = no
```
2. 编辑ldap模块配置文件并更改以下部分：
```
groupname_attribute = cn
groupmembership_filter = "(|(&(objectClass=Gro.....
groupmembership_attribute = radiusGroupName
```
更改为
```
groupname_attribute = radiusGroupName
groupmembership_filter = "(cn=%{%{Stripped-User-Name}:-%{UserName}})"
groupmembership_attribute = radiusGroupName
```
3. 在调试模式下重新启动FreeRADIUS并验证student1，student2和student3每次观察反馈。

让我们看一些重点：

+ 将DEFAULT条目添加到users文件时，添加在顶部设置AuthType：= Reject的条目。
+ 接下来应该遵循更多特权群体，以最少特权群体结束。
+ 这种安排可以将student2（Martin Prince）分配给教师和学生组。
+ 指定Ldap-Group时，它会使文件模块使用ldap模块通过检查LDAP用户的radiusGroupName属性来确定用户是否属于该组。
+ 如果用户是Ldap组的一部分，则将为用户分配用户配置文件。指定为DN的User-Profile会导致ldap模块在授权期间搜索DN：
`[ldap]在cn = teacher中执行搜索，ou = profiles，ou = radius，dc = my-domain，dc = com，带过滤器（objectclass = radiusprofile）`
+ 然后使用搜索的返回值来构建具有检查和回复AVP的用户简档。

下一部分将介绍从LDAP目录中检索密码的方法。

## 从LDAP读取密码
ldap模块可以配置为直接从LDAP服务器读取用户密码，然后将此值作为“已知正确密码”传递给身份验证部分中的其他模块。这使我们能够绕过ldap模块尝试与Ldap-UserDn属性绑定以验证凭据的过程。

为此，我们需要执行以下操作：

1. 编辑ldap模块的配置文件并添加auto_header = yes指令。这将允许pap模块找出密码的哈希（如果存在）。
2. 编辑ldap模块的配置文件并取消注释以下行：
`password_attribute = userPassword`
这指定哪个LDAP属性包含用户的密码。
3. 编辑ldap模块的配置文件，并确保设置以下内容：
`set_auth_type = no`
这可以防止ldap模块设置Auth-Type：= LDAP。
4. 将以下行添加到ldap.attrmap文件中：
`checkItem Cleartext-Password userPassword`
这将返回userPassword LDAP属性为Cleartext-Password AVP。
5. 在调试模式下重新启动FreeRADIUS并尝试使用student1，student2和student3进行身份验证，观察每个密码的不同密码哈希的反馈。
6. 如果您使用的radtest程序支持-t开关（FreeRADIUS版本2.1.10及更高版本），您还可以在不同的密码哈希值上测试CHAP和MS-CHAP身份验证的结果。 CHAP和MS-CHAP应仅适用于student1，而不适用于student2和student3。

从LDAP读取密码而不是绑定到身份验证具有一些优点：
+ 如果密码在LDAP中以明文形式存储，则允许我们使用CHAP和MS-CHAP身份验证协议。
+ 它更快，因为不需要将用户绑定到LDAP。
+ 如果LDAP服务器的userPassword上的用户已加密但sambaLmPassword和sambaNtPassword属性存在且具有相同的值，则我们应该能够使用MS-CHAP。这可以在典型的Samba服务器中找到，slapd作为后端。

但是，有一些事情需要注意。他们之中有一些是:
+ 安全;通过网络传输密码时使用安全的LDAP连接，尤其是明文密码。
+ 并非所有LDAP服务器都支持读取userPassword属性，因为它存在安全风险。如果您决定采用这种方式，请对LDAP服务器上的安全性进行微调，使其非常严格，以避免在黑客窃取您的所有密码时出现这种可怕的意外。还要注意可能易受LDAP注入攻击的Web管理软件的使用。

您通常将绑定用作Novell的eDirectory和Microsoft的Active Directory的用户方法。如果您可以从LDAP获取明文密码并且想要使用CHAP和MSCHAP协议，则将读取userPassword属性。

> Novell的eDirectory包含通用密码（UP）功能，允许FreeRADIUS从目录中提取用户的明文密码。但是，这需要在eDirectory上进行一些特定的配置调整

关于LDAP的这一部分已经涵盖了很多内容。 这里一些主题没有讨论，但是ldap配置文件和Wiki页面中的注释应该是非常有用的。

下一节将帮助您将Actve Directory用作FreeRADIUS中的用户存储。