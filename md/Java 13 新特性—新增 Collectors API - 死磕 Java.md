> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/2901979956)

> Java12 引入 Collectors.teeing()，进一步增强了 Java 的流（Stream）处理能力。teeing() 允许对同时对一个流进行两种不同的收集操作，并将这两种操作的结果合并成一个。方法

原

 2023-12-14  阅读 (51)  点赞 (0)

Java 12 引入 `Collectors.teeing()`，进一步增强了 Java 的流（Stream）处理能力。

`teeing()` 允许对同时对一个流进行两种不同的收集操作，并将这两种操作的结果合并成一个。方法定义如下：

```
Collectors.teeing(Collector<? super T,A,R1> downstream1, Collector<? super T,A,R2> downstream2, BiFunction<? super R1,? super R2,R> merger)


```

*   `downstream1` 和 `downstream2`：两个不同的 `Collector` 实例。
*   `merger`：一个 `BiFunction`，用于合并两个 `Collector` 的结果。

`teeing()` 非常适合于那些需要对同一个数据集进行多重处理的情景。

`teeing()` 非常适合于那些需要对同一个数据集进行多重处理的情景。例如，你可以同时计算一个数字列表的平均值和总和，或者找出一个员工列表中工资最高和最低的员工。

比如有一个整数列，我们需要计算出这个列的最大值和平均值。

*   **以前写法**

```
    @Test
    public void teeingTest() throws IOException {
        List<Integer> list = Arrays.asList(9,5,13,22,34,16,18,23,55);
        
        Integer max = list.stream()
                .max(Integer::compareTo).get();

        
        Double ave = list.stream()
                .collect(Collectors.averagingInt(Integer::intValue));

        System.out.println(list + "，最大值:" + max + ",;平均值：" + ave);
    }


```

平均值和最大值要分别使用两个不同的 Stream 来计算，要算两遍。如果使用 `teeing()` 呢？

*   使用 `teeing()`

```
    @Test
    public void teeingTest() throws IOException {
        List<Integer> list = Arrays.asList(9,5,13,22,34,16,18,23,55);

        Map<String,Object> resultMap = list.stream()
                .collect(Collectors.teeing(
                        Collectors.maxBy(Integer::compareTo),
                        Collectors.averagingInt(Integer::intValue),
                        (v1,v2) ->{
                            return Map.of("max",v1,"ave",v2);
                        }
                ));
        System.out.println(list + "，最大值:" + resultMap.get("max").toString() + ",;平均值：" + resultMap.get("ave").toString());
    }


```

利用 `maxBy()` 和 `averagingInt()` 分别求出最大值和平均值，然后利用那个 `teeing()`，将他们两个合并到 Map 中。

这种方式比上面求两次的方式更加简洁了，但是它也可能会带来一些性能开销，尤其是在处理大数据集时。所以在使用该方法的时候需要评估数据量，对性能进行测试，确保它满足应用的性能要求。