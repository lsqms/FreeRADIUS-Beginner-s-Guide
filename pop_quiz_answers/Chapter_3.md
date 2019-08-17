# clients.conf
1. 每个客户端部分在指示部分名称的关键字之间有一个简短的描述性名称，例如，本例中的客户端和左括号。
2. 不推荐这样做，因为它具有安全隐患。
3. 有一个名为dynamic-clients的虚拟服务器，可用作处理未知IP地址的客户端的模式。
4. 消息验证器(Message-Authenticator)可能丢失。 将require_message_authenticator指令设置为no以补偿此情况。
5. 是的，ipv6addr指令用于指定IPv6地址。
6. 错误，更多字符会使它更安全。 还要避免可识别的单词。
7. FreeRADIUS完成的同时使用检查可能不准确，即使用户受到限制，也允许用户进行多次会话。