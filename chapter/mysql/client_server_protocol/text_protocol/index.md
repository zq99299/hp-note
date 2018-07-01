# 文本协议

> https://dev.mysql.com/doc/internals/en/text-protocol.html

也就是sql查询什么的命令包协议；

这里已COM_QUERY协议简述下交互流程


## COM_QUERY
> https://dev.mysql.com/doc/internals/en/com-query.html

请求包，很简单，除了通用的包头外,就下面这两个参数了 
```
Payload
1              [03] COM_QUERY
string[EOF]    the query the server shall execute
```

这里需要注意的是，string.EOF,就是需要你得到string的长度然后截取字符串就行了,发送查询sql的包构建就是这样

```java
execSQL("SELECT * FROM employee");

    public void execSQL(String sql) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        buffer.position(3);
        buffer.put((byte) 0);
        buffer.put((byte) 3);
        // string.eof
        // 如果字符串是数据包的最后一个组件，那么它的长度可以从总体包长度减去当前位置来计算。
        // 也就是说 只要是这个类型的。直接获取到包头然后 - 1个字节的命令类型 就得到长度了
        // .eof与其他类型的不同。不需要修饰
        buffer.put(sql.getBytes());
        int position = buffer.position();
        buffer.position(0);
        BufferUtil.writeInt(buffer, position - 4);
        buffer.position(position);
        buffer.flip();
        write(buffer);
    }
```

难的在于响应包，代码有点多，`cn.mrcode.newstudy.hpbase.mysql.mymysql2.MySqlConnectHandler#handler` 中；

大致的步骤如下：
* 第一个包：列个数；从sql中select xxx,xxx from 中的列个数
* 第二部分：列定义，每个列定义为一个包
* 第三部分：列定义后面跟随一个eof包
* 第四部分：返回值，一行一个包；行内字段顺序和前面的顺序一致
* 第五部分：结果后面跟随一个eof包

判定包是ok包还是err包的思路和之前的一样：
```
data[4] & 0xFF == 0xFF ErrPacket
```

看上面的描述，在一个正常的sql结果返回，其实分三个阶段；
* 第一阶段 ： 拿到当次sql语句所需要的列个数
* 第二阶段 ： 解析列的定义，里面包含了原始/虚拟列相关的各种信息
* 第三阶段 ： 结果集；相对来说比较简单

最简单直观的理解，就是找查看该页的说明文档，并且用抓包工具抓包3306端口，查看查询响应包的结构；会很清晰