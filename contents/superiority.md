# spark 介绍

## spark 优势

1. spark利用内存提升运行速度，减少了磁盘读写等消耗。
2. spark社区的目标不仅仅是建立一个比hadoop更快计算框架，而是想让spark成为一个全面且统一的大数据计算框架来整合和管理不同的数据集合数据源。其中特别是利用RDD的概念实现了批处理数据和实时流数据的统一抽象。
3. 四大库：spark streaming、spark sql、spark MLlib、spark graphx 极大程度方便开发者在大数据处理方面的不同需求。
4. 数据恢复机制减轻了开发者曾经在Hadoop MapReduce中遇见运行失败的痛苦。
5. 支持Hadoop yarn、mesos等等方式。
6. 活跃的社区，spark 变动和文档各方面都可以体现。

简单来说，spark是一个更快更友好的计算框架，这就是业界相比Hadoop MapReduce更偏好它的原因。

## spark 与编程语言

spark主要支持scala、java、python三个编程语言供用户开发spark应用。

听闻spark也正在试图将R融入其中，或许R的加入将极大简化spark应用的门槛。

个人认为要使用spark的话，scala最好作为最优先的选择：
1. spark本来就是scala编写的
2. scala本身就是一门优秀的语言，其的强大之处远大于java或python
3. scala的作者为scala做了一个伟大而狡猾的特性：基于jvm，基本无缝调用java的任意库，所以java有多少开源库可以，scala就有多少可用