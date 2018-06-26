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
    * 字符集
    * sequenceId 对于这个ID我现在还不能理解；握手包中的值为0，如果构造验证包也为0的话，将会造成协议异常，mysql服务器不能解析
    
**需要注意的是：**NIO中，读事件，不像平时练习那样，没有数据读事件不会进入就绪集合，这里却会，而且读取数据的话会返回 -1；表示到了流结尾； 但是实际实验看来，就算是读到了-1，也还可以使用该通道进行正常的数据交互，所以这里的-1也不是平常的概念，有可能以后还需要深入理解下NIO的知识点