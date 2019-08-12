# 实施限制
授权，实质上是关于限制。 基于某些检查，用户可能受到限制。 限制可以在以下两个地方之一中应用：

+ 在RADIUS服务器上
+ 在NAS

当访问请求数据包发送到RADIUS服务器时，在身份验证过程中确定限制。 Accounting-Request数据包不会也不能确定限制。

当在RADIUS服务器上应用限制时，服务器返回一个Access-Reject数据包，该数据包应该包含一个Reply-Message AVP，用于指定拒绝原因。

当在NAS上应用限制时，RADIUS服务器返回一个Access-Accept数据包，其中包含应由NAS应用的AVP。 这意味着您必须确保NAS接收到正确的AVP以实现限制，并且它首先也支持这些AVP。