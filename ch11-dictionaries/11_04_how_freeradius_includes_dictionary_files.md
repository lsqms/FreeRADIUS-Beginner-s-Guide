# FreeRADIUS如何包含字典文件
FreeRADIUS默认安装许多预定义的字典文件。这些字典文件存储在/usr/share/freeradius目录中（如果使用configure; make; make install模式从源代码安装它们将存储在/usr/local/share/freeradius中）。 Dictonary文件根据惯例命名。名称位于字典字典中。<identifier>。标识符可以分为三类，如下：

+ Vendor/technology(供应商/技术名称)：例如dictionary.mikrotik或dictionary.wimax
+ RFC number：例如dictionary.rfc2865
+ FreeRADIUS的内部词典：例如dictionary.freeradius.internal

如果安装了字典文件，它不会自动暗示FreeRADIUS将使用此字典文件。 FreeRADIUS必须配置为包含特定的字典文件。我们通过以下方式执行此操作：

1. 在FreeRADIUS配置目录中有一个名为dictionary的文件。此文件的内容不是很令人激动，因为它包含一行，取消注释：
$ INCLUDE /usr/share/freeradius/dictionary
2. 源文件（/usr/share/freeradius/dictionary）更令人兴奋，因为该文件包含要使用的各种字典文件的列表：
```
$INCLUDE dictionary.rfc2865
$INCLUDE dictionary.rfc2866
...
```

两个字典文件中的注释警告我们不要更改任何预定义的字典文件，因为它会在FreeRADIUS更新时引起问题。 下一节将向我们展示根据最佳实践做什么。

> 最好的做法是有原因的。 编辑位于/usr/share/freeradius目录中的文件可能非常棘手且危险。 仅通过编辑位于FreeRADIUS配置目录下的字典文件来添加属性和字典。

## 包括您自己的字典文件
有三种情况可以更改FreeRADIUS中的默认字典配置。
## 包括已安装的字典文件
有时字典文件安装在/usr/share/freeradius目录下，但未在/usr/share/freeradius/ dictionary文件中列出。然后，该字典应特定地包含在FreeRADIUS配置目录下的字典文件中。
请记住包含绝对路径：
`$ INCLUDE /usr/share/freeradius/dictionary.chillispot`
> 测试未包含的已安装AVP
> 如果grep查找/usr/share/freeradius目录中的required属性并找到该属性，请检查包含该属性的字典文件是否确实列在/usr/share/freeradius/dictionary中。
如果不是，则必须以这种方式包含它。
## 添加私有属性
这些私有属性由FreeRADIUS内部使用。在第7章授权中，我们使用了这些属性。它们通常与更复杂的unlang实现一起使用，并在FreeRADIUS配置目录下的字典文件中定义：
```
ATTRIBUTE FRBG-Reset-Type 3050 string
ATTRIBUTE FRBG-Total-Bytes 3051 string
```
建议为这些类型的属性使用唯一的前缀，以避免重复的名称。 它们的编号介于3000和4000之间，永远不会放在RADIUS数据包中。
## 更新现有字典
IT世界是一个快速变化的世界，新的玩家进入现场，或者现有的玩家改变他们的软件。 如果供应商进行了更改，我们需要更新现有的字典文件。 建议的方法是不要触摸预定义的字典文件，而是包含更新的字典，以便在FreeRADIUS启动时覆盖预定义的字典信息。
在下一个练习中，我们将更新现有的MikroTik词典，以包括对Mikrotik-Total-Limit AVP的支持。