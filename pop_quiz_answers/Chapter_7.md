# 授权
1. 可能是NAS不支持返回的AVP以限制带宽。 AVP的单位也可能不匹配。例如，计数器期望值为Kbit / s而不是bit / s。
2. 为了提高速度，应使用Perl代替Bash。如果你使用perl模块，当FreeRADIUS启动时，Perl解释器和Perl脚本将被加载到内存中。
3. FreeRADIUS内部使用的其他属性应在字典文件中定义，该文件位于FreeRADIUS配置目录下。
4. 内部属性列表称为控制列表。要引用Auth-Type属性，您可以在条件语句中使用control:Auth-Type 在双引号或反引号字符串中使用％{control：Auth-Type}。
5. 此代码定义了一个名为rewrite_calling_station_id的策略。策略代码搜索包含以下分隔符字符的MAC地址：或 - 并将其重写为以 - 字符分隔。