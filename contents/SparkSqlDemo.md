# Spark Sql Demo 

我们的demo是简单的用sql的方式从csv文件中筛选数据。

```Scala
package Demo

import org.apache.spark._
import org.apache.spark.sql._

import scala.collection.mutable

object SqlDemo {
  def main(argment: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("test").setMaster("local") //本地调试运行
    val sc = new SparkContext(conf)  // 建立spark操作上下文
    val sqlContext = new SQLContext(sc) // 建立spark sql 操作上下文

    val rowRDD = sc.textFile("xxxPath\\people.csv")   //读取 csv 文件
      .map(line => {                                  // 转换成 row 结构
        val data = line.split(";")                    // 切分字段
        val list = mutable.ArrayBuffer[Any]()
        list.append(data(0))                          //填充数据
        list.append(data(1).toInt)                   // 填充数据
        Row.fromSeq(list)                           //创建row
    })

    val fields = new mutable.ArrayBuffer[StructField]()
    fields.append(StructField("name",StringType,true))  //添加 name schema
    fields.append(StructField("age",IntegerType,true))  //添加 age schema
    val schema = StructType(fields)                     //创建schema 结构

    val rdd = sqlContext.applySchema(rowRDD, schema)  //将rdd与schema做匹配

    rdd.registerTempTable("people")                 //注册临时表

    sqlContext.sql("select * from people where age >= 18")     // 筛选成年人士
      .collect()
      .foreach(println)

    sc.stop()
  }
}

```