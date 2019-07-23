> # 行动时刻 - 在FreeRADIUS中整合Linux系统用户

FreeRADIUS文档建议它作为非特权用户运行。 当我们将系统用户作为用户存储时，此非特权用户将需要访问/etc/shadow文件。 对于/etc/shadow文件的权限和所有权，三个发行版中的每一个都有不同的默认配置。

## 准备权限

默认情况下，Ubuntu具有/etc/shadow文件的正确权限。 在Ubuntu中，/etc/shadow文件由名为shadow的组拥有，该组具有对该文件的读取权限。 安装FreeRADIUS时，会添加一个名为freerad的用户和组。 用户freerad被添加到影子组，允许对/etc/shadow进行freerad读取访问。
您可以使用以下命令在Ubuntu上进行确认。 要检查/etc/shadow文件的所有权：

```
$> ls -l /etc/shadow
-rw-r----- 1 root shadow 743 2012-06-06 18:32 /etc/shadow
```

使用以下命令确认freerad用户所属的组：

```
$> getent group | grep freerad
shadow:x:42:freerad
freerad:x:112:
ssl-cert:x:113:freerad
```

## SUSE与众不同

随FreeRADIUS一起发布的SUSE README文件建议您将radiusd.conf文件中的用户和组的值更改为以下内容：
```
user = root
group = root
```
虽然SUSE中的/etc/shadow文件由shadow组拥有，但将FreeRADIUS运行的非特权用户添加到shadow组不会产生预期的结果。 README文件的存在是有原因的。

> **当用户和组的值发生更改时**
> 
> 如果您在radiusd.conf中更改了用户和组的值，则在重新启动服务器时将显示以下消息：
> 
> **我们不拥有/var/run/radiusd/radiusd.sock**
> 
> 问题在于radiusd目录的所有权，因为radiusd.sock文件甚至不存在。 通过将/var/run/radiusd目录的所有权更改为radiusd.conf文件中指定的用户的所有权来修复它。 在SUSE上，以下命令将解决问题：chown root. /var/run/radius

## CentOS
CentOS没有名为shadow的组，而/ etc / shadow文件由用户和组root拥有。 要允许运行FreeRADIUS服务器的组（radiusd）读取/ etc / shadow文件，我们会将/ etc / shadow的组所有权更改为radius。
然后，我们使用以下命令将影子文件的读取权限提供给拥有它的组：
chgrp radiusd / etc / shadow
chmod g + r / etc / shadow
这使我们结束了环境准备; 现在我们可以激活系统用户了

## 激活系统用户
要将系统用户包含在FreeRADIUS中作为用户存储，这是一个简短而甜蜜的过程。
跟着这些步骤：
1.编辑 sites-enabled/default文件，并在授权部分(authorize)下取消注释unix。
2.在调试模式下重新启动FreeRADIUS。
3.使用Linux服务器上的existng系统用户执行身份验证测试。 我们假设bob是系统用户，他的密码是passbob。
radtest bob passbob 127.0.0.1 100 testing123
4.观察调试输出以查看测试是否成功。

## ***刚刚发生了什么？***
我们已将unix模块包含在默认虚拟服务器的authorize部分中。 这使得FreeRADIUS能够针对在服务器上定义的系统用户检查传入的访问请求。
让我们研究一下FreeRADIUS的调试输出，看看如何返回Access-Accept。

## 授权使用unix模块
在FreeRADIUS的调试输出中，以下行表示unix模块找到了一个名为bob的用户并更新了FreeRADIUS中的一些内部值：
++ [unix]returns updated
当用户文件中定义的alice进行身份验证时，您可以将其与输出进行比较：
++ [unix]returns notfound
unix模块返回一个已知的良好密码（采用Crypt格式），pap模块可以使用该密码对用户进行身份验证。

> 如果即使定义了用户，unix模块也返回notfound，请确认/etc/shadow文件上的权限是否正确。 如果仍然失败，请将radius.conf文件中的用户和组行更改为root用户，重新启动FreeRADIUS，然后重试。
还要记住，Linux/Unix区分大小写。 这适用于用户名和密码！

## 使用pap进行身份验证
调试输出的验证部分指示pap使用由unix模块给出的已知良好密码进行验证。

```
Executing group from file /etc/freeradius/sites-enabled/default
+- entering group PAP {…}
[pap] login attempt with password "passbob"
[pap] Using CRYPT password "$6$SI3ZfzEr$M0ujsOhTAXT7LP5KzzYhHdFL4/
iJtfEdX3lOeGJLbDDc.SQsTnl8yuOqB948DDvdKBScb7Mp8Myro5FeekgLw."
[pap] User authenticated successfully
++[pap] returns ok
```

## 包含系统用户的提示

将系统用户包括为用户存储时，需要记住以下几点：

+ 只能使用PAP身份验证协议。Chap和MS-Chap不工作。
+ Linux系统使用/etc/shadow文件来存储密码。非特权用户无法访问此文件。当您以root以外的用户身份运行freeradius时（按照建议），请确保此用户可以访问shadow文件。
+ SUSE是不同的，需要以用户和组根用户身份运行freeradius。这在radius.conf文件中指定。
+ 如果您创建的系统用户只会被freeradius使用，那么出于安全原因，最好将其默认主目录和shell更改为与用户nobody相同。

> 较旧的Linux/Unix系统可能只使用/etc/passwd文件而不是
实现shadow密码数据库机制。 用户密码将与用户的详细信息一起存储在/etc/passwd文件中。
在这些系统上，/etc/passwd文件的第二个字段将包含加密的密码而不是x。
在/etc/shadow文件的第二个字段中，可能会找到以下特殊字符：

> + NP或！ 或null（无密码）
> + LK或*（账户被锁定）
> + ！ （密码已过期）