# 连接阶段

简述下JAVA中的连接-验证的过程：

* NIO建立连接事件

    ```java
    SocketChannel channel = SocketChannel.open();
    channel.configureBlocking(false);
    channel.register(selector, SelectionKey.OP_CONNECT);
    channel.connect(new InetSocketAddress(host, port));
    ```
* 连接完成

    ```java
     if (channel.isConnectionPending()) {
     channel.finishConnect();
    ```
* 会收到服务端发送来的握手包/问候包  Initial Handshake Packet
* 客户端收到后，进行解析
* 解析完成后，构造验证包 Handshake Response Packet
    
    构造过程中会用到问候包中的一部分数据；比如：
    * 密码的加密（SHA1）需要用到authPluginDataPart1 + authPluginDataPart2
    * 字符集 ： 注意：这个字符集不是从握手包中拿的，表示的是 你需要什么样的字符集，比如你的数据库是utf-8的，那么必须这里指定为utf-8的，否则会出现异常；同样后续的解析中，有可能会用到该字符集
    * sequenceId 对于这个ID我现在还不能理解；握手包中的值为0，如果构造验证包也为0的话，将会造成协议异常，mysql服务器不能解析
    
**需要注意的是：** 如果在实现中出现了 读到了 -1 的流，那么查看mysql的响应是否有err包，如果有的话，有可能是你的nio的写没有控制好，在写乱七八糟的数据