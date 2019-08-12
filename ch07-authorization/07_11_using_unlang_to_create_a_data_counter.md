# 行动时刻 - 使用unlang创建数据计数器
我们首先必须确保某些事情到位，以便这项工作取得成功。 应首先完成以下项目作为准备：
  在字典中定义自定义属性。
  创建将由FreeRADIUS perl模块使用的Perl脚本。
  更新Mikrotk和Chillispot词典。
  准备用户文件。
  准备SQL数据库。
  将unlang代码添加到虚拟服务器以充当数据计数器。
  如果存在，则识别LD_PRELOAD错误。
这项工作涉及很多工作。 采取分而治之的方法将阻止我们被淹没。 我们来解决它吧！

定义自定义属性
我们从unlang的介绍中看到，变量的主要用途是通过属性。 我们需要定义一些在数据计数器中使用的属性。

编辑FreeRADIUS配置目录下的字典文件，并添加以下属性定义：
```
ATTRIBUTE FRBG-Reset-Type 3050 string
ATTRIBUTE FRBG-Total-Bytes 3051 string
ATTRIBUTE FRBG-Start-Time 3052 integer
ATTRIBUTE FRBG-Used-Bytes 3053 string
ATTRIBUTE FRBG-Avail-Bytes 3054 string
```

每个属性都以“FRBG”开头。 这只是为了与其他属性区分开来，代表FreeRADIUS初学者指南。
RADIUS协议只能通过线路传输数值最多为255的属性。 内部使用大于255的值的属性。
下表列出了每个属性及其含义：
|Atribute|Meaning|
|:-|:-:|
|FRBG-Reset-Type|指定何时应重置计数器。 值可以是每日，每周，每月或从不。|
|FRBG-Total-Bytes|用户可以使用的数据量(以字节为单位)，也称为数据上限。|
|FRBG-Start-Time |指定计数器开始时间的Unix时间戳。|
|FRBG-Used-Bytes |从开始时间到现在使用的数据量。|
|FRBG-Avail-Bytes |仍然可用的数据量：|
||FRBG-Avail-Bytes = FRBG-Total-Bytes – FRBG-UsedBytes|

如果你观察到，你会注意到涉及字节的属性都被定义为字符串类型而不是整数类型。 我们将它作为整数类型的32位限制的变通方法。 以下小节更详细地解释了32位整数限制。

## 32位限制
RADIUS中的整数值限制为32位。 这意味着integer类型的属性的值不能超过4,294,967,295（231-1）。 为了克服此限制，RADIUS使用Gigaword属性，该属性充当32位属性的进位。 例如，Accounting-Request数据包将包含Acct-Input-Octets和Acct-Input-Gigawords，以表示大于4,294,967,295的值。

假设我们的值为8.5 GB。这可以通过以下方式使用Gigaword进行定义：
+ 8.5 GB的整数值是9,126,805,504字节（8.5 x 1024 x 1024 x 1024）。一个字节等于网络术语中的八位字节。
+ 为了计算Gigaword的价值，我们将9,126,805,504除以4,294,967,296。结果是2.125。 （9126805504/4294967296 = 2.125）。Gigaword 进位的值是2。
+ 为了计算余数，我们将Gigaword进位值乘以4,294,967,296并从原值中减去该值（9126805504  - （2 x 4294967296）= 536870912）。
+ 这意味着可以在Accounting-Request数据包中显示8.5GB，如下所示：Acct-Input-Gigawords = 2，Acct-Input-Octets = 536870912。
此32位整数限制仅适用于RADIUS协议。数据库模式已经满足大整数bigint（20）。

> 2.x版本之前的FreeRADIUS部署将需要修改以包含对Gigawords的支持。

对于32位整数限制，当值大于4,294,967,295时，使用unlang进行比较变得很困难。出于这个原因，我们将属性设置为string类型，并使用perl模块为我们执行比较。perl模块没有32位限制。将属性类型定义为字符串允许以下内容：

+ 指定大于4,294,967,295的属性值。
+ 允许Perl将字符串值转换为数值以进行计算。这克服了32位的限制。

