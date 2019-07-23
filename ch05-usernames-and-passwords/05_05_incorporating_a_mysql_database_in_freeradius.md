# 行动时刻-在FreeRADIUS中加入MySQL数据库

我们假设在尚未部署FreeRADIUS的系统上尚未安装MySQL。 我们将首先安装然后配置MySQL，以便它可用于FreeRADIUS。

## 安装MySQL

确保Linux服务器上安装了MySQL服务器。 下表可用作在本书中讨论的三种发行版中的每一种上安装MySQL的指南：

|发行版|用于安装MySQL服务器的命令|
|:-|:-:|
|CentOS|yum install mysql-server|
|SUSE|zypper install mysql|
|Ubuntu|sudo apt-get install mysql-server|

> MySQL服务器有一个名为root的用户，默认情况下本地计算机上没有任何密码。 强烈建议您为此用户提供密码。

请注意每个分布上的以下几点：

+ CentOS：
	+ 可能已经安装了mysql-client软件包。
	+ 在安装MySQL服务器之前，需要为第一个服务器启动它。
	+ 使用命令/etc/init.d/mysqld start。 反馈消息包含有关如何为root用户添加密码的说明。
	+ 使用/sbin/chkconfig mysqld on确保MySQL服务器在重新启动后启动。
+ SUSE：
	+ 在安装MySQL服务器之前，需要为第一个服务器启动它。
	+ 使用rcmysql start命令。 反馈消息包含有关如何为root用户添加密码的说明。
	+ 使用/sbin/chkconfig -a mysql命令确保重启后MySQL服务器启动。

+ Ubuntu:
	+ mysql-server软件包是一个元软件包，它将安装最新版本的MySQL服务器（Ubuntu 10.4上的MySQL 5.1）。
	+ mysql-server软件包使用debconf实用程序进行用户输入。 在安装过程中，系统会要求您在MySQL服务器上为root用户提供密码。
	+ 如果以后需要提供或更改root用户的root用户密码，则可以使用dpkg-reconfigure mysql-server-5.1命令执行此操作。
	+ Ubuntu使用Upstart来启动和停止MySQL。 你可以使用sudo service mysql start命令启动和sudo service mysql stop命令来停止MySQL服务器。 MySQL的启动配置文件位于/etc/init/mysql.conf中。

我们现在可以继续为FreeRADIUS安装MySQL模块（如果需要），并为FreeRADIUS准备一个数据库供我们使用。

## 安装FreeRADIUS的MySQL包

CentOS和Ubuntu有单独的FreeRADIUS包，其中包含MySQL的特定sql模块（rlm_sql_mysql）。 使用下表作为安装指南：

|发行版|用于安装MySQL服务器的命令|
|:-|:-:|
|CentOS|yum install freeradius2-mysql
Or
yum --nogpgcheck install freeradiusmysql-2.1.10-1.i386.rpm
(if built from source)|
|Ubuntu|sudo apt-get install freeradius-mysql
Or
sudo dpkg -i freeradius-mysql_2.x.y+git_i386.
deb
(if built from source)|

> 请记住，您必须安装FreeRADIUS的MySQL软件包，它是已安装的FreeRADIUS构建的一部分。 您无法从源代码构建和安装最新的FreeRADIUS，然后期望软件包管理器引用的旧软件包将安装。 出于这个原因，我们在上表中的两者之间产生了分歧。

SUSE已将rlm_sql_mysql模块作为freeradius-server软件包的一部分包含在内。

## 准备数据库
FreeRADIUS提供所有必需的文件来准备数据库供其使用。

FreeRADIUS配置目录包含一个名为sql的子目录。 在sql子目录下是FreeRADIUS支持的各种数据库的子目录。 如果只有一个MySQL目录，那是因为没有安装FreeRADIUS软件包支持其他数据库。

