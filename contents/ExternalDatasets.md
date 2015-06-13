# 读取外部数据

除了从scala集合生成RDD外，我们也可以从文件、其他数据库中读取数据。这里我们遵从spark官方教程，将只介绍从基本的几种文件读取，而比如json文件，数据库等等其他常与spark sql 或 dataFrame 之中添加的读取方式将留在spark sql 和 dataframe 章节再作介绍。

spark支持基本读取方式有 ：
1. text file ，方法 SparkContext.textFile("xx.txt",切片数量参数)
    
    参数可以是文件(/a/xx.txt)，目录(/a/directory)，压缩文档(/a/a.gz)，甚至支持通配符(/a/*.txt)，URL格式可以，比如hdfs://, s3n://，etc URI

    方法返回结果为文件每一行
    
    如果要读取一个文件夹下的所有小文本文件，可以使用方法 SparkContext.wholeTextFiles，和textFile不同的是，方法返回结果是(filename,content) 这样的格式
2. sequence file，方法 SparkContext.sequenceFile[K, V]
3. Hadoop input format，方法 SparkContext.hadoopRDD 或者基于新的MapReduce API的 SparkContext.newAPIHadoopRDD 方法

PS：文件必须保证worker nodes 能访问，否则节点到哪去给您找文件啊？

spark在1.3之后，即推出dataframe之后，spark sql 库中支持以下几种方式读取数据：
1. Parquet Files
2. JSON 
3. Hive Tables
4. JDBC To Other Databases
 
spark 可以使用 RDD.saveAsObjectFile 或者 SparkContext.objectFile ,比如Avro这种序列化，当然效率可能不高。

