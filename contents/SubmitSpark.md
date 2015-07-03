# 提交spark应用

spark 有一个spark-submit命令可以提交spark应用，

比如：

```
./bin/spark-submit \
  --class <main-class>
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # other options
  <application-jar> \
  [application-arguments]
```

参数含义：

| 参数 | 含义|
| -- | -- |
| --class | 包含spark任务入口的类 |
| --master | spark 集群url |
| --deploy-mode | 部署方式，分为client和cluster两种 |
| --conf | 额外配置属性 |
| --application-jar | jar目录 |
| --application-argument | main方法参数 |


master参数可以是以下这些形式：

| 格式 | 含义 |
| -- | -- |
| local | 本地运行spark且只有一个worker线程 |
| local[K] | 本地运行spark且有K个worker线程。k一般设置为机器的CPU数量 | 
| local[*] | 本地运行spark并以尽可能多的worker线程运行 | 
| spark://HOST:PORT | 连接指定的standalone集群 | 
| mesos://HOST:PORT | 连接到指定的mesos集群，如果mesos集群使用zookeeper管理，则为mesos://zk://.... | 
| yarn-client | 以client方式连接到yarn集群 | 
| yarn-cluster | 以cluster模式连接到yarn集群 | 

例子：

```
./bin/spark-submit 
  --class org.apache.spark.examples.SparkPi 
  --master spark://xxx.xxx.xxx.xxx:7077 
  --executor-memory 20G 
  --total-executor-cores 100 
  /path/to/examples.jar 
```

官方也给出了这样直接在java代码中提交应用的例子：

```Java
import org.apache.spark.launcher.SparkLauncher;

   public class MyLauncher {
     public static void main(String[] args) throws Exception {
       Process spark = new SparkLauncher()
         .setAppResource("/my/app.jar")
         .setMainClass("my.spark.app.Main")
         .setMaster("local")
         .setConf(SparkLauncher.DRIVER_MEMORY, "2g")
         .launch();
       spark.waitFor();
     }
   }
```

spark 也有类似的类 ：https://spark.apache.org/docs/latest/api/scala/#org.apache.spark.launcher.SparkLauncher