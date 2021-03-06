# 基本数据类型
常用的数据类型 Integers 和 Strings

## integer
分为两种 固定长度整数类型 和 长度编码类型

### 固定长度整数类型 `INT<1、2、3、4、6、8>`

如`INT<3>`的 数值1，存储的bits为`01 00 00` ; 使用的是16进制标识法;

**如何读取，在java中：**
在一个bytes[]中，要读取（还原）的话需要使用位运算：

```
bytes[0] & 0XFF  
|  (bytes[1] & 0XFF) << 8
|  (bytes[2] & 0XFF) << 16
```

**如何写入，在java中**
```
int a = 0;
buffer.put((byte) a );
buffer.put((byte) (a  >>> 8));
buffer.put((byte) (a  >>> 16));
```



 `|` 或运算相当于加法； 注意这里的数值，是无符号整数，所以在对应java中的数据类型的时候，要注意精度问题


### 长度编码类型 `int<lenenc>`
根据数值的大小，使用1,3,4或9个字节（byte）的整数； 为什么是1，3，4，9 注意看下面的规则：

byte无符号最大整数为 255; 8字节标识一个long；所以在mysql中integer最大单位对应java中的long

* 如果它是0xfb (251)，则将其视为一个1字节的整数。
* 如果它是0xfc (252)，它后面跟着一个2字节的整数。
* 如果它是0xfd (253)，则后跟一个3字节的整数。
* 如果它是0xfe (254)，它后面跟着一个8字节的整数。

比如： 

```
fa  -  250 
fc fb 00  -  251   ; fc = 252; fb + 00 = 251
```
## string

* `string<fix>` 固定长度
* `string<NUL>` 以`[00]`字节结尾的字符串。
* `string<var>` 字符串的长度由另一个字段确定或在运行时计算
* `string<lenenc>` 长度编码类型，与Int类型的长度编码类型一致
* `string<EOF>` 如果一个字符串是数据包的最后一个组件，它的长度可以从总包长度减去当前位置来计算。

## `string<lenenc>` 长度编码类型
```
length （ int <lenenc>） - 字符串的长度

string （string<fix>） - [len = $length]字符串
```