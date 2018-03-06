# IO全接触

## IO里的大小头
首先要明白什么是大小头？

多字节数据在内存中存储的时候，有一个字节顺序问题，和不同的硬件有关；称之为 big endian 和 little endian

1. Big endian

    在SUN、IBM的cpu上存储的顺序是这样的；
    
    ```
    | 12 | 34 | 56 | 78      <- 高地址
    ```
2. Little Endian x86架构的CPU

    ```
    | 78 | 56 | 34 | 12      <- 底地址
    ```
    
Java IO包中的Bits类,默认使用的是大头
```java
/**
 * Utility methods for packing/unpacking primitive values in/out of byte arrays
 * using big-endian byte ordering.
 */
```