## 使用perl模块
FreeRADIUS允许对部分进行可选命名。这使我们能够为perl模块定义各种命名部分。随后可以将每个命名的部分用作模块。定义如下：
```
perl <name> {
	...
}
```
我们将在modules目录下创建两个命名的perl节：

1. 在FreeRADIUS配置目录下的modules目录中，使用以下内容创建名为reset_time的文件：
```
perl reset_time {
	module = ${confdir}/reset_time.pl
}
```
2. 在FreeRADIUS配置目录下的modules目录中，创建一个名为check_usage的文件，其中包含以下内容：
```
perl check_usage {
	module = ${confdir}/check_usage.pl
}
```
3. 这两个perl部分每个都引用一个Perl脚本，它将由perl模块调用。确保这些Perl脚本位于FreeRADIUS配置目录中。
脚本及其内容列在以下部分中。

## reset_time.pl
reset_time.pl脚本用于执行以下操作：
+ 如果FRBG-Reset-Type的值为每日，每周或每月，则将添加FRBG-Start-Time AVP。
+ FRBG-Start-Time的值是计数器启动时的Unix时间。
+ 如果添加了FRBG-Start-Time AVP，则脚本的返回代码将具有更新值。
+ 如果未添加FRBG-Start-Time AVP（FRBG-Reset-Type = never），则脚本的返回码的值为noop。

这是reset_time.pl的内容：
```
! /usr/bin/perl -w
use strict;
use POSIX;
 use ...
 This is very important !
use vars qw(%RAD_CHECK);
use constant RLM_MODULE_OK=> 2;# /* the module is OK,
continue */
use constant RLM_MODULE_NOOP=> 7;
use constant RLM_MODULE_UPDATED=> 8;# /* OK (pairs modified) */
sub authorize {
#Find out when the reset time should be
if($RAD_CHECK{'FRBG-Reset-Type'} =~ /monthly/i){
$RAD_CHECK{'FRBG-Start-Time'} = start_of_month()
}
if($RAD_CHECK{'FRBG-Reset-Type'} =~ /weekly/i){
$RAD_CHECK{'FRBG-Start-Time'} = start_of_week()
}
if($RAD_CHECK{'FRBG-Reset-Type'} =~ /daily/i){
$RAD_CHECK{'FRBG-Start-Time'} = start_of_day()
}
if(exists($RAD_CHECK{'FRBG-Start-Time'})){
return RLM_MODULE_UPDATED;
}else{
return RLM_MODULE_NOOP;
}
}
sub start_of_month {
#Get the current timestamp;
my $reset_on = 1; #you decide when the monthly CAP will reset
my $unixtime;
my ($sec,$min,$hour,$mday,$mon,$year,$wday,$yday,$isdst)=localtim
e(time);
if($mday < $reset_on ){
$unixtime = mktime (0, 0, 0, $reset_on, $mon-1, $year, 0, 0);
#We use the previous month
}else{
$unixtime = mktime (0, 0, 0, $reset_on, $mon, $year, 0, 0);
#We use this month
}
return $unixtime;
}
sub start_of_week {
Get the current timestamp;
my ($sec,$min,$hour,$mday,$mon,$year,$wday,$yday,$isdst)=localtim
e(time);
#create a new timestamp:
my $unixtime = mktime (0, 0, 0, $mday-$wday, $mon, $year, 0, 0);
return $unixtime;
}
sub start_of_day {
Get the current timestamp;
my ($sec,$min,$hour,$mday,$mon,$year,$wday,$yday,$isdst)=localtim
e(time);
#create a new timestamp:
my $unixtime = mktime (0, 0, 0, $mday, $mon, $year, 0, 0);
return $unixtime;
}

```

## check_usage.pl
check_usage.pl脚本用于执行以下操作：
+ 添加回复属性以指定用户的可用字节。 这包括Gigaword值的计算。
+ 如果添加了回复属性，请将返回代码指定为已更新。
+ 如果数据使用超过分配部分，则通过将返回代码指定为拒绝并添加回复消息来拒绝请求。

