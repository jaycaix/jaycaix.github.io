---
layout: post
title: "Apache Spark 内存管理详解"
subtitle: 'Apache Spark Memory Management'
date:       2017-04-18 14:00:00 +0800
author: "Jay Cai"
header-style: text
tags:
  - Spark
---

## Apache Spark 内存管理详解

### 引言

Spark作为一个基于内存的分布式计算引擎，其内存管理模块在整个系统中占据着非常重要的角色。理解Spark内存管理的基本原理，有助于更好地开发Spark应用程序和进行性能调优。本文旨在梳理出Spark内存管理的脉络，抛砖引玉，引出读者对这个话题的深入探讨。本文中阐述的原理基于Spark 2.1版本，阅读本文需要读者有一定的Spark和Java基础，了解RDD、Shuffle、JVM等相关概念。

在执行Spark的应用程序时，Spark集群会启动Driver和Executor两种JVM进程，前者为主控进程，负责创建Spark上下文，提交Spark作业（Job），并将作业转化为计算任务（Task），在各个Executor进程间协调任务的调度，后者负责在工作节点上执行具体的计算任务，并将结果返回给Driver，同时为需要持久化的RDD提供存储功能<sup>[1]</sup>。由于Driver的内存管理相对来说较为简单，本文主要对Executor的内存管理进行分析，下文中的Spark内存均特指Executor的内存。

 ![图1 Spark的Driver和Worker](/img/posts/memory/35301-c8a1ca935c56f158.png)

 图1 Spark的Driver和Worker



### 1 堆内和堆外内存

作为一个JVM进程，Executor的内存管理建立在JVM的内存管理之上，Spark对JVM的堆内（On-heap）空间进行了更为详细的分配，以充分利用内存。同时，Spark引入了堆外（Off-heap）内存，使之可以直接在工作节点的系统内存中开辟空间，进一步优化了内存的使用。

 ![图2 堆外和堆内内存](/img/posts/memory/35301-c896dd56ffe88b27.png)

 图2 堆外和堆内内存



#### 1.1 堆内

堆内内存的大小，由Spark应用程序启动时的`–executor-memory`或`spark.executor.memory`参数配置。Executor内运行的并发任务共享JVM堆内内存，这些任务在缓存RDD和广播（Broadcast）数据时占用的内存被规划为存储（Storage）内存，而这些任务在执行Shuffle时占用的内存被规划为执行（Execution）内存，剩余的部分不做特殊规划，那些Spark内部的对象实例，或者用户定义的Spark应用程序中的对象实例，均占用剩余的空间。不同的管理模式下，这三部分占用的空间大小各不相同（下面第2小节介绍）。

Spark对堆内内存的管理是一种逻辑上的“规划式”的管理，因为对象实例占用内存的申请和释放都由JVM完成，Spark只能在申请后和释放前**记录**这些内存：

*   **申请内存**：
    1. Spark在代码中new一个对象实例
    2. JVM从堆内内存分配空间，创建对象并返回对象引用
    3. Spark保存该对象的引用，记录该对象占用的内存
*   **释放内存**：
    1. Spark记录该对象释放的内存，删除该对象的引用
    2. 等待JVM的垃圾回收机制释放该对象占用的堆内内存

我们知道，JVM的对象可以以序列化的方式存储，序列化的过程是将对象转换为二进制字节流，本质上可以理解为将非连续空间的链式存储转化为连续空间或块存储，在访问时则需要进行序列化的逆过程——反序列化，将字节流转化为对象，序列化的方式可以节省存储空间，但增加了存储和读取时候的计算开销。

对于Spark中序列化的对象，由于是字节流的形式，其占用的内存大小可直接计算，而对于非序列化的对象，其占用的内存是通过周期性地采样近似估算而得，即并不是每次新增的数据项都会计算一次占用的内存大小，这种方法降低了时间开销但是有可能误差较大，导致某一时刻的实际内存有可能远远超出预期<sup>[2]</sup>。此外，在被Spark标记为释放的对象实例，很有可能在实际上并没有被JVM回收，导致实际可用的内存小于Spark记录的可用内存。所以Spark并不能准确记录实际可用的堆内内存，从而也就无法完全避免内存溢出（OOM, Out of Memory）的异常。

