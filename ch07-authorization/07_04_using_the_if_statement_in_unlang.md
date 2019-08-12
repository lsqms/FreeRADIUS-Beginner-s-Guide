# 行动时刻 - 在unlang中使用if语句
if语句本身并不复杂。 它具有以下格式：

```
if(condition){
	...
}
```

由于其许多可能性，条件部分可能变得复杂。
## 使用if语句获取返回码
我们现在将查看模块的返回代码，并使用此代码与指定的条件进行比较。 FreeRADIUS中的每个模块都需要在调用时返回代码。
随后可以将此代码的值用作if语句中的条件检查。

### 使用if语句授权用户
如果用户不在用户文件中，本练习使用if条件拒绝访问请求。

1. 编辑FreeRADIUS配置目录下的sites-available / default虚拟服务器，并在authorize部分的files条目下面添加以下行：

```
if(noop){
	reject
}
```

2. 在调试模式下重新启动FreeRADIUS，并尝试使用用户文件中不存在的用户名和密码进行身份验证。 我们假设在任何地方都没有定义ali：
`radtest ali passme 127.0.0.1 100 testing123`

	你应该得到一个Access-Reject数据包。

## 刚刚发生了什么？
我们在授权部分下的文件模块下添加了一个条件检查。
要激活此更改，必须重新启动FreeRADIUS。 这将导致radius服务器读取和解释unlang代码。 然后通过向服务器发送Access-Request数据包来测试unlang代码。

我们首先看看if条件，然后看看if条件满足时所采取的行动。

### 模块返回码

如果查看FreeRADIUS的调试输出，您将看到每个模块如何返回代码。 以下是可用的返回代码列表及其含义：

|模块返回码|描述|
|:-|:-:|
|notfound|未找到信息|
|noop|该模块什么也没做|
|ok|该模块成功执行|
|updated|该模块更新了请求|
|	|例如，它设置Auth-Type内部AVP|
|fail|模块失败了|
|reject|该模块拒绝了该请求|
|userlock|用户被锁定了|
|invalid|配置无效|
|handled|该模块本身处理了请求|

我们可以使用unlang的if语句来测试模块的指定返回码。为此，我们必须给unlang一个提示，它必须测试模块返回代码。我们通过在if语句的条件中指定要作为不带引号的字符串进行测试的返回代码来完成此操作。

>如果条件是不带引号的字符串和上表中列出的模块返回代码之一，则unlang会将此字符串与最新模块的返回代码进行比较。

您可能认为我们应该测试notfound而不是noop。当用户不在users文件中时，files模块返回noop而不是notfound。在创建条件测试之前，请记住测试或找出模块在某些情况下返回的值。如果您没有这样做，结果可能与您的预期不同。

## unlang中的关键字
当满足if条件时，我们可以采取各种行动。 Unlang使用关键字来处理请求。 if语句是一个关键字。还有update关键字，在操作属性时使用，将在本章后面讨论。然后还有一个可以在if语句中使用的关键字列表。如果用户不在用户文件中，我们使用reject关键字立即拒绝请求。下表列出了if语句中可以使用的关键字以及它们对请求的影响。

|Keywords|描述|
|:-|:-:|
|noop|什么也不做|
|ok|指示服务器正确处理请求。|
||如果本地管理员确定故障不是灾难性的，则此关键字可用于覆盖先前的故障。|
|fail|导致请求被视为发生故障。|
|reject|导致请求立即被拒绝。|

请注意，虽然名称与模块的返回代码相同，但模块返回代码与这些关键字之间存在差异。 这些关键字是unlang的一部分，在if语句中使用。 如果if语句中没有定义关键字，默认情况下它将返回noop。

除了这些关键字，我们还可以指定任何FreeRADIUS模块的名称。模块名称被视为关键字。 if语句也可以嵌套。

## 试一试 - 使用条件语句的其他测试

有条件的陈述为我们提供了各种测试功能。这些可以在授权期间使用，例如，检查NAS是否已经使用访问请求提供了必需属性。我们甚至可以使用逻辑运算符组合测试来创建在授权用户之前必须满足的复杂条件。本节将介绍另外两个条件测试，这些测试可用作构建块以创建灵活的授权策略。

这些练习假设一个未触及的站点可用/默认文件，可以独立完成。我们还假设用户文件包含一个名为alice的用户，其密码为passme（这与前面所有章节中定义和使用的用户相同）。

## 检查属性是否存在
我们可以检查是否存在指定的AVP。如果我们在条件中将属性的名称指定为不带引号的字符串，则unlang将检查请求中是否存在此AVP。

