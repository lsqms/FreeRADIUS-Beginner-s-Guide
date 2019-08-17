# 行动时刻 - 更新MikroTik词典
Isaac给他的朋友发了电子邮件，通知他有关配置损坏的信息。 然后他的朋友回复并指示Isaac访问以下URL，其中显示了MikroTik支持的最新RADIUS属性：
http://wiki.mikrotik.com/wiki/Manual:RADIUS_Client
虽然网页的内容最初有点令人困惑，但Isaac设法执行以下操作以修复其FreeRADIUS服务器上的所有内容：

1. 将预定义的dictionary.mikrotik文件复制到FreeRADIUS配置目录中的文件夹：
mkdir /etc/raddb/dictionary.local
cp /usr/share/freeradius/dictionary.mikrotik /etc/raddb/dictionary.local
2. 根据MikroTik的网页更新/etc/raddb/dictionary.local/dictionary.mikrotik以包含以下内容：
```
#Add New Mikrotik Attributes
ATTRIBUTE Mikrotik-Wireless-PSK 16 string
ATTRIBUTE Mikrotik-Total-Limit 17 integer
ATTRIBUTE Mikrotik-Total-Limit-Gigawords 18 integer
ATTRIBUTE Mikrotik-Address-List 19 string
ATTRIBUTE Mikrotik-Wireless-MPKEY 20 string
ATTRIBUTE Mikrotik-Wireless-Comment 21 string
```
3. 编辑FreeRADIUS配置目录中的字典文件以包含此更新的字典：
```
$INCLUDE /usr/share/freeradius/dictionary
#
# Place additional attributes or $INCLUDEs here.
# They will over-ride the definitions in
# the pre-defined dictionaries
$INCLUDE /etc/raddb/dictionary.local/dictionary.mikrotik
```
4. 在调试模式下重新启动FreeRADIUS并测试alice的身份验证。 应返回以下内容：
```
rad_recv: Access-Accept packet from host 127.0.0.1 port 1812,
id=10, length=32
Mikrotik-Total-Limit = 10240
```

## 刚刚发生了什么？
我们更新了MikroTik字典以包含最新属性。未删除预定义的字典文件，但是以覆盖预定义字典的方式包括更新的字典文件。虽然并不困难，但要记住一些重要的要点。

## 查找最新支持的属性
在更新字典时，查找供应商支持的最新RADIUS属性通常是最困难的部分。开始查看的好地方是固件更新的发布说明或供应商的网站。不要总是接受供应商提出的关于如何更新字典的建议或说明，因为这会引入新的问题。使用FreeRADIUS提出的指令更好。

## 更新的字典文件的位置
更新的字典文件的位置由您决定。在本练习中，我们将其存储在FreeRADIUS配置目录中的子目录中。这有助于我们将所有配置项保存在一个位置。

## 字典的顺序
字典文件的列出顺序非常重要。为了确保更新的字典文件的内容覆盖预定义的内容，我们将其列为原始字典文件之后的位置。

## 属性名称
在本章开头，我们提到了词典可以使我们受益。字典中条目的拼写并不重要，因为它只是用于映射。出于这个原因，我们更改了MikroTik网页上列出的属性名称以符合RADIUS约定。我们没有指定MIKROTIK_TOTAL_LIMIT，而是使用了Mikrotik-Total-Limit。在为用户指定AVP时，遵守约定也有助于我们。
现在我们知道何时以及如何更改默认字典配置，现在是时候更仔细地查看字典文件的格式了。

## 升级FreeRADIUS
较新版本的FreeRADIUS还可能包含对现有字典文件的修改和更新。 建议在升级FreeRADIUS之前创建包含字典文件的目录的备份，以便在升级过程在现有字典文件中造成破坏时从中恢复它们。
升级还可能会更改在初始安装后调整的某些目录和文件的权限。 确保升级后FreeRADIUS服务器启动时没有错误，以确认一切按计划进行。























