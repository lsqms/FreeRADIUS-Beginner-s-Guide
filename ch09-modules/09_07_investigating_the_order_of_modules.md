# 行动时刻 - 调查模块的顺序
本练习要求您注意各个部分中列出的模块的顺序
在虚拟服务器内部。
1. 打开FreeRADIUS配置目录下的sites-enabled / default文件。
2. 仔细阅读并注意各部分内部使用模块的顺序。 一些注释会提到为什么模块位于某个部分内的某个位置。

以下关于默认文件的说明应该使事情更清楚。

## 访问请求
当Access-Request数据包进入FreeRADIUS服务器时，它首先由虚拟服务器的授权(authorize)部分处理。然后它可以由验证(authenticate)和会话(session)部分处理，最后传递给post-auth部分。

授权部分中列出的第一个模块是预处理。这是第一个原因。这是授权部分的一般流程：
+ 预处理模块进行健全性检查，并将奇怪的属性更改为更标准的属性。
+ chap和mschap等模块测试请求是CHAP还是MS-CHAP，并相应地更改Auth-Type的值。
+ sql，ldap和files等模块尝试从用户存储中找到用户。
+ expiration和logintime等模块将确定是否对用户施加了任何限制。这基于sql，ldap和files模块收集的信息。
+ pap模块最后列出，因为它检查上述模块中是否都没有设置Auth-Type的值，如果是，则将其设置为PAP，以便验证部分可以使用pap模块进行验证。

从列出的项目中可以看出，模块内部模块的位置必须合乎逻辑。如果我们将expiration和logintime模块放在sql，ldap和files模块之前，则他们将没有任何可用的数据来进行检查。
您可以以相同的方式关注Accounting-Requests。会计请求首先由preacct处理，然后由accounting部分处理。

## 返回代码
每个模块都必须返回一个代码。第7章授权中讨论了可返回的各种代码。返回代码可以极大地影响请求的流程。例如，如果模块在authorize部分内返回reject，则authenticate和session部分将不会处理该请求，但会直接传递给post-auth部分。通过使用unlang，您还可以使用逻辑来测试某些返回代码，并在满足条件时以指定的方式进行响应。
有了这个，我们涵盖了模块使用的所有重点。下一节将介绍FreeRADIUS中可用的一些有趣模块。