虽然不能精准控制堆内内存的申请和释放，但Spark通过对存储内存和执行内存各自独立的规划管理，可以决定是否要在存储内存里缓存新的RDD，以及是否为新的任务分配执行内存，在一定程度上可以提升内存的利用率，减少异常的出现。

#### 1.2 堆外

为了进一步优化内存的使用以及提高Shuffle时排序的效率，Spark引入了堆外（Off-heap）内存，使之可以直接在工作节点的系统内存中开辟空间，存储经过序列化的二进制数据。利用JDK Unsafe API（从Spark 2.0开始，在管理堆外的存储内存时不再基于Tachyon，而是与堆外的执行内存一样，基于JDK Unsafe API实现<sup>[3]</sup>），Spark可以直接操作系统堆外内存，减少了不必要的内存开销，以及频繁的GC扫描和回收，提升了处理性能。堆外内存可以被精确地申请和释放，而且序列化的数据占用的空间可以被精确计算，所以相比堆内内存来说降低了管理的难度，也降低了误差。

在默认情况下堆外内存并不启用，可通过配置`spark.memory.offHeap.enabled`参数启用，并由`spark.memory.offHeap.size`参数设定堆外空间的大小。除了没有other空间，堆外内存与堆内内存的划分方式相同，所有运行中的并发任务共享存储内存和执行内存。

#### 1.3 接口

