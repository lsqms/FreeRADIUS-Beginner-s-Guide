# 字典文件的格式
有两种类型的AVP。 标准RADIUS属性称为属性值对（AVP），而来自供应商的属性称为供应商特定属性（VSA）。 虽然有两种类型的AVP，但我们通常不区分这两种类型，只需将它们称为AVP即可。 VSA使用RADIUS属性类型26（Vendor-Specific）。 此AVP的值用于依次包装特定于供应商的属性。 查看更新的dictionary.mikrotik文件的内容以了解dictionary文件的内容。 以下讨论将基于此。

## 注释中的注释
以＃开头的行被视为注释。 在某些情况下，注释包含有价值的信息，应该注意这些信息以避免系统损坏：
```
# -*- text -*-
# http://www.mikrotik.com
#
# http://www.mikrotik.com/documentation//manual_2.9/dictionary
#
# Do NOT follow their instructions and replace
# the dictionary in /etc/raddb with the one that they
# supply. It is NOT necessary.
#
# On top of that, the sample dictionary file they provide
# DOES NOT WORK. Do NOT use it.
```

## 供应商定义
定义VSA的字典采用以下格式：
```
VENDOR Mikrotik 14988
BEGIN-VENDOR Mikrotik
...
END-VENDOR Mikrotik
```

供应商定义必须在开始时包含供应商的编号。 供应商编号由IANA分配。 您可以从以下URL获取现有分配：`http://www.iana.org/assignments/enterprise-numbers`

夹在BEGIN-VENDOR和END-VENDOR之间的是ATTRIBUTE和VALUE定义。

## 属性和值
我们将仅在本章中介绍基本属性和值定义。但是，FreeRADIUS包含一个字典手册页，其中包含更多详细信息：
`man dictionary`
属性定义采用以下格式：
ATTRIBUTE <name> <number> <type> [vendor | options]
## 名称字段
虽然名称字段可以是任何非空格文本，但最好遵循来自RFC的现有约定。这就是我们使用新的MikroTik属性定义所做的。该名称通常表示它映射到的数字的功能，例如Mikrotik-Total-Limit用于限制会话期间的总字节数。
## 数字字段
这些数字通常是递增的，由供应商或IANA等标准组织确定。每个号码对客户端或服务器都有特殊含义。这就是我们必须从MikroTik获取最新数字及其含义的原因。您自己分配的唯一数字是在主词典文件中指定的3000到4000之间的数字：
```
#
# If you want to add entries to the dictionary file,
# which are NOT going to be placed in a RADIUS packet,
# add them here. The numbers you pick should be between
# 3000 and 4000.
#
```

## 类型字段
类型字段可以是几种预定义类型之一。 下表列出了这些类型以及简短描述：

|Type|Descripton|
|:-|:-:|
|date|表示自1970年1月1日格林尼治标准时间00:00:00以来的秒数的32位整数值（Unix纪元）|
|integer|32位整数值（值从0到4,294,967,295）|
|text|包含UTF-8编码字符的1-253个八位字节|
|string|包含二进制数据的1-253个八位字节|
|ipaddr|网络字节顺序（NBO）中占4个八位字节|
|ifid|网络字节顺序中占8个八位字节|
|ipv6addr|网络字节顺序中占16个八位字节|
|ipv6prefix|网络字节顺序中占18个八位字节|
|ether|“hh:hh:hh:hh:hh:hh”的6个八位字节,其中'h'是十六进制数字，大写或小写|
|abinary|Ascend的二进制flter格式|
|octets|原始八位字节，打印并输入为十六进制字符串，例如0x123456789abcdef|

没有例外的生活会怎样！ dictionary.wimax文件指定了一些非标准数据类型。 例如，有一个名为signed的有符号整数类型。 这与不带符号的整数数据类型不同。 WiMAX VSA也具有非标准格式，在dictionary.wimax文件的注释中对此进行了讨论。

## 可选供应商字段

作为我们通过获取一个完整的新文件来更新MikroTik字典的练习的替代方法，我们可以简单地将以下内容添加到位于FreeRADIUS配置目录下的字典文件中：
```
#Add New Mikrotik Attributes
ATTRIBUTE Mikrotik-Wireless-PSK 16 string Mikrotik
ATTRIBUTE Mikrotik-Total-Limit 17 integer Mikrotik
ATTRIBUTE Mikrotik-Total-Limit-Gigawords 18 integer Mikrotik
ATTRIBUTE Mikrotik-Address-List 19 string Mikrotik
ATTRIBUTE Mikrotik-Wireless-MPKEY 20 string Mikrotik
ATTRIBUTE Mikrotik-Wireless-Comment 21 string Mikrot
```
用于更新字典的方法取决于您。 我更喜欢将供应商的所有属性定义保存在单个更新的字典文件中。

## 值定义
定义值以为integer类型的属性提供人类可读选项。这有助于我们记住名称而不是数字。在dictionary.mikrotik文件中，定义了以下值：
```
# MikroTik Values
VALUE Mikrotik-Wireless-Enc-Algo No-encryption 0
VALUE Mikrotik-Wireless-Enc-Algo 40-bit-WEP 1
VALUE Mikrotik-Wireless-Enc-Algo 104-bit-WEP 2
```
这意味着对于Mikrotik-Wireless-Enc-Algo属性，我们可以指定这三个值中的任何一个而不是使用等效整数。例如，如果我们也想对Alice强制不执行加密，我们可以将用户文件修改为以下内容：
```
“alice” Cleartext-Password：=“passme”
Mikrotik-Total-Limit = 10240，Mikrotik-Wireless-Enc-Algo = Noencryption
```
值定义是一个很好的帮助，这使得属性赋值的读取更加容易。值定义仅适用于整数类型的属性。

## 访问字典文件
在我们讨论本章结尾之前，还有一个重要的问题仍然需要提及。虽然/usr/share/freeradius下的文件通常可供所有人读取，但位于FreeRADIUS配置目录下的主字典配置文件可能并非总是如此。如果您尝试以普通用户身份运行radtest并且拒绝启动，请检查FreeRADIUS配置目录以及此目录中的字典文件的访问权限。
为了允许普通用户对目录中的文件进行读访问，我们设置了目录的执行权限。我们以SUSE为例：
`＃> chmod o + x /etc/raddb`
要使普通用户能够对字典文件进行读取访问，请为其添加读取权限：
`#> chmod o+r /etc/raddb/dictionary`




























