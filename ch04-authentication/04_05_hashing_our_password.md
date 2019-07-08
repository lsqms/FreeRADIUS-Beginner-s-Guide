
#行动时间 - 哈希我们的密码
我们将在此部分中使用更安全的哈希密码AVP替换用户文件中的Cleartext-Password AVP。
关于如何创建和呈现散列密码似乎存在一般性的混淆。 我们将帮助您澄清此问题，以便为每种格式生成工作哈希值。
OpenLDAP常见问题解答是帮助我们处理哈希值的有价值的URL：
http://www.openldap.org/faq/data/cache/419.html
有几个部分展示了如何创建不同类型的密码哈希。我们可以在FreeRADIUS中将其用于我们自己的使用。
Crypt-Password
Crypt密码哈希起源于Unix计算。 尽管仍然广泛使用crypt，但更强的散列方法优于crypt。

1. 以下Perl one-liner将为passme生成一个crypt密码，其盐值为'salt'：
`#> perl -e 'print(crypt("passme","salt")."\n");'`

2. 使用此输出并更改用户文件中Alice的检查条目：“alice”
`Cleartext-Password := "passme" to: "alice" Crypt-Password := "sa85/iGj2UWlA"`

3. 以调试模式重新启动FreeRADIUS服务器。

4. 再次对其运行身份验证请求。

5. 通过在FreeRADIUS调试反馈中查找以下行，确保pap现在使用crypt密码：
`[pap] Using CRYPT password "sa85/iGj2UWlA"`

##MD5密码
MD5哈希用于检查文件的完整性。 下载Linux ISO映像时，通常还会提供该文件的MD5总和。 然后，您可以使用md5sum命令确认文件的完整性。

我们还可以从密码生成MD5哈希。 我们将使用Perl以pap模块所需的正确格式生成和编码MD5哈希。 此密码哈希的创建涉及外部Perl模块，您可能必须先安装这些模块才能使用该脚本。 以下步骤将向您展示如何：

1. 使用以下内容创建Perl脚本; 我们将它命名为4088_04_md5.pl：
```
/#! /usr/bin/perl -w
use strict;
use Digest::MD5;
use MIME::Base64;
unless($ARGV[0]){
print "Please supply a password to create a MD5 hash from.\n";
exit;
}
my $ctx = Digest::MD5->new;
$ctx->add($ARGV[0]);
print encode_base64($ctx->digest,'')."\n";
```
2. 使得4088_04_md5.pl文件可执行
`chmod 755 4088_04_md5.pl`
3. 获得MD5值
`./4088_04_md5.pl passme`
4. 使用此输出并将用户文件中的Alice条目更新为：
`“alice”MD5-Password：=“ugGBYPwm4MwukpuOBx8FLQ ==”`
5. 以调试模式重新启动FreeRADIUS服务器。
6. 再次对其运行身份验证请求。
7. 通过在FreeRADIUS调试反馈中查找以下行，确保pap现在使用MD5密码：

##SMD5密码
这是带有salt的MD5密码。 此密码哈希的创建涉及外部Perl模块，您可能必须先安装这些模块才能使用该脚本。
1. 使用以下内容创建Perl脚本; 我们将它命名为4088_04_smd5.pl：
```
/#! /usr/bin/perl -w
use strict;
use Digest::MD5;
use MIME::Base64;
unless(($ARGV[0])&&($ARGV[1])){
print "Please supply a password and salt to create a salted MD5
hash from.\n";
exit;
}
my $ctx = Digest::MD5->new;
$ctx->add($ARGV[0]);
my $salt = $ARGV[1];
$ctx->add($salt);
print encode_base64($ctx->digest . $salt ,'')."\n";
```
2. 使得4088_04_smd5.pl文件可执行：
`chmod 755 4088_04_smd5.pl`
3. 使用盐值“盐”获取密码的SMD5值：
`./4088_04_smd5.pl passme salt`
请记住，您应该使用盐的随机值。 我们这里只使用盐来进行演示。
4. 使用此输出并将用户文件中的Alice条目更新为：
`"alice" SMD5-Password := "Vr6uPTrGykq4yKig67v5kHNhbHQ="`
5. 以调试模式重新启动FreeRADIUS服务器。
6. 再次对其运行身份验证请求。
7. 通过在FreeRADIUS调试反馈中查找以下行，确保pap现在使用SMD5密码。
`[pap] Using SMD5 encryption.`

##SHA-密码
SHA代表安全散列算法。 SHA1最常用于SHA系列加密哈希函数。它由国家安全局（NSA）设计并作为其政府标准出版。 SHA-1产生160位散列值。在国家安全局不久之后，SHA-0已被SHA-0撤回，并被SHA-1取代。还有SHA-2系列，它具有SHA-1的显着变化。 SHA-2包括SHA-224，SHA-256，SHA-384，SHA-512加密功能。目前正在开发一种名为SHA-3的新哈希标准。
此密码哈希的创建涉及外部Perl模块，您可能必须先安装该模块才能使用该脚本。

