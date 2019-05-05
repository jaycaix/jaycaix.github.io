---
layout:     post
title:      "MapReduce2Spark Migration 总结"
date:       2018-03-14 15:00:00 +0800
author:     "Jay Cai"
header-style: text
catalog: true
tags:
    - SparkSQL
    - Spark
---

## MapReduce2Spark 总结
通过MapReduce to Spark的Migration项目，主要总结了以下一些性能方面的tips.

## 性能
*   多个Job归一化
    <br>Spark中的Application参考图：
    ![Spark中的Application](/img/posts/projects/spark-application.png)
    *   在Map->Reduce间反复操作的多个Job合并成一个Spark Job可能缩短执行时间
    *   Spark不需要做MapReduce的中间文件出力，减少了I/O，所以可能更快
    *   各个处理阶段转化成了Spark的Pipeline，可以使resource的使用率更高效
*   并行执行
    *   Job是从Driver开始顺序执行的，是同步的。但是Driver端可以利用多线程来实现并行执行
        例如：入力文件有多种时，可以充分利用resource来做并行加载
        <br>Job并行执行例:
        ![Job并行执行例](/img/posts/projects/job-parallel.png)
    *   各Stage也可以并行执行(注意：形成依赖关系的Stage是无法做到的)
        <br>Stage并行执行例:
        ![Stage并行执行例](/img/posts/projects/stage-parallel.png)
    *   Task是在Executor上并行执行的
*   Map-Reduce的线程数需要指定，而Spark利用Yarn Container的resource动态分配来提高利用率
*   Spark可以更高效的使用JVM
    <br>Yarn中的Spark环境参考图：
    ![Yarn中的Spark环境参考图](/img/posts/projects/yarn-spark.png)
    <br>Yarn Container中的内存管理参考图：
    ![Yarn Container中的内存管理参考图](/img/posts/projects/yarn-container-memory.png)
    *   Task是在Excutor内并行计算，所以减少了JVM的数量，消减了总的Overhead
    *   多个Task在同一JVM上运行，共享heap和cache
    *   Executor可以循环利用，减少启动executor带来的overhead
*   惰性计算
*   动态Heap管理
*   SparkSQL引擎使得DataFrame处理能够自动最优化
    *   Catalyst优化器根据数据的元数据自动选择执行计划(Join策略，排序算法等)
*   DataFrame
    *   内部构造优化(Spark自身优化)
        *   比起Java Object的RDD，Spark内部优化了DataFrame的数据结构(in-memory,结构化)
    *   需要重复利用的DataFrame，缓存至Memory，避免重复计算。后续有异常时也可以从内存加载数据。
    *   逐条运算和集合运算并用
        *   DataFrame可以像SQL一样进行集合运算
        *   `mapPartitions`和`groupByKeys`
    <br>RDD与DataFrame的对比

        |               |           RDD                 |        DataFrame          |
        |  数据结构      |  JVM Object形式的分散集合       |  Row Object形式的分散集合  |
        |  算子操作      |  函数操作(`map`、`filter`、etc)  |  表达式操作及UDF           |
        |  优化         |     -                          |  逻辑执行计划和物理执行计划  |
        |  内部结构      |    -                           |  高效的内部表示结构<br>→减少Shuffle量<br>→减少Heap  |
        
        TIPS:RDD与DataFrame之间可以相互转换，但代价较高(因为要进行序列化和反序列化操作)

## 关于调优
*   Executor resource分配是否合理
    *   Stage的`partition数=Executor数 x VCores` 是比较理想的
    *   1个Partition的输入数据量太多的话会拖慢速度
        <br>结合上一条调整Partition数，`partition数=Executor数 x VCores的整数倍`可以使得每一轮都能充分利用Executor的resource
    *   但是，可能会有各种异常导致一部分Executor失效而导致性能下降，所以应该预留10%～20%的resource来预防这种情况
        <br>`partition数=Executor数 x VCores的整数倍 - ((10%～20%) x resource)`
*   数据倾斜
    *   查看Stage的Event Timeline是否有数据倾斜现象
    *   对数据做必要的处理来使得数据均匀分布
    <br>数据倾斜示例：
    ![数据倾斜示例](/img/posts/projects/spark-data-skew.png)
*   确认一个Partition的数据量是否过大
    *   partition内的数据量该如何决定？
        *   默认是数据块大小(mapr-fs的默认值是256MB)
        *   Stage间没有发生Shuffle的时候，partition数是前一个Stage的partition数，数据量也保持一致
        *   Stage间发生Shuffle的时候，
            *  `partition数=spark.default.parallelism (RDD) 设定值`，数据量重新计算
            *  `partition数=spark.sql.shuffle.partitions (DataFrame)` 指定值，数据量重新计算
            *  `partition数=RDD.repartion() 指定值`，数据量重新计算
    *   每个Partition的数据量大小最大不要超过500MB(最佳实践)
*   超过memory上限而引起的retry
    ![](/img/posts/projects/spark-job-killed-by-yarn.png)
    1.  在History UI修改Stage的URL(改为:attempt=0)可以查看到Job Fail时的情况
    2.  可以看到Task使用的memory超过了Yarn Contanier的上限
    ![](/img/posts/projects/spark-job-killed-by-yarn-2.png)
    3.  根据上图得知，应该增加partition数或者提高`yarn.executor.memoryOverhead`
*   OutOfMemoryError
    *   java.lang.OutOfMemoryError: Java heap space
        *   表示heap memory不足
        *   此时，应增加partition数或者提高`spark.executor.memory`
    *   java.lang.OutOfMemoryError: GC Overhead limit exceeded
        *   表示GC时间过长导致程序执行不下去
        *   **source review查找原因。。**
            *   检查是否使用了大量的Java Object,例如：RDD.toList..
            *   尽量使用惰性计算
    *   Shuffle时是否出现Spill
        如果出现Spill表示partition内数据量过大，应该增加partition数了

## 案例

### DAG Visualization
![案例](/img/posts/projects/spark-example-1.png)