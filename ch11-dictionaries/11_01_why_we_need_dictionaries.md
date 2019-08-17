# 为什么我们需要字典？
计算机最适合使用数字，而人类最适合使用姓名。我们创建了DNS以便记住主机名而不是IP地址。字典以相同的方式使用，因此我们可以记住AVP名称而不是类型编号。当FreeRADIUS解析请求或生成响应时，会查阅字典。

但是，字典与DNS不同，因为RADIUS客户端不知道FreeRADIUS使用的这些“友好”名称。永远不会在RADIUS客户端和RADIUS服务器之间交换AVP名称。 AVP名称仅由服务器使用。 radclient和radtest程序是使用与服务器相同的字典的特殊客户端，因为它们是FreeRADIUS程序套件的一部分。字典仅供本地管理员使用，具体取决于FreeRADIUS的版本。相比之下，JRadius Simulator有自己的字典，它们独立于FreeRADIUS使用的字典。

## 解析请求

当FreeRADIUS收到访问请求数据包时，数据包包括映射到用户名（1）和用户密码（2）的类型编号。数据包不包含字符串User-Name或User-Password。大多数RADIUS客户端很少需要显示表示类型编号的字符串，因为客户端不是由人类直接使用的。这意味着他们不需要字典。在人类参与使用RADIUS客户端（radclient或JRadius Simulator）的情况下，客户端将需要一些字典以显示有意义的结果。
另一方面，用户的信息以人类可读的格式存储在服务器上。有关存储在users文件中的Alice，请参阅以下条目：
```
“alice”Cleartext-Password：=“passme”
Mikrotik-Total-Limit = 10240
```
为了使涉及认证和授权的模块使用人类可读的数据，他们必须查阅字典以将人类可读的值映射到类型号。这允许我们将AVP存储为我们理解的名称，而不是计算机理解的类型编号。例如，Cleartext-Password AVP映射到数字1100，用户名映射到数字1，用户密码映射到数字2。

## 生成响应
在FreeRADIUS通过Access-Reply确定要返回的AVP的过程中，它将再次查阅字典，以便在将数据封装在RADIUS数据包中之前将应答属性名称映射到类型编号。这也允许我们以人类可理解的格式存储回复属性，而不是在RADIUS数据包内使用的类型号。
我们现在看到字典是因为对我们有帮助。在下一节中，我们将看到FreeRADIUS如何知道要使用哪些字典。