这是check_usage.pl的内容：
```
#! usr/bin/perl -w
use strict;
# use ...
# This is very important!
use vars qw(%RAD_CHECK %RAD_REPLY);
use constant RLM_MODULE_OK=> 2;# /* the module is OK,
continue */
use constant RLM_MODULE_UPDATED=> 8;# /* OK (pairs modified) */
use constant RLM_MODULE_REJECT=> 0;# /* immediately reject the
request */
use constant RLM_MODULE_NOOP=> 7;
my $int_max = 4294967296;
sub authorize {
#We will reply, depending on the usage
#If FRBG-Total-Bytes is larger than the 32-bit limit we have
to set a Gigaword attribute
if(exists($RAD_CHECK{'FRBG-Total-Bytes'}) && exists($RAD_
CHECK{'FRBG-Used-Bytes'})){
$RAD_CHECK{'FRBG-Avail-Bytes'} = $RAD_CHECK{'FRBGTotal-Bytes'} - $RAD_CHECK{'FRBG-Used-Bytes'};
}else{
return RLM_MODULE_NOOP;
}
if($RAD_CHECK{'FRBG-Avail-Bytes'} <= $RAD_CHECK{'FRBG-UsedBytes'}){
if($RAD_CHECK{'FRBG-Reset-Type'} ne 'never'){
$RAD_REPLY{'Reply-Message'} = "Maximum $RAD_
CHECK{'FRBG-Reset-Type'} usage exceeded";
}else{
$RAD_REPLY{'Reply-Message'} = "Maximum usage
exceeded";
}
return RLM_MODULE_REJECT;
}
if($RAD_CHECK{'FRBG-Avail-Bytes'} >= $int_max){
#Mikrotik's reply attributes
$RAD_REPLY{'Mikrotik-Total-Limit'} = $RAD_CHECK{'FRBGAvail-Bytes'} % $int_max;
$RAD_REPLY{'Mikrotik-Total-Limit-Gigawords'} =
int($RAD_CHECK{'FRBG-Avail-Bytes'} / $int_max );
#Coova Chilli's reply attributes
$RAD_REPLY{'ChilliSpot-Max-Total-Octets'} = $RAD_
CHECK{'FRBG-Avail-Bytes'} % $int_max;
$RAD_REPLY{'ChilliSpot-Max-Total-Gigawords'} =
int($RAD_CHECK{'FRBG-Avail-Bytes'} / $int_max );
}else{
$RAD_REPLY{'Mikrotik-Total-Limit'} = $RAD_CHECK{'FRBGAvail-Bytes'};
$RAD_REPLY{'ChilliSpot-Max-Total-Octets'} = $RAD_
CHECK{'FRBG-Avail-Bytes'};
}
return RLM_MODULE_UPDATED;
}
```

由于Isaac使用Mikrotk和Coova Chilli，我们将其AVP用于限制总数据。 请参阅NAS的文档，以确定它是否支持数据限制以及应该使用哪些AVP。

## 在CentOS上安装perl模块
FreeRADIUS perl模块在CentOS中单独打包。 确保已安装此软件包。 SUSE和Ubuntu已经包含了带有FreeRADIUS标准安装的perl模块，尽管还有一个bug，我们将在以后删除它。

## 更新字典文件
FreeRADIUS包含各种供应商的字典文件。 我们希望返回的属性是这些供应商后来开发的一部分，并不包含在FreeRADIUS标准的字典文件中。 我们必须通过将它们添加到现有字典文件中来包含它们。 供应商的字典文件通常位于/usr/share/freeradius目录下。 如果您使用configure，make，make install patern安装了FreeRADIUS，它将位于/ usr/local/share/freeradius下。
找到供应商词典的位置，并将以下属性添加到相应的词典：
+ dictionary.chillispot
ATTRIBUTE ChilliSpot-Max-Input-Gigawords 21 integer
ATTRIBUTE ChilliSpot-Max-Output-Gigawords 22 integer
ATTRIBUTE ChilliSpot-Max-Total-Gigawords 23 integer
+ dictionary.mikrotik
ATTRIBUTE Mikrotik-Total-Limit 17 integer
ATTRIBUTE Mikrotik-Total-Limit-Gigawords 18 integer