Spark为存储内存和执行内存的管理提供了统一的接口——`MemoryManager`，同一个Executor内的任务都调用这个接口的方法来申请或释放内存，同时在调用这些方法时都需要指定内存模式（MemoryMode），这个参数决定了是在堆内还是堆外完成这次操作。MemoryManager的具体实现上，Spark 1.6之后默认为统一管理（[Unified Memory Manager](https://github.com/apache/spark/blob/v2.1.0/core/src/main/scala/org/apache/spark/memory/UnifiedMemoryManager.scala)）方式，1.6之前采用的静态管理（[Static Memory Manager](https://github.com/apache/spark/blob/v2.1.0/core/src/main/scala/org/apache/spark/memory/StaticMemoryManager.scala)）方式仍被保留，可通过配置`spark.memory.useLegacyMode`参数启用。两种方式的区别在于对空间分配的方式，下面分别对这两种方式进行介绍。

### 2 内存空间分配

#### 2.1 静态内存管理

###### 堆内

在静态内存管理机制下，存储内存、执行内存和其他内存三部分的大小在Spark应用程序运行期间是固定的，但用户可以在应用程序启动前进行配置，堆内内存的分配如图3所示：

 ![图3 静态内存管理图示——堆内](/img/posts/memory/35301-1c5ca5f6aa6cbd28.png)
 图3 静态内存管理图示——堆内

可以看到，可用的堆内内存的大小需要按照下面的方式计算：
>可用的存储内存 = systemMaxMemory * spark.storage.memoryFraction * spark.storage.safetyFraction  
>可用的执行内存 = systemMaxMemory * spark.shuffle.memoryFraction * spark.shuffle.safetyFraction

其中`systemMaxMemory`取决于当前JVM堆内内存的大小，最后可用的执行内存或者存储内存要在此基础上与各自的`memoryFraction`参数和`safetyFraction`参数相乘得出。上述计算公式中的两个`safetyFraction`参数，其意义在于在逻辑上预留出`1-safetyFraction`这么一块保险区域，降低因实际内存超出当前预设范围而导致OOM的风险（上文提到，对于非序列化对象的内存采样估算会产生误差）。值得注意的是，这个预留的保险区域仅仅是一种逻辑上的规划，在具体使用时Spark并没有区别对待，和“其它内存”一样交给了JVM去管理。

###### 堆外

堆外的空间分配较为简单，存储内存、执行内存的大小同样是固定的，如图4所示：

 ![图4 静态内存管理图示——堆外](/img/posts/memory/35301-f9f9801130508114.png)

 图4 静态内存管理图示——堆外


可用的执行内存和存储内存占用的空间大小直接由参数`spark.memory.storageFraction`决定，由于堆外内存占用的空间可以被精确计算，所以无需再设定保险区域。

静态内存管理机制实现起来较为简单，但如果用户不熟悉Spark的存储机制，或没有根据具体的数据规模和计算任务或做相应的配置，很容易造成“一半海水，一半火焰”的局面，即存储内存和执行内存中的一方剩余大量的空间，而另一方却早早被占满，不得不淘汰或移出旧的内容以存储新的内容。由于新的内存管理机制的出现，这种方式目前已经很少有开发者使用，出于兼容旧版本的应用程序的目的，Spark仍然保留了它的实现。

#### 2.2 统一内存管理

Spark 1.6之后引入的统一内存管理机制，与静态内存管理的区别在于存储内存和执行内存共享同一块空间，可以动态占用对方的空闲区域，如图5和图6所示

 ![图5 统一内存管理图示——堆内](/img/posts/memory/35301-671d45082ea6ea92.png)
 图5 统一内存管理图示——堆内


 ![图6 统一内存管理图示——堆外](/img/posts/memory/35301-76396dd2a12cdeba.png)
 图6 统一内存管理图示——堆外

动态占用机制的规则如下：

*   设定基本的存储内存和执行内存区域（`spark.storage.storageFraction`参数），该设定确定了双方各自拥有的空间的范围
*   双方的空间都不足时，则存储到硬盘；若己方空间不足而对方空余时，可借用对方的空间;（存储空间不足是指不足以放下一个完整的Block）
*   执行内存的空间被对方占用后，可让对方将占用的部分转存到硬盘，然后“归还”借用的空间
*   存储内存的空间被对方占用后，无法让对方“归还”，因为需要考虑Shuffle过程中的很多因素，实现起来较为复杂<sup>[4]</sup>

 ![ 图7 动态占用机制图解](/img/posts/memory/35301-ef5c111168db7c31.png)
 图7 动态占用机制图解


凭借统一内存管理机制，Spark在一定程度上提高了堆内和堆外内存资源的利用率，降低了开发者维护Spark内存的难度，但并不意味着开发者可以高枕无忧。譬如，所以如果存储内存的空间太大或者说缓存的数据过多，反而会导致频繁的全量垃圾回收，降低任务执行时的性能，因为缓存的RDD数据通常都是长期驻留内存的<sup>[5]</sup>。所以要想充分发挥Spark的性能，需要开发者进一步了解存储内存和执行内存各自的管理方式和实现原理。

### 3\. 存储内存管理

#### 3.1 RDD的持久化机制

弹性分布式数据集（RDD）作为Spark最根本的数据抽象，是只读的分区记录（Partition）的集合，只能基于在稳定物理存储中的数据集上创建，或者在其他已有的RDD上执行转换（Transformation）操作产生一个新的RDD。转换后的RDD与原始的RDD之间产生的依赖关系，构成了血统（Lineage）。凭借血统，Spark保证了每一个RDD都可以被重新恢复。但RDD的所有转换都是惰性的，即只有当一个返回结果给Driver的行动（Action）发生时，Spark才会创建任务读取RDD，然后真正触发转换的执行。

Task在启动之初读取一个分区时，会先判断这个分区是否已经被持久化，如果没有则需要检查Checkpoint或按照血统重新计算。所以如果一个RDD上要执行多次行动，可以在第一次行动中使用persist或cache方法，在内存或磁盘中持久化或缓存这个RDD，从而在后面的行动时提升计算速度。事实上，cache方法是使用默认的MEMORY_ONLY的存储级别将RDD持久化到内存，故缓存是一种特殊的持久化。**堆内和堆外存储内存的设计，便可以对缓存RDD时使用的内存做统一的规划和管理**（存储内存的其他应用场景，如缓存broadcast数据，暂时不在本文的讨论范围之内）。

RDD的持久化由Spark的Storage模块<sup>[7]</sup>负责，实现了RDD与物理存储的解耦合。Storage模块负责管理Spark在计算过程中产生的数据，将那些在内存或磁盘、在本地或远程存取数据的功能封装了起来。在具体实现时Driver端和Executor端的Storage模块构成了主从式的架构，即Driver端的BlockManager为Master，Executor端的BlockManager为Slave。Storage模块在逻辑上以Block为基本存储单位，RDD的每个Partition经过处理后唯一对应一个Block（BlockId的格式为`rdd_RDD-ID_PARTITION-ID`）。Master负责整个Spark应用程序的Block的元数据信息的管理和维护，而Slave需要将Block的更新等状态上报到Master，同时接收Master的命令，例如新增或删除一个RDD。

 ![图8 Storage模块示意图](/img/posts/memory/35301-d83f9c38f26f5752.png)
 图8 Storage模块示意图

在对RDD持久化时，Spark规定了MEMORY_ONLY、MEMORY_AND_DISK等7种不同的[存储级别](http://spark.apache.org/docs/latest/programming-guide.html#rdd-persistence)，而存储级别是以下5个变量的组合<sup>[8]</sup>：
```scala
class StorageLevel private(
    private var _useDisk: Boolean, //磁盘
    private var _useMemory: Boolean, //这里其实是指堆内内存
    private var _useOffHeap: Boolean, //堆外内存
    private var _deserialized: Boolean, //是否为非序列化
    private var _replication: Int = 1 //副本个数
)
```
通过对数据结构的分析，可以看出存储级别从三个维度定义了RDD的Partition（同时也就是Block）的存储方式：

*   存储位置：磁盘／堆内内存／堆外内存。如MEMORY_AND_DISK是同时在磁盘和堆内内存上存储，实现了冗余备份。OFF_HEAP则是只在堆外内存存储，目前选择堆外内存时不能同时存储到其他位置。
*   存储形式：Block缓存到存储内存后，是否为非序列化的形式。如MEMORY_ONLY是非序列化方式存储，OFF_HEAP是序列化方式存储。
*   副本数量：大于1时需要远程冗余备份到其他节点。如DISK_ONLY_2需要远程备份1个副本。

#### 3.2 RDD缓存的过程

RDD在缓存到存储内存之前，Partition中的数据一般以迭代器（[Iterator](http://www.scala-lang.org/docu/files/collections-api/collections_43.html)）的数据结构来访问，这是Scala语言中一种遍历数据集合的方法。通过Iterator可以获取分区中每一条序列化或者非序列化的数据项(Record)，这些Record的对象实例在逻辑上占用了JVM堆内内存的other部分的空间，同一Partition的不同Record的空间并不连续。

RDD在缓存到存储内存之后，Partition被转换成Block，Record在堆内或堆外存储内存中占用一块连续的空间。**将Partition由不连续的存储空间转换为连续存储空间的过程，Spark称之为“展开”（Unroll）**。Block有序列化和非序列化两种存储格式，具体以哪种方式取决于该RDD的存储级别。非序列化的Block以一种DeserializedMemoryEntry的数据结构定义，用一个数组存储所有的Java对象，序列化的Block则以SerializedMemoryEntry的数据结构定义，用字节缓冲区（ByteBuffer）来存储二进制数据。每个Executor的Storage模块用一个链式Map结构（LinkedHashMap）来管理堆内和堆外存储内存中所有的Block对象的实例<sup>[12]</sup>，对这个LinkedHashMap新增和删除间接记录了内存的申请和释放。

因为不能保证存储空间可以一次容纳Iterator中的所有数据，当前的计算任务在Unroll时要向MemoryManager申请足够的Unroll空间来临时占位，空间不足则Unroll失败，空间足够时可以继续进行。对于序列化的Partition，其所需的Unroll空间可以直接累加计算，一次申请。而非序列化的Partition则要在遍历Record的过程中依次申请，即每读取一条Record，采样估算其所需的Unroll空间并进行申请，空间不足时可以中断，释放已占用的Unroll空间。如果最终Unroll成功，当前Partition所占用的Unroll空间被转换为正常的缓存RDD的存储空间，如下图2所示。

 ![图9 Spark Unroll示意图](/img/posts/memory/35301-63923cd1946dd929.png)
 图9 Spark Unroll示意图

图3和图5中可以看到，在静态内存管理时，Spark在存储内存中专门划分了一块Unroll空间，其大小是固定的，统一内存管理时则没有对Unroll空间进行特别区分，当存储空间不足是会根据动态占用机制进行处理。

#### 3.3 淘汰和落盘

由于同一个Executor的所有的计算任务共享有限的存储内存空间，当有新的Block需要缓存但是剩余空间不足且无法动态占用时，就要对LinkedHashMap中的旧Block进行淘汰（Eviction)，而被淘汰的Block如果其存储级别中同时包含存储到磁盘的要求，则要对其进行落盘（Drop），否则直接删除该Block。
存储内存的淘汰规则为：

*   被淘汰的旧Block要与新Block的MemoryMode相同，即同属于堆外或堆内内存
*   新旧Block不能属于同一个RDD，避免循环淘汰
*   旧Block所属RDD不能处于被读状态，避免引发一致性问题
*   遍历LinkedHashMap中Block，按照最近最少使用（LRU）的顺序淘汰，直到满足新Block所需的空间。其中LRU是LinkedHashMap的特性。

落盘的流程则比较简单，如果其存储级别符合`_useDisk`为true的条件，再根据其`_deserialized`判断是否是非序列化的形式，若是则对其进行序列化，最后将数据存储到磁盘，在Storage模块中更新其信息。

### 4\. 执行内存管理

#### 4.1 多任务间的分配

Executor内运行的任务同样共享执行内存，Spark用一个HashMap结构保存了任务到内存耗费的映射。每个任务可占用的执行内存大小的范围为`1/2N ~ 1/N`，其中N为当前Executor内正在运行的任务的个数。每个任务在启动之时，要向MemoryManager请求申请最少为1/2N的执行内存，如果不能被满足要求则该任务被阻塞，直到有其他任务释放了足够的执行内存，该任务才可以被唤醒。

#### 4.2 Shuffle的内存占用

执行内存主要用来存储任务在执行Shuffle时占用的内存，Shuffle是按照一定规则对RDD数据重新分区的过程，我们来看Shuffle的Write和Read两阶段对执行内存的使用：

*   Shuffle Write
    *   若在map端选择普通的排序方式，会采用ExternalSorter进行外排，在内存中存储数据时主要占用堆内执行空间。
    *   若在map端选择Tungsten的排序方式，则采用ShuffleExternalSorter直接对以序列化形式存储的数据排序，在内存中存储数据时可以占用堆外或堆内执行空间，取决于用户是否开启了堆外内存以及堆外执行内存是否足够。
*   Shuffle Read
    *   在对reduce端的数据进行聚合时，要将数据交给Aggregator处理，在内存中存储数据时占用堆内执行空间。
    *   如果需要进行最终结果排序，则要将再次将数据交给ExternalSorter处理，占用堆内执行空间。

在ExternalSorter和Aggregator中，Spark会使用一种叫AppendOnlyMap的哈希表在堆内执行内存中存储数据，但在Shuffle过程中所有数据并不能都保存到该哈希表中，当这个哈希表占用的内存会进行周期性地采样估算，当其大到一定程度，无法再从MemoryManager申请到新的执行内存时，Spark就会将其全部内容存储到磁盘文件中，这个过程被称为溢存(Spill)，溢存到磁盘的文件最后会被归并(Merge)。

Shuffle Write阶段中用到的Tungsten是Databricks公司提出的对Spark优化内存和CPU使用的计划<sup>[10]</sup>，解决了一些JVM在性能上的限制和弊端。Spark会根据Shuffle的情况来自动选择是否采用Tungsten排序。Tungsten采用的页式内存管理机制建立在MemoryManager之上，即Tungsten对执行内存的使用进行了一步的抽象，这样在Shuffle过程中无需关心数据具体存储在堆内还是堆外。每个内存页用一个MemoryBlock来定义，并用`Object obj`和`long offset`这两个变量统一标识一个内存页在系统内存中的地址。堆内的MemoryBlock是以long型数组的形式分配的内存，其`obj`的值为是这个数组的对象引用，`offset`是long型数组的在JVM中的初始偏移地址，两者配合使用可以定位这个数组在堆内的绝对地址；堆外的MemoryBlock是直接申请到的内存块，其`obj`为null，`offset`是这个内存块在系统内存中的64位绝对地址。Spark用MemoryBlock巧妙地将堆内和堆外内存页统一抽象封装，并用页表(pageTable)管理每个Task申请到的内存页。

Tungsten页式管理下的所有内存用64位的逻辑地址表示，由页号和页内偏移量组成：

```
1. 页号：占13位，唯一标识一个内存页，Spark在申请内存页之前要先申请空闲页号。
2. 页内偏移量：占51位，是在使用内存页存储数据时，数据在页内的偏移地址。
```
有了统一的寻址方式，Spark可以用64位逻辑地址的指针定位到堆内或堆外的内存，整个Shuffle Write排序的过程只需要对指针进行排序，并且无需反序列化，整个过程非常高效，对于内存访问效率和CPU使用效率带来了明显的提升<sup>[11]</sup>。

### 小结

Spark的存储内存和执行内存有着截然不同的管理方式：对于存储内存来说，Spark用一个LinkedHashMap来集中管理所有的Block，Block由需要缓存的RDD的Partition转化而成；而对于执行内存，Spark用AppendOnlyMap来存储Shuffle过程中的数据，在Tungsten排序中甚至抽象成为页式内存管理，开辟了全新的JVM内存管理机制。

### 结束语

Spark的内存管理是一套复杂的机制，且Spark的版本更新比较快，笔者水平有限，难免有叙述不清、错误的地方，若读者有好的建议和更深的理解，还望不吝赐教。

### 参考文献

1.  [Spark Cluster Mode Overview](https://link.jianshu.com?t=http://spark.apache.org/docs/latest/cluster-overview.html)
2.  [Spark Sort Based Shuffle内存分析](https://www.jianshu.com/p/c83bb237caa8)
3.  [Spark OFF_HEAP](https://www.jianshu.com/p/c6f6d4071560)
4.  [Unified Memory Management in Spark 1.6](https://link.jianshu.com?t=https://issues.apache.org/jira/secure/attachment/12765646/unified-memory-management-spark-10000.pdf)
5.  [Tuning Spark: Garbage Collection Tuning](https://link.jianshu.com?t=http://spark.apache.org/docs/latest/tuning.html#garbage-collection-tuning)
6.  [Spark Architecture](https://link.jianshu.com?t=https://0x0fff.com/spark-architecture/)
7.  [《Spark技术内幕：深入解析Spark内核架构于实现原理》——第8章 Storage模块详解](https://link.jianshu.com?t=https://book.douban.com/subject/26649141/)
8.  [Spark存储级别的源码](https://link.jianshu.com?t=https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/storage/StorageLevel.scala)
9.  [Project Tungsten: Bringing Apache Spark Closer to Bare Metal](https://link.jianshu.com?t=https://databricks.com/blog/2015/04/28/project-tungsten-bringing-spark-closer-to-bare-metal.html)
10.  [Spark Tungsten-sort Based Shuffle 分析](https://www.jianshu.com/p/d328c96aebfd)
11.  [探索Spark Tungsten的秘密](https://link.jianshu.com?t=https://github.com/hustnn/TungstenSecret/tree/master)
12.  [Spark Task 内存管理（on-heap&off-heap）](https://www.jianshu.com/p/8f9ed2d58a26);


### 原文链接

[Apache Spark 内存管理详解](https://www.ibm.com/developerworks/cn/analytics/library/ba-cn-apache-spark-memory-management/)