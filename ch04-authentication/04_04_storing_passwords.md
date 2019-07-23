
## 存储密码
用户名和密码组合必须存储在某处。以下列表提到了一些受欢迎的地方：

+ 文字：你现在应该熟悉这种方法。
+ SQL数据库：FreeRADIUS包含与SQL数据库交互的模块。 MySQL非常受欢迎，并且广泛用于FreeRADIUS。
+ 目录：Microsof的Actve目录或Novell的电子目录是典型的企业级目录。OpenLDAP是一种流行的开源替代方案。

FreeRADIUS可以使用的用户文件和SQL数据库将用户名和密码存储为AVP。当此AVP的值是明文时，如果错误的人抓住它，则可能是危险的。让我们看看如何最大限度地降低这种风险。

## 哈希格式
为降低此风险，我们可以将密码存储为散列格式。密码的散列格式类似于该密码的文本值的数字指纹。有许多不同的方法来计算此哈希值，例如MD5或SHA1。散列的最终结果应该是唯一的固定长度加密字符串，它唯一地表示密码。从哈希中检索原始密码应该是不可能的。

为了使哈希更加安全并且对字典攻击更具免疫力，我们可以在生成哈希的功能中添加一个盐。 盐是随机生成的比特，用于组合密码作为单向散列函数的输入。 使用FreeRADIUS，我们将盐与哈希一起存储。 因此，每个哈希都有一个随机盐来制作一个彩虹表，这是很有意义的。 pap模块用于PAP身份验证，可以使用以下哈希格式存储的密码来验证用户身份：

Hash format 						|AVP name
Unix-style crypted password 		|Crypt-Password
MD5 hashed password 				|MD5-Password
MD5 hashed password with a salt 	|SMD5-Password
SHA1 hashed password 				|SHA-Password
SHA1 hashed password with a salt 	|SSHA-Password
Windows NT hashed password 			|NT-Password
Windows Lan Manager (LM) password 	|LM-Password

MD5和SSH1哈希函数都可以与salt一起使用，以使其更安全