我们刚刚进行的更改是针对供应商（VSA）的属性，并且可以通过线路在RADIUS数据包内传输。 VSA与我们之前添加到FreeRADIUS服务器内部使用的主字典文件中的属性不同。
请记住，使用最新版本的Mikrotk和Coova Chilli将确保支持这些属性。另请参阅是否有需要包含的最新版本引入的附加属性。
### 更新词典的推荐方法
本练习不遵循建议的更新字典文件的方法。本书后面有一个关于词典的专门章节，其中介绍了使词典保持最新的推荐方法。
在启动期间包含字典的方式也在字典一章中有更详细的介绍。简而言之，FreeRADIUS配置目录下的主字典文件包含一个名为/usr/share/freeradius/dictionary的文件。这个文件依次包括位于/usr/share/freeradius/下的各种.dictionary文件。
## 准备用户文件
我们将用户文件用作用户存储。在现实世界中，您可以使用其他来源，例如SQL数据库或LDAP目录。
确保用户文件包含以下alice条目：
```
"alice" Cleartext-Password := "passme",FRBG-Total-Bytes
:='9126805504',FRBG-Reset-Type := 'monthly'
Reply-Message = "Hello, %{User-Name}"
```
这将限制每月数据使用量为8.5 GB。

## 准备SQL数据库
虽然我们在users文件中定义了alice，但我们将使用SQL进行记帐。请按照以下步骤准备数据库：
1. 确保您具有第5章“用户名和密码的来源”中指定的有效SQL配置。
2. 我们不会将SQL用作用户存储。确认在FreeRADIUS配置目录下的sites-enabled / default文件中的authorize部分内禁用（注释掉）sql。
3. 为了使我们的数据计数器成功，确保在FreeRADIUS配置目录下的sites-enabled / default文件的accounting部分内启用（取消注释）sql。
4. 使用mysql客户端程序并清除数据库中可能仍存在的任何以前的计费详细信息：
```
$>mysql -u root -p radius
delete from radacct;
```
5. 在本练习的后面，我们将使用radclient程序模拟计费。确保创建第6章“计费”中指定的4088_06_acct_start.txt和4088_06_acct_stop.txt文件。
您可能担心sql模块也具有32位整数限制。
幸运的是，在新版本的FreeRADIUS中已经完成了这一切。在更新SQL数据库之前，FreeRADIUS负责将Gigaword进位AVP与八位位组AVP相结合。 MySQL的SQL模式还使用bigint（20）来存储八位字节值，这足以存储大数字。

## 将unlang代码添加到虚拟服务器

到目前为止的所有准备工作都是为了让我们将下一段unlang代码添加到虚拟服务器定义中。在FreeRADIUS配置目录下的sites-enabled / default文件中，在authorize部分的每日条目下面添加以下代码。
```
if((control:FRBG-Total-Bytes)&&(control:FRBG-Reset-Type)){
reset_time
if(updated){ # Reset Time was updated,
# we can now use it in a query
update control {
#Get the total usage up to now:
FRBG-Used-Bytes := "%{sql:SELECT
IFNULL(SUM(acctinputoctets - GREATEST((%{control:FRBGStart-Time} - UNIX_TIMESTAMP(acctstarttime)), 0))+
SUM(acctoutputoctets -GREATEST((%{control:FRBG-Start-Time}
- UNIX_TIMESTAMP(acctstarttime)), 0)),0) FROM radacct WHERE
username='%{request:User-Name}' AND UNIX_TIMESTAMP(acctstarttime) +
acctsessiontime > '%{control:FRBG-Start-Time}'}"
}
}
else{
#Asumes reset type = never
#Get the total usage of the user
update control {
FRBG-Used-Bytes := "%{sql:SELECT IFNULL(SUM(ac
ctinputoctets)+SUM(acctoutputoctets),0) FROM radacct WHERE
username='%{request:User-Name}'}"
}
}
#Now we know how much they are allowed to use and the usage.
check_usage
}
```
## SUSE和Ubuntu BUG
如果FreeRADIUS使用perl模块，SUSE和Ubuntu中有一个错误会导致BUG。 是时候解决了！
在调试模式下重新启动FreeRADIUS服务器。 虽然听起来很奇怪，如果一切顺利，你应该得到类似如下的错误信息：
`/usr/sbin/radiusd: symbol lookup error: /usr/lib/perl5/5.10.0/i586-linux-thread-multi/auto/Fcntl/Fcntl.so: undefined symbol: Perl_Istack_sp_ptr`
这是因为动态加载所需的perl模块存在问题。

