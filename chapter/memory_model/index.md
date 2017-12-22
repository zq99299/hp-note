# JAVA内存模型

## volatile

* 特性：

    1. 可见性：对一个volatile变量的读，总是能看到(任意线程)对这个volatile变量最后的写入
    2. 原子性：对任意单个volatile变量的读/写具有原子性，但类似与volatile++这种复合操作不具有原子性

* 与读/写建立的happens before关系
* 写/读的内存语义
    
    当写一个volatile变量时，该操作之前的操作都将会刷新到主内存中去，包括自身
    
    当读一个volatile变量时，JMM把本地内存置为无效，直接从主内存中读取

* 内存语义的实现
    在每个volatile写操作的前面插入一个StoteStore屏障
    在每个volatile写操作的后面插入一个StoreLoad屏障
    在每个volatile读操作的后面插入一个LoadLoad屏障
    在每个volatile读操作的后面插入一个LoadStore屏障
    
    ![](/assets/memory_model/volatile-barrier.png)
    
    这里也能看出为什么会在一个volatile前后都插入屏障了，一个防止与前面发生重排序，一个防止与后面发生重排序
    
