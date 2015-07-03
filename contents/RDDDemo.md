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