1. 要创建名为radius的数据库，请发出以下命令：
`mysqladmin -u root -p create radius`
2. 要创建具有radius数据库的正确权限的admin用户，请使用admin.sql文件作为模板，并针对radius数据库运行它。
建议您更改默认值。 使用以下命令：
`mysql -u root -p </etc/raddb/sql/mysql/admin.sql`
3. 使用schema.sql文件，使用以下命令为数据库创建架构：
`mysql -u root -p radius </etc/raddb/sql/mysql/schema.sql`
4. 以测试用户身份将Bob添加到数据库。
```
mysql -u root -p radius
INSERT INTO radcheck (username, attribute, op, value) VALUES
('bob', 'Cleartext-Password', ':=', 'passbob');
INSERT INTO radreply (username, attribute, op, value) VALUES
('bob', 'Reply-Message', '=', 'Hello Bob!');
```

>如果您不熟悉MySQL，可以使用一个名为mysqlshow的方便命令来获取快速信息。 要获取可以使用的数据库列表：
mysqlshow -u root
然后，您可以通过向mysqlshow命令添加参数（例如，mysqlshow -u root radius radcheck）来进一步向下钻取表中的数据库，表和列。 使用此命令确认使用其表创建radius数据库。

## 配置FreeRADIUS

SQL模块打破了FreeRADIUS的传统，模块的配置位于FreeRADIUS配置目录下的modules子目录下。

### 连接信息
位于FreeRADIUS配置目录中的sql.conf文件包含连接到数据库的所有配置选项。如果您使用了默认值，则无需更改此文件中的任何内容。但是，我们鼓励您浏览此文件的内容，以便了解可以指定的各种指令。这也有助于仔细检查并确认前面步骤中使用的值。

### 包括SQL配置
要让FreeRADIUS在启动时包含SQL模块，请取消注释radiusd.conf中的以下行：＃$ INCLUDE sql.conf

### 虚拟服务器
如前所述，每个虚拟服务器都包含主要部分。要将SQL模块用作用户存储，请在sites-enabled / default的authorize部分中取消注释sql行。

>如果您将上一个练习中的unix部分取消注释，请再次禁用它。如果不这样做，将导致FreeRADIUS使用系统用户的详细信息验证bob。

## 测试MySQL用户存储
一切都已配置好并准备好供我们测试。请按照以下步骤测试用户存储：

1. 在调试模式下重启FreeRADIUS。扫描调试输出并检查rlm_sql反馈。这表明包含了SQL模块。
2. 使用radtest程序验证为bob：
radtest bob passbob 127.0.0.1 100 testing123
3. 观察FreeRADIUS服务器的输出，以了解rlm_sql如何处理请求。
4. 
## 刚刚发生了什么？
我们已经配置并添加了一个MySQL数据库，作为FreeRADIUS的用户存储。让我们看看一些兴趣点。

如本章开头所述，FreeRADIUS有两种使用数据存储的方法。使用MySQL数据库，它从数据库中读取用户的信息。然后，该数据可以由诸如pap模块之类的认证模块用于密码验证。

FreeRADIUS不对数据库进行身份验证，而是使用数据库作为存储来保存用户数据。数据库用作用户文件的替代或替代。

> 如果用户的详细信息似乎不是来自MySQL数据库，请确认您是否已在授权部分中禁用了unix模块，并观察调试输出以获取更多信息以查找问题。

## SQL优于文件的优点
与将数据存储在文件中相比，在SQL数据库中存储用户的详细信息具有各种优势。 以下是一些优点：

+ 可扩展：数据库可以位于另一台服务器上，不需要在FreeRADIUS服务器上。
+ 用户友好：有许多基于Web的软件可用于管理数据库中的数据。
+ 灵活：可以在文件中添加或删除用户和属性，而无需重新启动FreeRADIUS。
+ 可管理：可以将用户分配到一个或多个组以管理公共属性。 也可以使用配置文件。
+ 安全：可以使用内置函数对敏感信息进行散列和加密，这些函数通常是SQL数据库引擎的一部分。

## SQL数据库的其他用途
SQL数据库不仅用于存储FreeRADIUS中的用户详细信息。 其他功能包括：

+ 计费：我们可以将用户的计费详细信息写入数据库而不是文件。
+ 使用控制：rlm_sqlcounter模块允许定义各种计数器（基于tme或数据）以跟踪用户的使用情况。
+ NAS设备：client.conf文件中默认定义的NAS设备也可以存储在数据库表中。
+ IP池管理：向数据库添加额外的表允许我们在数据库的帮助下管理IP租约。

