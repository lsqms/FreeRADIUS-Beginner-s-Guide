# 虚拟服务器
1. 在sites-available目录中创建新的虚拟服务器。 为此新虚拟服务器配置并指定单独的SQL数据库。 将此虚拟服务器链接到启用站点(sites-available)的目录。 在client.conf文件中将VPN服务器定义为客户端，并使用virtual_server指令强制将此新虚拟服务器用于RADIUS请求。
2. sites-available目录下的buffered-sql虚拟服务器可以用作模板来解决慢速SQL响应问题。
3. 这是因为身份验证部分不包含Auth-Type PERL {...}子部分。 通常，Auth-Type将由一个模块或在authorize部分内的unlang来设置。 然后，authenticate部分需要Auth-Type的子部分来处理其值。