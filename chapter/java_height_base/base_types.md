# 基本数据类型

> [官网解释点这里](https://zq99299.gitbooks.io/java-tutorial/content/content/java/nutsandbolts/datatypes.html)

按类型分：

1. 字符类型：char
2. 布尔类似：boolean
3. 数值类型：byte、short、int、long、float、double

按长度分：

1. 未明确定义：boolean
2. 8位（1byte）：byte
3. 16位：char、short
4. 32位：int、float
5：64位：long，double

上面的8中类型，除去boolean和char外，其他的都是有符号的，范围大小是负数~正数。char数据类型是一个16位Unicode字符。它的最小值为'\u0000'（或0），最大值为'\uffff'（或65,535）。

## 位运算
位运算常用在 网络通信、协议解析、高性能编程中。

> **左移操作**


