# 行动时间 - 引用属性
在本节中，我们将使用属性。
## if语句中的属性
Unlang可以在虚拟服务器定义内的各个部分中使用。 以前我们在授权部分使用过它。 根据FreeRADIUS作者的指示，您不应在authenticate部分中使用unlang。 我们将在post-auth部分中使用unlang来确定是否使用了Auth-Type = PAP，如果它确实用于验证用户，则给出反馈。
1. 编辑FreeRADIUS配置目录下的sites-available / default虚拟服务器，并在该部分顶部的post-auth部分中添加以下内容：
```
if(control:Auth-Type == 'PAP'){
	update reply {
		Reply-Message := "We are using %{control:Auth-Type}
		authentication"
	}
}
```
2. 在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证。
	指定的Reply-Message应包含在回复中。

## 刚刚发生了什么？
我们使用unlang来测试控制属性列表中的Auth-Type AVP的值。如果它等于PAP，我们修改了回复属性列表中的Reply-Message AVP。虽然if语句只包含五行，但有一些重要的事情需要讨论。我们将讨论以下内容：
+ 在条件中引用属性的方法
+ 比较运算符
+ 在属性列表中更改和添加属性
### 引用条件中的属性
引用属性时，我们在条件中使用了另一种语法。它如下：
`if（“％{control：Auth-Type}”=='PAP'）{`
虽然两者都可以使用，但首先是易于阅读的首选。替代语法通常在字符串中使用。然后将插入属性的值以成为字符串的一部分。这称为字符串扩展。我们使用字符串扩展来创建Reply-Message的值。
`Reply-Message := "We are using %{control:Auth-Type} authentication"`
如果我们省略对属性列表（control :)的引用，unlang将使用request：Auth-Type。如果此属性不在属性列表中，则unlang返回false。
### 比较运算符
可以在条件测试中使用相当多的比较运算符。 AVP的数据类型将确定哪些运营商可供使用。

|运算符|数据类型|示例|
|:-|:-:|:-|
|==|字符串或数字|(control:Auth-Type == 'PAP')|
|!=||(reply:Idle-Timeout != 60)|
|<|数字|(reply:Idle-Timeout <= 60)|
|<=|||
|>|||
|>=|||
|=~|字符串的正则表达式|(request:User-Name =~ /^.*\.co\.
za/i)|
|!~|||

### 属性操作
Unlang可用于修改AVP。要修改AVP的值，我们需要在unlang中使用update关键字。 update语句的概要如下：

update语句只能包含属性。运算符的值非常重要，因为它将确定如何处理具有该名称的列表中的现有属性。我们将在这里讨论三个常用的运算符。请注意，其他运算符确实存在。有关它们的更多信息，请参阅unlang手册页。

|运算符|说明|
|:-|:-:|
|=|当且仅当该列表中不存在同名属性时，才将该属性添加到列表中。|
|:=|将属性添加到列表中。如果该列表中已存在任何同名属性，则其值将替换为当前属性的值。|
|+=|将属性添加到列表尾部，即使列表中已存在同名属性也是如此。|

请注意，为属性指定的值是正确的类型。将字符串值分配给应采用整数值的属性时，将导致错误。

## 变量
变量不能像在其他语言中一样在unlang中声明。使用unlang所有属性都是变量，但并非所有变量都是属性。在属性可以作为属性列表中的变量引用之前，必须先将其添加到列表中。对变量的所有引用必须包含在双引号或反引号字符串中。对此引用字符串中的变量的引用采用％{<variable>}形式。在上一节中，我们引用了Auth-Type变量，它是一个属性：
`Reply-Message := "We are using %{control:Auth-Type} authentication"`
在本节中，我们将引用非属性的变量。