> 如果您没有收到错误，可能是因为您安装的FreeRADIUS没有报告此问题。 例如，如果您使用configure，make，make install模式自行编译FreeRADIUS，则不应出现此错误。

### 预加载Perl库
为了克服动态加载问题，我们首先必须在启动FreeRADIUS服务器之前设置LD_PRELOAD环境变量。 不幸的是，要预加载的库的位置和名称在每个分发上都是不同的。
我们必须指定libperl.so库。 以下命令将帮助我们准确定位和确定在您的发行版上调用此库的内容。 如果结果产生多个，则它们通常是指向单个库的各种符号链接。
$> find / -name“* libperl.so *”
使用下表作为参考：
|Distributon|Locaton of libperl.so|
|:-|:-:|
|SLES|/usr/lib/perl5/5.10.0/i586-linux-thread-multi/CORE/libperl.so|
|Ubuntu|/usr/lib/libperl.so|

> 了解环境变量
> Wikipedia将环境变量定义如下：
> http://en.wikipedia.org/wiki/Environment_variable
> 环境变量是一组动态命名值，它们可以影响运行进程在计算机上的行为方式。从某种意义上说，它们可以创建一个进程运行的操作环境。

## 测试数据计数器
真相的时刻到了。 是时候测试数据计数器了。 如果您使用SUSE或Ubuntu，请记住LD_PRELOAD。 CentOS没有这个问题。
1. 在调试模式下重启FreeRADIUS; 如果需要，设置LD_PRELOAD环境变量。 我们在这里假设Ubuntu。 请更改以适合您的分发：
`>LD_PRELOAD=/usr/lib/libperl.so /usr/sbin/freeradius -X`
2. 尝试使用radtest命令以alice身份验证：
`$> radtest alice passme 127.0.0.1 100 testing123`
3. 您应该在FreeRADIUS的回复中获得以下AVP：
```
ChilliSpot-Max-Total-Gigawords = 2
ChilliSpot-Max-Total-Octets = 536870912
Mikrotik-Total-Limit-Gigawords = 2
Mikrotik-Total-Limit = 536870912
Reply-Message = "Hello, alice"
```
4. 使用radtest结合4088_06_acct_start.txt和4088_06_acct_stop.txt文件模拟一些会计：
```
$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt
$> radclient 127.0.0.1 auto testing123 -f 4088_06_acct_stop.txt
```
5. 尝试使用radtest命令再次作为alice进行身份验证。您现在应该在回复中显示不同的值，显示剩余数据是如何耗尽的：
```
ChilliSpot-Max-Total-Gigawords = 2
ChilliSpot-Max-Total-Octets = 536866638
Mikrotik-Total-Limit-Gigawords = 2
Mikrotik-Total-Limit = 536866638
```
6. 重复会计模拟和认证测试周期几次。请注意每个周期如何减少可用字节。

## 清理
要以适当的方式完成此练习，您可以执行以下操作作为挑战：
  在policy.conf文件中定义策略。 内容将是我们添加到默认文件的unlang代码。 用新创建的策略替换该代码。
  将LD_PRELOAD环境变量添加到SUSE和Ubuntu上的FreeRADIUS启动脚本。