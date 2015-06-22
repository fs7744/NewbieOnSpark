# 共享变量

一般来说，spark操作函数(如map或者reduce)中的变量都不支持分布式，每个节点上都是独立的副本。只有以下两种变量才是spark支持的共享变量:

## 广播变量

广播变量允许在每台节点机器上缓存只读的变量，而不是每个task持有一份拷贝副本。Spark也尝试着利用有效的广播算法去分配广播变量，以减少通信的成本。

使用方式如下：

```Scala
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)
```

广播变量在创建之后，可以在集群的任意函数中使用，只是在广播之后，对应的变量是不能被修改的，因为修改的值不会被广播出去。

## 累加器

累加器是解决RDD并行操作实现count计数、sum求和等情况涉及“加”操作的变量。Spark已原生支持数字类型的累加器，自定义类型必须自己再实现。

使用方式：

```Scala
scala> val accum = sc.accumulator(0, "My Accumulator")
accum: spark.Accumulator[Int] = 0

scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum += x)
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

scala> accum.value
res2: Int = 10
```

自定义累加器必须继承于[AccumulatorParam](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.Accumulable)。该类有两个接口方法：
1. zero ，提供类似数学中0这个值一般作用的值得方法
2. addInPlace 如何计算两个值得和

例子：
```Scala
object VectorAccumulatorParam extends AccumulatorParam[Vector] {
  def zero(initialValue: Vector): Vector = {
    Vector.zeros(initialValue.size)
  }
  def addInPlace(v1: Vector, v2: Vector): Vector = {
    v1 += v2
  }
}

// Then, create an Accumulator of this type:
val vecAccum = sc.accumulator(new Vector(...))(VectorAccumulatorParam)
```

如果使用Scala写spark，还可以用[Accumulable](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.Accumulable)接口实现。也可以SparkContext.accumulableCollection累加scala中的基本集合类型。