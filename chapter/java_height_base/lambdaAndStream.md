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

* `Supplier<T>`：该接口不接收任何参数，返回一个对象T
* `Predicate<T>`：该接口接收一个对象T，返回一个Boolean
* `Function<T,R>`: 该接口接收一个对象T，经过转换判断，返回一个对象R
* `BiFunction<T,U,R>` 输入 T,U  返回R
* `UnaryOperator<T>` 输入T,返回T; 用于给这个t加工处理

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


