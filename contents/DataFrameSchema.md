# DataFrames Schema 介绍

DataFrame 可以以两种不同的方式从RDD转化而来，其实准确来说，转化只有一种方式，只是如何识别RDD，如何对应DataFrame的Schema信息有两种方式。

1. 通过现有的class获取schema信息
    
例如如下代码：

```Scala
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
import sqlContext.implicits._

case class Person(name: String, age: Int)
val people = sc.textFile("examples/src/main/resources/people.txt").map(_.split(",")).map(p => Person(p(0), p(1).trim.toInt)).toDF()
```

但是case calss 在scala 2.10中字段数量是有限的,只能22个,为了规避这个问题,你可以通过实现scala的[Product接口](http://www.scala-lang.org/api/current/index.html#scala.Product):

2. 通过构建具体的schema信息，比如设定字段名、类型等

例如如下代码：

```Scala
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
import org.apache.spark.sql.Row;
import org.apache.spark.sql.types.{StructType,StructField,StringType};

val schema =
  StructType(
    "name age".split(" ").map(fieldName => StructField(fieldName, StringType, true)))
    
val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))

val peopleDataFrame = sqlContext.createDataFrame(rowRDD, schema)
```