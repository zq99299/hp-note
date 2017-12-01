# Java常用排序算法

http://blog.csdn.net/daogepiqian/article/details/50770404


插入排序
```java
 int[] a = {49, 38, 65, 97, 76, 13, 27, 49, 78, 34, 12, 64, 1};
        System.out.println(Arrays.toString(a));

        for (int i = 1; i < a.length; i++) {
            int temp = a[i]; // 待插入元素
            // 扫描的是已排序的，第一次假设第一个元素是有序的
            // 那么扫描有序序列是倒序
            int k;
            for (k = i - 1; k >= 0; k--) {
                //
                // 待插入的值和已排序的最后一个比较，如果大于就让这个值往后移动
                // 然后继续比较已排序倒数第二个，如果相等则退出当次比较，然后在前一次的位置上插入需要插入的元素
                // 假设 3，2，1，4
                // 第一次：3 > 2 ？是的，把3往后移动,3,3,1,4 ;退出比较，然后把插入的值写入原来3所在的索引
                if (a[k] > temp) {
                    a[k + 1] = a[k];
                } else {
                    break;
                }
            }
            a[k + 1] = temp;
        }

        System.out.println(Arrays.toString(a));
```