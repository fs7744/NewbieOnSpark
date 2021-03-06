# 并行集合

并行集合的官方解释是： 并行集合 (Parallelized collections) 的创建是通过在一个已有的集合(Scala Seq)上调用 SparkContext 的 parallelize 方法实现的。集合中的元素被复制到一个可并行操作的分布式数据集中。

上述内容其实只是表明：scala 本身的集合类型和RDD不能直接交互，如果数据在scala的集合中，就得把集合转为RDD来使用。

为什么必须这样呢？

很简单：RDD是分布式的，scala 内置的集合不是分布式。要混合使用，要么把集合做出分布式集合，要把RDD全拿到一个节点上和集合做处理。显而易见，我们只能选择前者。

创建并行集合的方式：

```Scala
val pc = sc.parallelize(Array(1,2,3))
```

既然是并行集合，那么这个集合被分成几份将是影响并行集合性能的关键因素，spark中将其称为切片数。spark会在每个切片区使用一个任务做处理(并行处理的普遍做法)。通常spark会自动根据集群情况设置分片数，除非你手动设置parallelize方法的第二个参数,比如：
```Scala
val pc = sc.parallelize(Array(1,2,3,5,4,3), 2)
```
