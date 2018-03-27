# lambda与stream api

## 函数式接口

这个简单理解看例子：java多线程启动方式
```java
new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("hello word");
            }
        }).start();

// 这种方式 等同于上面的传统方式
new Thread(() -> System.out.println("hello word")).start();
```

Thread 的构造函数接受一个 Runnable 接口；而上面的代码又是等同的；
那么你可以理解为这个构造中的代码是等同的；

可以这样理解，`() 空参 = runnable的 run() 方法` 因为runnable接口只有这一个方法，也称为功能接口； 后面的就是方法体；
重点来了； 把这个拉姆达表达式**转换成传统方式**，也就是第一种方法的写法；

这里一定要记得 **转换**；然后来理解java8中预置的几个功能函数接口

**转换**、**转换**、**转换** 重要的事情说三遍。拉姆达表达式可以转换成普通的java代码



| 接口名称 | 默认接口方法 | 作用
|----------|------------|-----
| `Supplier<T>` | `T get() ` | 该接口不接收任何参数，返回一个对象T
| `Predicate<T>` | `boolean test(T t)` | 该接口接收一个对象T，返回一个Boolean
| `Function<T,R>` | `R apply(T t) ` | 该接口接收一个对象T，经过转换判断，返回一个对象R
| `BiFunction<T,U,R>` | `R apply(T t, U u) ` | 输入 T,U  返回R
| `UnaryOperator<T>` | `R apply(T t)` | 输入T,返回T; 用于给这个t加工处理
| `Consumer<T>` | `accept(T t) ` | 输入T不返回结果

上面是几种常用的预定义函数接口，下面总结下

java.util.function中提供了很多函数式接口，归类为以下几个：

* Function 对单个值操作，并返回一个值
* Supplier 工厂，不接受值，返回一个结果
* Predicate 对单个值操作，并返回一个boolean
* Consumer  对单个值操作，不返回结果
* Bi 前缀的基本上都是 对两个数操作（接收2个数）


上面列出来的几个都是不同场景下的 功能函数接口；拿stream的map来示例解说

```java
 List<Apple> apps = Arrays.asList(
                new Apple("red", 120),
                new Apple("blue", 80),
                new Apple("green", 100));
        Function<Apple, String> f = (app) -> app.getColor();

        apps.stream().map(f).forEach(System.out::println);
```

这是 map的源码签名

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

可以看出来，需要的是一个 function接口;该接口的功能是 该接口接收一个对象T，经过转换判断，返回一个对象R； 那么在这里是把Apple 转成了 String；  所以只要有是Function的子类，就都可以丢给map函数；我相信其他的api也一定是类似的；

> [其他的函数接口测试请右键新窗口查看GIT](https://github.com/zq99299/newstudy/blob/master/hp-base/src/test/java/cn/mrcode/newstudy/hpbase/_04/functioninterface/Practice.java)

再看一个 `.collect(Collectors.groupingBy(Locale::getCountry));` 一个分组函数
 
```java
    public static <T, K> Collector<T, ?, Map<K, List<T>>>
    groupingBy(Function<? super T, ? extends K> classifier) {
        return groupingBy(classifier, toList());
    }
```

可以看到需要的是一个function；但是你继续往下看源码就会发现，里面多了很多的逻辑，function只是用来语义化的一个操作。 最主要的是嵌套在了处理它的框架代码中，所以就出现了很多变体。

## stream - 测试总结

* 创建流
    * `Stream.of(arr)`
    * 各种集合的 `.stream()`
    * 创建无限流：` Stream.generate(Math::random)`
    * 创建无限流：` Stream.iterate(0, n -> n + 1)`
* 流转换
    * filter : 过滤
    * map ：把每个元素转换成你所需要的类型
    * flatMap : 展开流，把一个流集合展开成一个流
* 提取子流
    * skip ： 跳过n个元素
    * limit ：返回包含n个元素的流
* 组合流
    * concat : 把多个流组合成一个流
* 有状态的转换
    * distinct ： 去重，去重就得记录之前读取到的元素才能去重，所以叫有状态
    * sorted ： 排序
* 聚合方法 ：将流聚合成一个值，流被终结不能再使用
* 分组和分片：
    * groupingBy ：按逻辑分组
    * partitioningBy ：只返回 true 或则 false的分组















