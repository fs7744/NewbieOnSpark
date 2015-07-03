# RDD Demo

我们在这里用一个简单的Demo简述下如何实现一个自定义的RDD。


## RDD 的自定义涉及主要代码结构 
```Scala
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging {
  
  /**
   * :: DeveloperApi ::
   * Implemented by subclasses to compute a given partition.
   * rdd的子类如何从一个partition取值
   */
  @DeveloperApi
  def compute(split: Partition, context: TaskContext): Iterator[T]

  /**
   * Implemented by subclasses to return the set of partitions in this RDD. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   * 如何生成RDD的partition，该方法之后调用一次
   */
  protected def getPartitions: Array[Partition]
  
  /**
   * Implemented by subclasses to return how this RDD depends on parent RDDs. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   * 如何得到RDD的依赖
   */
  protected def getDependencies: Seq[Dependency[_]] = deps

  /**
   * Optionally overridden by subclasses to specify placement preferences.
   * 子类在有特殊放置选择时可选择性实现
   */
  protected def getPreferredLocations(split: Partition): Seq[String] = Nil
  
}
```

除了上述可重载的方法外，RDD代码中还有很多map等各类方法，RDD子类都是可以使用的，所以这里我们将不做介绍。

一般来说，自定义一个RDD最核心涉及的两个需要重载是 compute 和 getPartitions。前者是从RDD中取值的方法，后者是RDD如何存放数据的方法。

接下来我们实现一个持有string集合的RDD，并且为了简化实现，demo不会和spark中textfil利用HadoopRDD进行数据存储，demo将直接简单的用Partition的属性持有string值：

我们自定义的RDD：
```Scala

  /**
   * RDD数据实际是放在Partition中，我们这里就简单建一个Partition，用属性Content持有数据
  */
class ContentPartition(idx: Int, val content: String) extends Partition {
  override def index: Int = idx
}

  /**
   * 我们的RDD继承于RDD[String]。Nil表示空集合，参数含义是空依赖，因为我们的目的是实现数据源的RDD功能，之前是没有依赖的
  */
class ContentRDD(sc : SparkContext,strs: Array[String]) extends RDD[String](sc,Nil){

  /**
   * 由于是数据源RDD，partition只能我们实现，这里我们就简单的按string集合的数量实现对应数量的partition，partition重要的id标志我们就简单用index表示
   */
  override  def getPartitions : Array[Partition] = {
    val array = new Array[Partition](strs.size)
    for (i <- 0 until strs.size) {
      array(i) = new ContentPartition(i, strs(i))
    }
    array
  }

  /**
   * 拿取string，我们就简单的拿Array[String]生成一个迭代器
   */
  override  def compute(split: Partition, context: TaskContext): Iterator[String] = {
    val splits = split.asInstanceOf[ContentPartition]
    Array[String](splits.content).toIterator
  }
}
```

好的，我们已经建好了自己的RDD，接下来我们就利用这个RDD做一个字符数量的统计：

```Scala
object CharDemo {
  def main(argment: Array[String]): Unit = {
    countChar(Array[String]("tttf","aad")) //传入我们的数据集合
  }

  def countChar( strs :Array[String]) :Unit = {
        val conf = new SparkConf().setAppName("test").setMaster("local") //我们直接在本地调试看这个结果
        val sc = new SparkContext(conf) //建立spark操作上下文
        new ContentRDD(sc,strs)                     //建立我们的RDD
            .flatMap { s => s.map { c => (c, 1) } } // 平展开成每个（char，1）的格式方便统计
            .reduceByKey((c1,c2) => c1 + c2)        // 统计所有的char
            .collect()                              // 收集所有结果
            .foreach(println)                       // 展示结果
        sc.stop()                                   //停止spark应用
  }
}
```

由这个例子可以看出RDD的自定义和操作都是比较方便的，这也是spark现在比hadoop map reduce 火的一个原因。