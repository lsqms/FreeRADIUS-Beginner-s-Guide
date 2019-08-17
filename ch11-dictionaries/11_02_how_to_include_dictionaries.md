# 如何包含词典
Isaac有几个MikroTik RouterBOARD(路由器)，他希望将每个会话的总字节数（发送和接收）限制为10 MB。 一位朋友告诉他只需在回复中添加Mikrotik-Total-Limit AVP。 让我们跟随Isaac实施这一目标。
我们假设本章中有一个干净的FreeRADIUS安装。
RouterBOARD是MikroTik制造的硬件名称。 有各种RouterBOARD型号可供选择。 该硬件由MikroTik的RouterOS软件提供支持。