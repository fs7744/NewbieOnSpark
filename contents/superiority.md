# spark 介绍

## spark 优势

1. spark利用内存提升运行速度，减少了磁盘读写等消耗。
2. spark社区的目标不仅仅是建立一个比hadoop更快计算框架，而是想让spark成为一个全面且统一的大数据计算框架来整合和管理不同的数据集合数据源。其中特别是利用RDD的概念实现了批处理数据和实时流数据的统一抽象。
3. 四大库：spark streaming、spark sql、spark MLlib、spark graphx 极大程度方便开发者在大数据处理方面的不同需求。
4. 数据恢复机制减轻了开发者在Hadoop MapReduce中运行失败的痛苦。

简单来说，spark是一个更快更友好的计算框架，这就是业界相比Hadoop MapReduce更偏好它的原因。