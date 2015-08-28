# Spark Sql 坑

由于公司是1.2 的spark，所以我遇见的问题不一定在其他版本存在。

1. spark sql 的字段居然是大写区分的，囧，一旦大小写对应不上，sql异常，你就只能眼花了
2. spark sql 的错误提示信息太奇葩了，一点也不准确，比如你where 条件的字段名不对，绝对不会提示你哪个字段找不到，直接把select 的n多字段列出来，把你看晕死
3. 本地测试时一定要使用多核心模式，否则多线程问题你是看不出来的，并且单核心真心慢死人
4. udf function 必须是可序列化到分布式环境的，如果出现无法序列化的问题，多半是你不小心使用了类成员等依赖于某个类的变量或方法，在scala中可放在伴生类中
5. spark sql StructField 的 DataType 无法转为 DecimalType， 当我们根据 StructField做处理时，DecimalType暂时无处处理
6. spark sql 的 schemaRDD 实际列少于定义,比如使用union all 时如果列数不一样，在运行时会报java.lang.ArrayIndexOutOfBoundsException错误，该异常是不是很好看懂呢？