1. 编辑FreeRADIUS配置目录下的sites-available / default虚拟服务器，并在authorize部分的files条目下面添加以下内容：
```
if(Framed-Protocol){
reject
}
```
2. 在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证，但也在radtest命令的末尾添加1。这将包括请求中的Framed-Protocol AVP：
`radtest alice passme 127.0.0.1 100 testing123 1`

	你应该得到一个Access-Reject数据包。

我们可以从调试输出中看到if语句是如何计算的。将Framed-Protocol包含在请求中时的输出与丢失时的输出进行比较。

>虽然我们在此概念验证期间使用Framed-Protocol属性以方便，但在现实世界中，Framed-Protocol AVP可以存在于Access-Accept和Access-Reply数据包中。它表示用于框架访问的框架。最常见的值是点对点协议（PPP）。

如果Framed-Protocol存在，则输出以下内容：
```
++? if (Framed-Protocol)
? Evaluating (Framed-Protocol) -> TRUE
++? if (Framed-Protocol) -> TRUE
++- entering if (Framed-Protocol) {...}
+++[reject] returns reject
++- if (Framed-Protocol) returns reject
```
如果不存在，则输出以下内容：
```
++? if (Framed-Protocol)
? Evaluating (Framed-Protocol) -> FALSE
++? if (Framed-Protocol) -> FALSE
```

请记住，此条件测试仅检查请求中是否存在AVP。它不检查AVP的特定值。我们将在本章后面测试某些值。

到目前为止，我们有两种类型的未加引号的字符串：

+ noop被unlang解释为最后一个模块的返回码。
+ 框架协议由unlang解释为属性。

如果没有引用字符串，会发生什么？答案如下：

+ 一个字等于真。
+ 零的数量等于假，其他数字等于真。

>如果要检查属性是否不存在，只需在属性前面添加一个exclamaton标记（！）。 unlang中的感叹号是逻辑NOT，并测试条件是否不存在。

## 使用逻辑表达式对用户进行身份验证

Unlang还在conditon语句中支持逻辑AND（&&）和逻辑OR（||）。在本练习中，我们将拒绝不在用户文件中的用户或请求中存在Framed-Protocol AVP的用户。

1. 编辑FreeRADIUS配置目录下的sites-available / default虚拟服务器，并在authorize部分的files条目下面添加以下行：
```
if((noop)||(Framed-Protocol)){
	reject
}
```
2. 在调试模式下重新启动FreeRADIUS并尝试作为alice进行身份验证，但也在radtest命令的末尾添加1。这将包括请求中的Framed-Protocol AVP：
`radtest alice passme 127.0.0.1 100 testing123 1`
你应该得到一个Acces-Reject数据包。
3. 尝试使用用户文件中不存在的用户名和密码进行身份验证。
您还应该获得一个Access-Reject数据包。
4. 最后尝试使用用户文件中不存在的用户名和密码进行身份验证，并在radtest命令的末尾添加1。您还应该获得一个Access-Reject数据包。

来自FreeRADIUS服务器的调试反馈将指示在请求期间如何评估if条件。
## 属性和变量
RADIUS中的授权在很大程度上取决于属性。我们可以使用Access-Request中的AVP来验证它是否符合我们在授权期间的要求。我们还可以在Access-Reply内返回AVP，以指示NAS仅授权用户执行某些操作。

由于RADIUS协议都是关于属性的，因此unlang主要使用属性作为变量。还有一些例外，其中变量不是属性。本节也将介绍这一点。

### 属性列表
FreeRADIUS通过将属性存储在列表中来管理属性。列表就像一个名称空间，它允许具有相同名称的属性独立存在于不同的位置。 Unlang可用于在这些不同的列表中操作或添加属性：
+ 有一个请求列表，其中包含请求中的所有AVP，例如User-Name。
+ 有一个回复列表，其中包含最终将在回复中的所有AVP，例如Reply-Message。
+ 我们还使用了前面章节中的控制列表，其中我们将此列表中的属性称为内部属性，例如Auth-Type。
+ 要引用特定列表中的属性，我们使用列表的名称和冒号后跟属性名称，例如request：Framed-Protocol。
+ 如果列表的名称被省略，则它引用请求列表。这就是为什么我们可以在之前的练习中没有指定列表名称。
+ 可以使用以下属性列表：request，reply，control，proxy-request，proxy-reply，outer.request，outer.reply，outer.control，outer.proxy-request和outer.proxy-reply。
+ 通过使用update关键字添加或修改属性。 update关键字与必须修改的列表名称一起使用会创建一个更新部分，可以在其中修改或添加属性。

在介绍属性和属性列表后，是时候在实际练习中使用它们了。