计费和使用控制将在本书后面的专门章节中介绍。

## 重复用户
FreeRADIUS允许不同的用户存储共存，但是当在不同的用户存储中定义相同的用户时会发生什么？

这取决于授权部分中列出模块的顺序。验证部分将使用最后一个模块的用户详细信息。

不幸的是，这条规则不适用于所有模块。 unix模块总是将重复用户的详细信息发送到authenticate部分，而不是在authorize部分内的顺序是什么。

使用默认顺序，SQL数据库中定义的用户将“赢取”用户文件中定义的用户。

如果您希望用户文件中定义的用户对SQL数据库中的副本“获胜”，则应该在sql模块中列出文件模块。

## 数据库架构

SQL数据库包含与files模块使用的用户文件相同类型的详细信息。就像用户文件一样，我们有检查和回复项目。这些项目存放在radcheck和radreply表中。

### 组
SQL数据库还允许我们定义组的检查和回复属性。 它们存储在radgroupcheck和radgroupreply表中。

现在可以将用户分配给零个或多个定义的组。 通过radusergroup表分配组。 此表中的条目指定特定组对用户的优先级。 这允许具有较高优先级（较小值）的组中的某些项目值覆盖具有较低优先级（较大值）的组中的项目值。

考虑到这一点，让我们看一些实际的例子。

试一试吧 - 探索组使用
本节介绍SQL数据库的更高级方面。 我们将通过实践练习介绍以下内容：

+ 小组作业
+ 使用Fall-Through内部AVP
+ 使用User-Profile内部AVP进行配置文件分配

### 使用SQL组

在本练习中，我们将向学生组添加bob。 学生组有一个检查属性来测试Access-Request是否包含值为PPP的Framed-Protocol AVP。 如果AVP存在且正确，我们会返回回复AVP：

1. 登录radius MySQL数据库并发出以下SQL命令以创建所需的条目：
```
delete from radcheck;
delete from radreply;
delete from radgroupreply;
delete from radgroupcheck;
delete from radusergroup;
INSERT INTO radcheck (username, attribute, value,op) VALUES
('bob', 'Cleartext-Password', 'passbob',':=');
INSERT INTO radreply (username, attribute, value,op) VALUES
('bob', 'Reply-Message', 'Hello Bob!','=');
INSERT INTO radgroupreply (groupname, attribute, value,op) VALUES
('students', 'Reply-Message', 'Hello PPP protocol!',':=');
INSERT INTO radgroupreply (groupname, attribute, value,op) VALUES
('students', 'Session-Timeout', '900',':=');
www.it-ebooks.infoSources of Usernames and Passwords
[ 98 ]
INSERT INTO radgroupcheck (groupname, attribute, value,op) VALUES
('students', 'Framed-Protocol', 'PPP','==');
INSERT INTO radusergroup (username, groupname, priority) VALUES
('bob', 'students', 10);
```
2. 使用radtest程序验证为bob，但最后添加1。 这将导致Access-Request数据包包含Framed-Protocol = PPP的AVP：
radtest bob passbob 127.0.0.1 100 testing123 1
3. 您应该获得一个Access-Accept数据包，其中Reply-Message为Reply-Message =“Hello PPP protocol！”。
4. 再次作为bob进行身份验证，但这次排除1.您仍然应该获得一个Access-Accept数据包，其中Reply-Message为Reply-Message =“Hello Bob！”。

您可能希望在Framed-Protocol AVP缺失或不同值时拒绝该请求。 Radgroupcheck通过以下方式从radcheck开始工作：

+ 如果在radcheck中定义了check属性，并且该属性不匹配或丢失，则请求失败并返回Access-Reject。
+  如果在radgroupcheck中定义了检查属性并且它不匹配或丢失，则请求通过，但不返回radgroupreply中的回复属性。

此行为使一个用户可以属于多个组，并且根据属性传递的radgroupcheck，该组的回复属性将随回复一起返回。

