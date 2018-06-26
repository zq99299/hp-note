# 通用响应数据包

对于客户端发送给服务器的大多数命令，服务器会返回其中的一个数据包作为响应;

* OK_Packet
* ERR_Packet
* EOF_Packet

对于在文档中看到有 `capabilities & CLIENT_TRANSACTIONS` 类似的描述你需要了解以下知识：

* capabilities 能力集合
* | 或运算符和 & 与运算符

## capabilities 

> 能力标识 https://dev.mysql.com/doc/internals/en/capability-flags.html

标识了一些能力值，这些值表示支持某些功能，或则某种规则
## 运算符

```
a |= CLIENT_TRANSACTIONS     表示能力相加
if(capabilities & CLIENT_TRANSACTIONS) 表示 if(capabilities & CLIENT_TRANSACTIONS == CLIENT_TRANSACTIONS)
也就是表示如果你的能力集中存在指定的能力，那么你就应该按照具体的能力规则进行操作 
```

