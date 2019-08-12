# 行动时刻 - 纳入热点欢乐时光政策

我们将使用虚拟服务器来合并Hotspot Happy Hour策略。 然后将其添加到食堂客户端定义中的接入点。 当我们将虚拟服务器应用于客户端定义时，它也可以轻松地将同一虚拟服务器与其他客户端一起使用。

## 启用欢乐时光虚拟服务器
请按照以下步骤启用服务器：
1. 在FreeRADIUS配置目录中的sites-available目录下，使用以下内容创建名为happy_hour的文件：
```
server happy_hour {
	authorize {
		files
		# If user not present allow them free access
		# between 13:00 and 14:00
		if(noop){
			update control {
				Login-Time := 'Al1300-1400'
				Auth-Type := "Accept"
			}
		}
		# April Fools' Day prank - Rickroll everyone
		update reply {
			WISPr-Redirection-URL :="http://www.youtube.com/watch?v=oHg5SJYRHA0"
		}
		logintime
		pap
	}
	authenticate {
		Auth-Type PAP {
			pap
		}
	}
}
```
2. 确保您位于FreeRADIUS配置目录中。 通过创建指向启用站点的目录的符号链接来启用happy_hour虚拟服务器：
`ln -s ../sites-available/happy_hour sites-enabled/happy_hour`

## 将虚拟服务器添加到客户端
在本练习中，我们将设想localhost客户端是食堂中的接入点。 我们将happy_hour虚拟服务器绑定到localhost客户端：
1. 编辑FreeRADIUS配置中的clients.conf文件，并在localhost客户端部分的末尾添加虚拟服务器指令：
`virtual_server = happy_hour`
2. 在调试模式下重新启动FreeRADIUS并使用radtest程序测试身份验证。 回复属性将根据一天中的时间以及用户是已知还是未知而更改。 然而今天每个人都会被碾压！
3. 您可以将happy_hour虚拟服务器中的登录时间值更改为您测试模拟热点欢乐时光的时间。
4. 完成本练习后，再次注释virtual_server指令。 这将使FreeRADIUS服务器在练习之前保持原样。

### 刚刚发生了什么？
我们在客户端部分使用了virtual_server指令来强制客户端使用虚拟服务器。
我们创建的虚拟服务器并不多。 它使用基本的unlang来满足我们的要求。 如果文件的流程没有意义，建议您重新阅读第7章授权。
> 在现实世界
> 你不应该只是在不限制每个连接的带宽的情况下运行这样的促销。 大多数强制门户网站都具有用于此目的的回复属性。 在Isaac的案例中，他可以选择通用的WISPr-Bandwidth-Max-Up和WISPr-Bandwidth-Max-Down或将specifi属性指定为Mikrotk（Mikrotik-Rate-Limit）和Coova Chilli（Chillispot-Bandwidth-Max- [Up | Down]）。

所有其他客户端仍将使用默认虚拟服务器。 只有食堂中的接入点才会使用happy_hour虚拟服务器。 此功能使新策略的初始测试运行变得非常容易。 在将这些策略投入生产之前，可以在有限的客户端上进行测试。

## 在SQL中定义客户端
如果您希望在MySQL数据库的nas表而不是clients.conf文件中定义客户端，您可能会注意到sql/mysql/nas.sql中的nas表的模式不包含默认情况下指定虚拟服务器的选项。
在nas表定义中注释掉的服务器列用于此目的。如果要使用此服务器列指定虚拟服务器，请在创建表时取消注释该行。记得还要更新sql/mysql/dialup.conf文件中的nas_query以包含服务器列：
nas_query =“SELECT id，nasname，shortname，type，secret，server FROM $ {nas_table}”
最后确保在sql.conf文件中取消注释readclients = yes行。
在本章的第一个练习中，我们将虚拟服务器绑定到监听部分。在本练习中，我们将虚拟服务器连接到客户端部分。下一个练习将探索包含在虚拟服务器定义中的客户机和侦听部分。