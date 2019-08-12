# 使用具有不同配置的一个模块
FreeRADIUS允许您使用具有各种配置的一个模块。 如前所述，如果您熟悉编程术语，则类似于具有不同实例的对象。

## 试一试 - 创建一个模块的多个实例
Isaac陷入困境。 他完全忘记了教授的帐户也会过期，他不想在回复信息中向教授讲学生语言。 是时候解决这个问题：
1. 编辑FreeRADIUS配置目录下modules子目录中的expiration文件。 更改以下行：
```
#reply-message = "Password Has Expired\r\n"
reply-message = "Dude, you are like sooooo expired\r\n"
```
更改为：
```
reply-message = "Password Has Expired\r\n"
#reply-message = "Dude, you are like sooooo expired\r\n"
```
2. 在FreeRADIUS配置目录下的modules子目录中创建一个名为exp_students的文件，其中包含以下内容：
```
expiration exp_students {
	reply-message = "Dude, you are like sooooo expired\r\n"
}
```
3. 在FreeRADIUS配置目录下的modules子目录中创建一个名为exp_professors的文件，其中包含以下内容：
```
expiration exp_professors {
	reply-message = "Dear professor %{User-Name}, kindly contact helpdesk concerning your expired account\r\n"
}
```
4. 编辑FreeRADIUS配置目录下的sites-available / default文件，并将authorize部分内的expiraton部分修改为以下内容：
```
if(control:Group == "students"){
	exp_students
}
elsif(control:Group == "professors"){
	exp_professors
}
else{
	expiration
}
```
5. 确保用户文件具有以下要测试的条目：
```
"alice" Cleartext-Password := "passme", Group := "students", Expiration := "4 May 2010"
"bob" Cleartext-Password := "passbob", Group := "professors", Expiration := "4 May 2010"
```
6. 在调试模式下重新启动FreeRADIUS并尝试以alice和bob身份进行身份验证，以查看回复消息中的差异：
```
$> radtest alice passme 127.0.0.1 100 testing123
$> radtest bob passbob 127.0.0.1 100 testing123
```
## 刚刚发生了什么？
我们已经设法根据用户所属的组在回复消息中调整术语。 虽然练习非常简单，但原则可以应用于任何模块。
+ 模块的第一个实例不需要命名，使用它只是一个引用部分内部模块的情况。 我想到了像ldap，chap，files和sql这样的例子。
+ 如果要使用具有不同配置的相同模块，则必须为模块声明一个命名部分，其中包含备用配置。 要在一个部分中使用它，您必须引用命名部分的名称。 我们使用了exp_students和exp_professors，它们是为expiration模块创建的命名部分。
FreeRADIUS中的此功能允许您使用sql模块连接到不同的数据库或ldap模块以使用不同的目录或文件模块来使用不同的用户文件。

![my-logo.png](https://upload-images.jianshu.io/upload_images/13623636-6d878e3d3ef63825.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "my-logo")

> 注意条件陈述的结构
> 条件陈述的结构非常重要。 if，elsif和else关键字都应该在一个新行上。 如果你不这样做，你将得到意想不到的结果。
错误示例：
```
if(condition) {
	…
}else{
	…
}
```
正确示例
```
if(condition) {
	…
}
else{
	…
}
```