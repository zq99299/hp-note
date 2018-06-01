# IO全接触

## IO里的大小头
首先要明白什么是大小头？

在一个32位的cpu中“字长”为32个bit，4个byte；在这样的cpu中，总是以4字节对齐的方式来读取或写入内存， 写入内存的时候就存在byte字节的存储顺序

多字节数据在内存中存储的时候，有一个字节顺序问题，和不同的硬件有关；称之为 big endian 和 little endian

 数字 0x1234556在大小头中是下面这样存储的

1. Big endian

    在SUN、IBM的cpu上存储的顺序是这样的；
    
    ```
    | 12 | 34 | 56 | 78      <- 高地址
    ```
    
2. Little Endian x86架构的CPU

    ```
    | 78 | 56 | 34 | 12      <- 高地址
    ```
    
Java IO包中的Bits类,默认使用的是大头
```java
/**
 * Utility methods for packing/unpacking primitive values in/out of byte arrays
 * using big-endian byte ordering.
 */
 
     static void putInt(byte[] b, int off, int val) {
        b[off + 3] = (byte) (val       );
        b[off + 2] = (byte) (val >>>  8);
        b[off + 1] = (byte) (val >>> 16);
        b[off    ] = (byte) (val >>> 24);
    }
```

上面这个是大头的 int 存储为byte；怎么看待 (byte)val（低位） 存储在 索引3(高地址)？
反过来看：val >>> 24 ;剩下的是8个高位，存储在了 索引0位置， 高位存储在低位；

上面二进制来看，0x12345678;假设右移，0x12 剩下的是高位；存储在底地址上

![](/chapter/java_height_base/io/大头小头原理示意图.png)

为什么大头模式高位是放在地址上面的？cpu开始读取起始地址+长度，还是和cpu有硬件有关； 

`java.nio.ByteOrder` 类告诉你了怎么能得到到底是大头还是小头的判定


下面的api使用可以去测试下 怎么用的
```java
  ByteBuffer buffer = ByteBuffer.allocate(100);
  buffer.order(ByteOrder.BIG_ENDIAN).getInt();
````

## 装饰器模式：
![](/assets/java_height_base/02/装饰器模式.png)

* Component：组件对象的接口，可以给这些对象动态的添加职责
* ConcreteComponent：具体的组件对象，实现组件对象接口，通常就是被装饰器装饰的原始对象，也就是可以给这个对象添加职责。
* Decorator：所有装饰器的抽象父类，需要定义一个与组件接口一致的接口，并持有一个component对象，其实就是持有一个被装饰的对象。	注意这个被装饰的对象不一定是最原始的那个对象了，也可能是被其他装饰器装饰过后的对象，反正都是实现的同一个接口，也就是同一类型。
* ConcreteDecorator：实际的装饰器对象，实现具体要向被装饰对象添加的功能。

**装饰模式的本质是：** 动态组合【重点是组合】
何时选用：      

1. 如果需要在不影响其他对象的情况下，以动态、透明的方式给对象添加职责，可以使用装饰模式，这几乎就是装饰模式的主要功能。       
2. 如果不合适使用子类来进行扩展的时候，可以考虑使用装饰模式，因为装饰模式是使用“对象组合”的方式。	所谓不适合用子类扩展的方式，比如：扩展功能需要的子类太多，造成子类数码呈爆炸性增长。

## NIO的ByteBuffer
在nio中最重要的两个组件就是缓冲Buffer和通道Channel。

* Buffer ： 一块连续的内存块，是NIO读写数据的中转地
* Channel : 表示缓冲数据的源头或则目的地，用于向缓冲读取或则写入数据，是访问缓冲的接口。

ByteBuffer 只是Buffer的其中一个子类，因为它多用于绝大多数标准I/O接口，所以比较常用；

Buffer参数的含义：

| 参数 | 写模式 | 读模式
|------|--------|-----
| 位置 position | 当前缓冲区的位置，将从position的下一个位置写数据 | 当前缓冲区读取的位置，将此位置后读取数据
| 容量 capactiy | 缓冲区的总容量大小 | 缓冲区的总容量上限
| 上限/界 limit | 缓冲区的实际上限，它总是小于等于容量，通常情况下，和容量相等 | 代表可读取的总容量，和上次写入的数据量相等
| 标记 mark | - | -

使用下面的一段测试程序来理解buffer的各个参数的含义

```java
@Test
public void fun2() {
    ByteBuffer buffer = ByteBuffer.allocate(15); // 分配一个15字节的缓冲区
    fun2println(buffer);
    IntStream.range(0, 10).forEach(i -> buffer.put((byte) i)); // 存入十个字节
    fun2println(buffer);
    buffer.flip(); // 重置位置为0，limit 变成存入的十个字符上限，标识有10个位置的数据只有效数据
    fun2println(buffer);
    // 上面因为重置了buffer的位置为0，所以这里读取5个是从头开始读取的
    IntStream.range(0, 5).forEach(i -> System.out.print(buffer.get() + " "));
    System.out.println("");
    fun2println(buffer);
    buffer.rewind(); // 重置位置为0 ，不改变limit
    fun2println(buffer);
    IntStream.range(0, 5).forEach(i -> System.out.print(buffer.get() + " "));
    System.out.println("");
    fun2println(buffer);
    buffer.flip();// 重置，因为上面读取了5个字符，当前位置等于5.重置的时候limit就会变成5.标识只有5个有效数据可读
    fun2println(buffer);
    // 使用hasRemaining api可以验证上面的说法，只会打印5个字符
    // 但是数据一直都在，重置操作只是重置了 标识位
    while (buffer.hasRemaining()){
        System.out.print(buffer.get() + " ");
    }
}