1. 使用以下内容创建Perl脚本;我们将它命名为4088_04_sha1.pl：
```
/#! /usr/bin/perl -w
use strict;
use Digest::SHA1;
use MIME::Base64;
unless($ARGV[0]){
print "Please supply a password to create a SHA1 hash from.\n";
exit;
}
my $ctx = Digest::SHA1->new;
$ctx->add($ARGV[0]);
print encode_base64($ctx->digest,'')."\n";
```
2. 使得4088_04_sha1.pl文件可执行：
`chmod 755 4088_04_sha1.pl`
3. 获取passme的SHA值：
`./4088_04_sha1.pl passme`
4. 使用此输出并将用户文件中Alice的条目更新为：
`“alice”SHA-Password：=“/ waczsxHgPn1JIkpJENLNV5Jp5k =”`
5. 以调试模式重新启动FreeRADIUS服务器。
6. 再次运行authenticntcaton请求。
7. 通过在FreeRADIUS调试反馈中查找以下行，确保pap现在使用SHA密码：
`[pap] Using SHA encryption.`

##SSHA密码
这是带有salt的SHA密码。 此密码哈希的创建涉及外部Perl模块，您可能必须先安装这些模块才能使用脚本。

1. 使用以下内容创建Perl脚本; 我们将它命名为4088_04_ssha1.pl：
```
/#! /usr/bin/perl -w
use strict;
use Digest::SHA1;
use MIME::Base64;
unless(($ARGV[0])&&($ARGV[1])){
print "Please supply a password and salt to create a salted SHA1
hash from.\n";
exit;
}
my $ctx = Digest::SHA1->new;
$ctx->add($ARGV[0]);
my $salt = $ARGV[1];
$ctx->add($salt);
print encode_base64($ctx->digest . $salt ,'')."\n";
```
2. 使得4088_04_ssha1.pl文件可执行：
`chmod 755 4088_04_ssha1.pl`
3. 使用盐值“盐”获取passme的SSHA值：
`./4088_04_ssha1.pl passme salt`
请记住，您应该使用盐的随机值。 我们这里只使用盐来进行演示。
4. 使用此输出并将用户文件中Alice的条目更新为：
`“alice”SSHA-Password：=“bXUygZ + GToKwJysZyzghIEwf9tJzYWx0”`
5. 以调试模式重新启动FreeRADIUS服务器。
6. 再次对其运行身份验证请求。
7. 通过查找FreeRADIUS调试反馈中的以下行，确保pap现在使用SSHA密码：
`[pap] Using SSHA encryption.`

##NT密码或LM密码
LM-Password AVP用于存储用户密码的LM哈希。 NT-Password AVP用于存储用户密码的NTLM哈希值。 LM哈希是Windows NT之前的Microsoft LAN Manager使用的密码哈希。 NTLM 哈希是在Windows NT中引入的。

由于他们已知的flaws，现在建议不再使用它们。 flaws包含预先计算的攻击的漏洞，因为它们不使用盐。密码也是分开的。这样可以减少每个密码块的可能性，从而更容易猜测。

尽管存在漏洞，但由于许多传统的第三方CIFS实现，LM散列和NTLM散列被广泛使用。虽然未启用，但Windows Server 2008 stll包含对LM哈希的支持。

要创建NT-Password或LM-Password哈希，我们使用smbencrypt程序，该程序随FreeRADIUS一起安装。因为NT-Password哈希比LM-Password哈希更安全，所以我们将在这里使用它。

1. 使用以下命令获取密码的NT密码：
`smbencrypt passme`
2. 使用此输出并将用户文件中的Alice条目更新为：
`“alice”NT-Password：=“CED46D3B902D60F779ED78BFD90ED00A”`
3. 以调试模式重新启动FreeRADIUS服务器。
4. 再次对其运行身份验证请求。
5. 通过在FreeRADIUS调试反馈中查找以下行，确保pap现在使用NT密码：
`[pap] NT-Hash of passme = ced46d3b902d60f779ed78bfd90ed00a`
##刚刚发生了什么？
我们创建并测试了不同的哈希格式，用于在用户文件中存储用户密码。
##散列格式和身份验证协议
散列密码会对可以使用此密码的可用身份验证协议施加限制。如您所见，PAP可以与所有这些一起使用。 CHAP要求密码以明文形式存储。 MS-CHAP只能使用明文或NT密码。

>以下URL有一个很好的身份验证协议和密码加密查找网格：
http://deployingradius.com/documents/protocols/compatibility.html