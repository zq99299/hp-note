# MyCat2中用到的stream api精选
## flatMap 展开流
```java
> 来源：io.mycat.mycat2.MycatSession#getBackendConCounts

public int getBackendConCounts(MySQLMetaBean metaBean) {
	return (int)backendMap.values()
		.stream()
		.flatMap(f->f.stream())
		.filter(f->f.getMySQLMetaBean().equals(metaBean))
		.count();
}

// backendMap.values() 的类型是 Collection<List<MySQLSession>>
```
flatMap 作用：由于现在你手里是一堆的list，你要遍历这一堆的list怎么办？使用flatMap把这一堆的list转变成一个stream（这里只是表象），实际上是把各个list转成了流，当一个list的流使用完成之后，再接着使用下一个；

分清楚中间操作和终端操作：看返回值！！
```java
上面的flatmap 和 filter 操作都返回了 new StatelessOp<P_OUT, P_OUT>，而该类继承了以下类的声明
 abstract class ReferencePipeline<P_IN, P_OUT>
        extends AbstractPipeline<P_IN, P_OUT, Stream<P_OUT>>
        implements Stream<P_OUT> ；
 可见是一个流；
再来看.count的操作；

    @Override
    public final long count() {
        return mapToLong(e -> 1L).sum();
    }
	
	mapToLong 同样是返回的 LongStream，而 sum 则是调用 具体的统计操作
    @Override
    public final long sum() {
        // use better algorithm to compensate for intermediate overflow?
        return reduce(0, Long::sum);
    }
```

这里是一个例子
```java
@org.junit.Test
    public void fun1() {
        List<List<Integer>> lists = new ArrayList<>(3);
        lists.add(randomList());
        lists.add(randomList());
        lists.add(randomList());
        long count = lists
                .stream()
                .flatMap(f -> {
                    // [0, 1, 2, 3, 4] 多次被打印
                    System.out.println(f);
                    return f.stream();
                })
                .filter(f -> f > 0)
                .count();
        System.out.println(count);
    }

    private List<Integer> randomList() {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            list.add(i);
        }
        return list;
    }
```

## Optional

```java
> 来源：io.mycat.mycat2.MycatSession#getMySQLSession

下面这段代码在该类被多次使用，来看看是什么

        return backendList.stream().filter(f -> {
            return metaBean == f.getMySQLMetaBean() && f.isIDLE();
        }).findFirst().orElse(null);
```

先来看下下面这个例子
```java
    @Test
    public void fun2() {
        List<Integer> arrs = Arrays.asList(1, 1, 2, 3, 4, 5, 5);
        List<Integer> empty = new ArrayList<>();
        Integer integer =
//                empty   // 99
                arrs    // 1
                        .stream()
                        .findFirst().orElse(99);
        System.out.println(integer);
    }
```

从结果来看，当list为空的时候，返回了99；也就是说 上面代码的意图用于判断这个流是否有值；
类似于 comms工具包中的 isEmpty方法； 不过使用流来做还能增加自己的各种逻辑，比如这里的 filter；

那么Optional 是什么? 简单来说 防止npe的发生几率；例如orElse() （如果值不存在则返回默认值），还提供了多个判断值是否为空的相关操作；

## IntStream.rangeClosed 和ascii

```java
> 来源：io.mycat.mycat2.sqlparser.byteArrayInterface.Tokenizer2#Tokenizer2

 IntStream.rangeClosed('0', '9').forEach(c -> charType[c<<1] = DIGITS);
 IntStream.rangeClosed('A', 'Z').forEach(c -> charType[c<<1] = CHARS);
 IntStream.rangeClosed('a', 'z').forEach(c -> charType[c<<1] = CHARS);
```

具体是不知道在干什么，但是写了个小demo测试了下; 一个是两个api的区别，一个是使用char类型循环可以直接获取到ascii码值

```java
  // 包括0-9
  IntStream.rangeClosed('0', '9').forEach(c -> System.out.println(c));
  // 不包括9
  IntStream.range('0', '9').forEach(c -> System.out.println(c));

  for (int i = '0'; i < '9'; i++) {
      System.out.println(i);
  }
```

## 难点

目前mycat2中的流使用方式，大部分都是 过滤，转换，统计，循环；

还有一个应该是模仿了stream api的简化版；研究了下 没有看明白；本想再简化一下，只实现 limit 和 count 两个功能来体现是 stream api 大体上是怎么工作的。结果失败了；有会的朋友希望可以分享下

```java
io.mycat.mycat2.hbt.pipeline  该包下面的一个架子
```
