# 行动时刻 - 包括新词典
以下步骤将演示如何包含新目录：

1. 编辑位于FreeRADIUS配置目录下的用户文件，并确保以下alice条目存在：
```
“alice”Cleartext-Password：=“passme”
Mikrotik-Total-Limit = 10240
```
2. 在调试模式下重新启动FreeRADIUS。重启应该不成功，将显示以下错误消息：
```
/etc/raddb/users[11]: Parse error (reply) for entry alice: Invalid
octet string "10240" for attribute name "Mikrotik-Total-Limit"
Errors reading /etc/raddb/users
```
3. 错误没有明确说明AVP不在任何字典中，但是如果你在/ usr / share / freeradius目录中对这个AVP进行快速grep，它将返回空状态，表明我们确实遇到了问题：
`grep -lr'Mikrotik-Total-Limit'/usr/share/freeradius/*` 

## 刚刚发生了什么？
我们发现Mikrotik-Total-Limit AVP不包含在FreeRADIUS附带的预定义词典中。在我们尝试修复损坏的服务器之前，有一些重点可以在这里讨论。