private void fun2println(ByteBuffer buffer) {
    System.out.println(MessageFormat.format("limit={0},capactiy={1},position={2}", buffer.limit(), buffer.capacity(), buffer.position()));
}
```

这是程序输出
```
limit=15,capactiy=15,position=0
limit=15,capactiy=15,position=10
limit=10,capactiy=15,position=0
0 1 2 3 4 
limit=10,capactiy=15,position=5
limit=10,capactiy=15,position=0
0 1 2 3 4 
limit=10,capactiy=15,position=5
limit=5,capactiy=15,position=0

0 1 2 3 4
```
按步骤一步一步看，然后对比输出，就能理解了；

上面的两个api都是重置的那么有什么区别呢？

| - | rewind() | clear() | flip() | mark()|
|---|-----------|---------|--------|-----|
| position | 置0 | 置0 | 置0 | -
| mark | 清空 | 清空 | 清空 | 设置为postion
| limit | 未改动 | 设置为capacity | 设置为position | -
| 作用 | 为读取Buffer中有效数据做准备 | 为重新写入Buffer做准备 | 在读写切换时调用 | 标记备忘位置

看下面的例子，来理解mark的作用

```java
@Test
public void fun3() {
    ByteBuffer buffer = ByteBuffer.allocate(15); // 分配一个15字节的缓冲区
    IntStream.range(0, 10).forEach(i -> buffer.put((byte) i)); // 存入十个字节
    buffer.flip(); // 重置，切换为读取模式

    fun2println(buffer);
    IntStream.range(0, 5).forEach(i -> {
        System.out.print(buffer.get() + " ");
        if (i == 3) buffer.mark();// 在索引3处标记
    });
    System.out.println("");
    fun2println(buffer);
    buffer.reset();  // 回到上次标记的标志位置,limit 被标志为10
    fun2println(buffer);
    while (buffer.hasRemaining()) {
        System.out.print(buffer.get() + " ");
    }
}
```
程序输出

```
limit=10,capactiy=15,position=0
0 1 2 3 4 
limit=10,capactiy=15,position=5
limit=10,capactiy=15,position=4
4 5 6 7 8 9 
```

可以看出 mark 和 reset可以作为组合使用；打标记，回到标记处

支持的操作分类
### 创建缓冲区
* allocate(int) 在堆上开辟空间
* wrap(arr) 使用自定义提供的数组来存储数据；更改提供的数组也会影响buffer的数据
* Buffer.allocateDirect(int capacity) 分配一个直接缓冲区，堆外内存

### 基础操作
* 存 put() 
    
     可以指定位置存；使用索引操作，不会改变位置属性
* 取 get()
     
     可以指定位置取/读；使用索引操作，不会改变位置属性
     hasRemaining() ： 可以配合该函数是否到了缓冲区的上界位置
     remaining() ：当前位置到上界剩余的元素个数
* 翻转 flip() 

    等同于buffer.limit(busser.position).position(0)
* 压缩 compact()

    释放一部分操作，让buffer更紧凑；丢弃已经释放的数据，保留未释放的数据，并使缓冲区对重新填充容量做准备
* 标记 mark()
    
    与以前的流中的标记功能类似，使用reset()可以回到该位置
* 比较 equals() 和 compareTo
   
    除了常规的引用等判定之外,再判定数据是否完全相等；但是会先判定limit - position 是否相等，首先是个数相等；
* 批量移动 也就是get和put的批量操作版本

### 视图操作
共享数据元素、拥有同样的容量、但每个缓冲区拥有各自的位置，上界和标记属性、对缓冲区内的元素做改变会反映到另外一个缓冲区

* duplicate()  创建一个与原始缓冲区相似的新缓冲区
* asReadOnlyBuffer() 生成一个只读缓冲区，和duplicate类似。只是是只读的    
* slice() 分割缓冲区
    分割的数量是 limit-position; 也就是从position开始到limit结束；
    与上面的类似；只是共享部分数据
* asXXXBuffer() 相关的方法返回指定的基础类型的buffer

## NIO的MappedByteBuffer 文件映射到内存

```java
    @Test
    public void fun4() throws IOException {
        RandomAccessFile raf = new RandomAccessFile("resources/_02/q04/mbb", "rw");
        FileChannel channel = raf.getChannel();
        MappedByteBuffer mbb = channel.map(FileChannel.MapMode.READ_WRITE, 0, raf.length());// 把文件的所有内容都映射到内存中

        while (mbb.hasRemaining()) {
            System.out.print(mbb.get());
        }

        // 改写buffer中的值，但是要注意，这里的索引不能大于 channel.map中映射的大小
        mbb.putChar(0, '1');
        channel.close();
        raf.close();
        // 这里很奇怪的问题是，在idea中没有看到文件有变化，但是用其他工具打开文件，的确被改写了
    }
```

只能看出来MappedByteBuffer的使用和Buffer使用类似，但是具体的使用场景是什么真搞不清楚

## 官方的字节字符解码器
`sun.nio.cs.StreamDecoder`  `sun.nio.cs.StreamEncoder`

大概看了下代码 ，没有看到怎么从字节转换到字符的。要调试去深入研究？