> 作为对op字段值的提醒，您可以遵循以下规则，直到我们进入授权章节。
> 回复项包含=并检查需要匹配传入AVP的项目==而其他人使用：=。如果您希望回复项覆盖现有项，请使用：=。

Radgroupcheck项目在逻辑上是AND。第一个失败将导致组不返回任何回复属性。

### 控制组的使用

默认情况下，如果用户在radcheck表中存在，则SQL模块会检查是否有分配给用户的组。可以通过两种方式控制此行为：

1. 要全局转换此函数，请在sql.conf文件中将read_groups指令的值设置为no。
2. 要为单个用户再次激活已分配组的检查，请在radreply表中指定Fall-Through = Yes。

首先将read_groups设置为no，试试这个。重新启动FreeRADIUS并作为Bob进行身份验证。即使在向radtest命令添加1之后，您仍然应该获得“Hello Bob！”。

添加以下SQL查询：
`INSERT INTO radreply (username, attribute, value,op) VALUES ('bob', 'Fall-Through', 'Yes','=');`
这将导致SQL模块检查分配给Bob的组。当您在radtest命令的末尾添加1时，您现在应该获得“Hello PPP协议！”。

当在radcheck表中使用密码定义用户并且所有检查都通过时，将使用正常身份验证进行上述方案。

当出现问题时，例如，如果密码不正确，或者radcheck中定义的检查未通过，或者用户甚至没有列在radcheck表中，则SQL模块将检查分配给用户的组。

当用户不在radcheck表中或者在radcheck中指定的AVP不匹配时，请记住以下两点。这些点不受sql.conf文件中read_groups值的影响。这意味着无论你喜不喜欢它都会发生。

+ 当SQL模块在radcheck表中找不到用户时，它会检查是否有分配给用户的组。
+ 当为用户定义的radcheck项不匹配时，SQL模块检查是否有分配给用户的组。

> #### 回复消息AVP
> 当您在radgroupreply中指定Reply-Message AVP时，即使用户提供了错误的密码，但是当radgroupcheck'子测试'通过时，它也将被返回。 Reply-Message AVP是特殊的，因为它是唯一可以伴随RADIUS中的Access-Reject数据包的AVP。即使radgroupcheck'子测试'通过，也不会使用Access-Reject数据包返回groupreply中指定的其他AVP。

## 配置文件
可以在SQL中创建配置文件，然后以两种方式将其分配给用户：

+ 通过default_user_profile指令为sql / mysql / dialup.conf文件中的所有用户指定默认配置文件
+ 通过User-Profile检查属性的形式为用户明确指定配置文件

配置文件是至少一个组的成员的用户。 此用户不需要radcheck和radreply表中的任何条目。 让我们修改数据库，以便bob使用一个配置文件：

1. 登录RADIUS MySQL数据库并发出以下SQL命令以创建所需的条目：
```
delete from radcheck;
delete from radreply;
delete from radgroupreply;
delete from radgroupcheck;
delete from radusergroup;
INSERT INTO radcheck (username, attribute, value,op) VALUES
('bob', 'Cleartext-Password', 'passbob',':=');
INSERT INTO radcheck (username, attribute, value,op) VALUES
('bob', 'User-Profile', 'student_profile',':=');
INSERT INTO radgroupreply (groupname, attribute, value,op)
VALUES ('students', 'Reply-Message', 'Hello Student!','=');
INSERT INTO radusergroup (username, groupname, priority) VALUES
('student_profile', 'students', 10);
```
2. 通过在sql.conf中将read_groups更改回默认值yes，重新激活组的读取。
3. 在调试模式下重新启动FreeRADIUS。
4. 使用radtest程序验证为bob：
`radtest bob passbob 127.0.0.1 100 testing123`

应返回用于student_profile用户的回复消息。

用户简档AVP的值是至少一个组的成员的用户的值。 在我们的示例中，配置文件用户名为student_profile，它是学生组的成员。

然后，分配给学生的radgroupcheck和radgroupreply属性将应用于具有检查属性User-Profile：= student_profile的任何用户。 

用户student_profile在此处用作配置文件而不是普通用户。

本节介绍了在FreeRADIUS中使用SQL模块的重点。 在下一节中，我们将使用LDAP目录作为用户存储。

