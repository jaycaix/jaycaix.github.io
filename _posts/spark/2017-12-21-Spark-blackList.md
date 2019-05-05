---
layout: post
title: "Spark黑名单机制"
subtitle: 'Spark BlackList'
author: "Jay Cai"
header-style: text
tags:
  - Spark
---

## 适用场景
在使用 Apache Spark的时候，作业会以分布式的方式在不同的节点上运行；特别是当集群的规模很大时，集群的节点出现各种问题是很常见的，比如某个磁盘出现问题等。我们都知道 Apache Spark是一个高性能、容错的分布式计算框架，一旦它知道某个计算所在的机器出现问题（比如磁盘故障），它会依据之前生成的 lineage 重新调度这个 Task。

我们现在来考虑下下面的场景：

*   有个节点上的磁盘由于某些原因出现间歇性故障，导致某些扇区不能被读取。假设我们的 Spark 作业需要的数据正好就在这些扇区上，这将会导致这个 Task 失败。
*   这个作业的 Driver 获取到这个信息，知道 Task 失败了，所以它会重新提交这个 Task。
*   Scheduler 获取这个请求之后，它会考虑到数据的本地性问题，所以很可能还是把这个 Task 分发到上述的机器，因为它并不知道上述机器的磁盘出现了问题。
*   因为这个机器的磁盘出现问题，所以这个 Task 可能一样失败。然后 Driver 重新这些操作，最终导致了 Spark 作业出现失败！

上面提到的场景其实对我们人来说可以通过某些措施来避免。但是对于 Apache Spark 2.2.0 版本之前是无法避免的，不过高兴的是，来自 Cloudera 的工程师解决了这个问题：引入了黑名单机制 Blacklist（详情可以参见[SPARK-8425](https://issues.apache.org/jira/browse/SPARK-8425)，具体的设计文档参见[Design Doc for Blacklist Mechanism](https://www.iteblog.com/redirect.php?url=aHR0cHM6Ly9pc3N1ZXMuYXBhY2hlLm9yZy9qaXJhL3NlY3VyZS9hdHRhY2htZW50LzEyODMyNjg0L0Rlc2lnbkRvY2ZvckJsYWNrbGlzdE1lY2hhbmlzbS5wZGY=&article=true)），并且随着 Apache Spark 2.2.0 版本发布，不过目前还处于实验性阶段。

黑名单机制其实是通过维护之前出现问题的执行器（Executors）和节点（Hosts）的记录。当某个任务（Task）出现失败，那么黑名单机制将会追踪这个任务关联的执行器以及主机，并记下这些信息；当在这个节点调度任务出现失败的次数超过一定的数目（默认为2），那么调度器将不会再将任务分发到那台节点。调度器甚至可以杀死那台机器对应的执行器，这些都可以通过相应的配置实现。

我们可以通过 Apache Spark WEB UI 界面看到执行器的状态（Status）：如果执行器处于黑名单状态，你可以在页面上看到其状态为 **Blacklisted** ，否则为 **Active**。如下图所示：

 [![blacklisting-in-apache-spark](/img/posts/spark-blacklist.png)](/img/posts/spark-blacklist.png)

拥有了黑名单机制之后，上面场景的问题就可以很好的解决。

目前黑名单机制可以通过一系列的参数来控制，主要如下：

| 参数 | 默认值 | 含义 |
|      |       |      |
| spark.blacklist.enabled | false | 如果这个参数这为 true，那么 Spark 将不再会往黑名单里面的执行器调度任务。黑名单算法可以由其他“spark.blacklist”配置选项进一步控制，详情参见下面的介绍。 |
| spark.blacklist.timeout | 1h | (实验性) 对于被加入 application 黑名单的 executor/节点 ，多长时间后无条件的移出黑名单以运行新任务。 |
| spark.blacklist.task.maxTaskAttemptsPerExecutor | 1 | (实验性) 对于同一个 task 在某个 executor 中的失败重试阈值。达到阈值后，在执行这个 task 时，该 executor 将被加入黑名单。 |
| spark.blacklist.task.maxTaskAttemptsPerNode | 2 | (实验性) 对于同一个 task 在某个节点上的失败重试阈值。达到阈值后，在执行这个 task 时，该节点将被加入黑名单。 |
| spark.blacklist.stage.maxFailedTasksPerExecutor | 2 | (实验性) 一个 stage 中，不同的 task 在同一个 executor 的失败阈值。达到阈值后，在执行这个 stage 时该 executor 将会被加入黑名单。 |
| spark.blacklist.stage.maxFailedExecutorsPerNode | 2 | (实验性) 一个 stage 中，不同的 executor 加入黑名单的阈值。达到阈值后，在执行这个 stage 时该节点将会被加入黑名单。 |
| spark.blacklist.application.maxFailedTasksPerExecutor | 2 | (实验性) 在同一个 executor 中，不同的 task的失败阈值 。达到阈值后，在整个 appliction 运行期间，该 executor 都会被加入黑名单，加入时间超过`spark.blacklist.timeout`后，自动从黑名单中移除。值得注意的是，如果开启了`dynamic allocation`，这些 executor 可能会由于空闲时间过长被回收。 |
| spark.blacklist.application.maxFailedExecutorsPerNode | 2 | (实验性) 在一个节点中，不同 executor 加入 application 黑名单的阈值。达到这个阈值后，该节点会进入 application 黑名单，加入时间超过`spark.blacklist.timeout`后，自动从黑名单中移除。值得注意的是，如果开启了`dynamic allocation`，该节点上的 executor 可能会由于空闲时间过长被回收。 |
| spark.blacklist.killBlacklistedExecutors | false | (实验性) 如果开启该配置，spark 会自动关闭并重启加入黑名单的 executor，如果整个节点都加入了黑名单，则该节点上的所有 executor 都会被关闭。 |
| spark.blacklist.application.fetchFailure.enabled | false | (实验性) 如果开启该配置，当发生 fetch failure时，立即将该 executor 加入到黑名单。要是开启了 external shuffle service，整个节点都会被加入黑名单。 |

因为黑名单机制目前还处于实验性状态，所以上面的一些参数可能会在后面的 Spark 中有所修改。

## 实现细节

### TaskSetBlacklist

黑名单账本：
```scala
//k:executor v:该executor上每个 task 的失败情况（task失败的次数和最近一次失败时间）
val execToFailures = new HashMap[String, ExecutorFailuresInTaskSet]()

//k:节点，v:该节点上有失败任务的 executor
private val nodeToExecsWithFailures = new HashMap[String, HashSet[String]]()
//k:节点, v:该节点上加入黑名单的 taskId
private val nodeToBlacklistedTaskIndexes = new HashMap[String, HashSet[Int]]()
  
//加入黑名单的 executor 
private val blacklistedExecs = new HashSet[String]()
//加入黑名单的 node
private val blacklistedNodes = new HashSet[String]()
```
```scala
// 判断 executor 是否加入了给定 task 的黑名单
def isExecutorBlacklistedForTask(executorId: String, index: Int): Boolean = {
    execToFailures.get(executorId).exists { execFailures =>
      execFailures.getNumTaskFailures(index) >= MAX_TASK_ATTEMPTS_PER_EXECUTOR
    }
}

//判断 node 是否加入了给定 task 的黑名单
def isNodeBlacklistedForTask(node: String, index: Int): Boolean = {
    nodeToBlacklistedTaskIndexes.get(node).exists(_.contains(index))
}
```
当有task失败时，TaskSetManager 会调用更新黑名单的操作：

1. 根据 `taskid` 更新 `excutor` 上该 `task` 的失败次数和失败时间
2. 判断 `task` 是否在该节点其他 `executor` 上有失败记录，如果有，将重试次数相加，如果 >= `MAX_TASK_ATTEMPTS_PER_NODE` ，则将该 `node` 加入这个 `taskId` 的黑名单
3. 判断在这个stage中，一个executor中失败的任务次数是否 >= `MAX_FAILURES_PER_EXEC_STAGE`，如果是，则将该 `executor` 加入这个 `stageId` 的黑名单
4. 判断在这个stage中，同一个 node 的 executor 的失败记录是否 >= `MAX_FAILED_EXEC_PER_NODE_STAGE`，如果是，则将该 `node` 加入这个 `stageId` 的黑名单

阈值参数：

*   MAX_TASK_ATTEMPTS_PER_EXECUTOR：每个 executor 上最大的任务重试次数
*   MAX_TASK_ATTEMPTS_PER_NODE：每个 node 上最大的任务重试次数
*   MAX_FAILURES_PER_EXEC_STAGE：一个 stage 中，每个executor 上最多任务失败次数
*   MAX_FAILED_EXEC_PER_NODE_STAGE：一个 stage 中，每个节点上 executor 的最多失败次数

```scala
private[scheduler] def updateBlacklistForFailedTask(
    host: String,
    exec: String,
    index: Int,
    failureReason: String): Unit = {
  latestFailureReason = failureReason
  val execFailures = execToFailures.getOrElseUpdate(exec, new ExecutorFailuresInTaskSet(host))
  execFailures.updateWithFailure(index, clock.getTimeMillis())

  val execsWithFailuresOnNode = nodeToExecsWithFailures.getOrElseUpdate(host, new HashSet())
  execsWithFailuresOnNode += exec
  val failuresOnHost = execsWithFailuresOnNode.toIterator.flatMap { exec =>
    execToFailures.get(exec).map { failures =>
      failures.getNumTaskFailures(index)
    }
  }.sum
  if (failuresOnHost >= MAX_TASK_ATTEMPTS_PER_NODE) {
    nodeToBlacklistedTaskIndexes.getOrElseUpdate(host, new HashSet()) += index
  }

  if (execFailures.numUniqueTasksWithFailures >= MAX_FAILURES_PER_EXEC_STAGE) {
    if (blacklistedExecs.add(exec)) {
      logInfo(s"Blacklisting executor ${exec} for stage $stageId")
      val blacklistedExecutorsOnNode =
        execsWithFailuresOnNode.filter(blacklistedExecs.contains(_))
      if (blacklistedExecutorsOnNode.size >= MAX_FAILED_EXEC_PER_NODE_STAGE) {
        if (blacklistedNodes.add(host)) {
          logInfo(s"Blacklisting ${host} for stage $stageId")
        }
      }
    }
  }
}
```
### BlacklistTracker
实现原理和`TaskSetBlacklist`，下文就不再贴出黑名单判断，黑名单对象等代码。
与 `TaskSetBlacklist` 不同的是，在一个 taskSet 完全成功之前，`BlacklistTracker` 无法获取到任务失败的情况。
当一个 taskSet 执行成功时会调用以下代码，流程如下：

1. 将每个 executor 上的 task 失败次数进行累计，如果 executor 最后一次 task 失败的时间超过 `BLACKLIST_TIMEOUT_MILLIS`，则移除该失败任务。
2. 如果 executor 上失败次数大于等于设定的阈值并且不在黑名单中

    * 将 `executor` 及其对应的到期时间加入到 `application` 的黑名单中，从executor失败列表中移除该 executor，并更新 `nextExpiryTime`，用于下次启动任务的时候判断黑名单是否已到期
    * 根据 `spark.blacklist.killBlacklistedExecutors` 判断是否要杀死 `executor`
    * 更新 `node` 上的 executor 失败次数
    * 如果一个节点上的 `executor` 的失败次数大于等于阈值并且不在黑名单中
        * 将 `node` 及其对应的到期时间加入到 `application` 的黑名单中
        * 如果开启了 `spark.blacklist.killBlacklistedExecutors`，则将此 `node` 上的所有 executor 杀死

* BLACKLIST_TIMEOUT_MILLIS：加入黑名单后的过期时间
* MAX_FAILURES_PER_EXEC：每个executor上最多的task失败次数
* MAX_FAILED_EXEC_PER_NODE: 每个节点上加入黑名单的executor的最大数量

```scala
def updateBlacklistForSuccessfulTaskSet(
    stageId: Int,
    stageAttemptId: Int,
    failuresByExec: HashMap[String, ExecutorFailuresInTaskSet]): Unit = {
  val now = clock.getTimeMillis()
  failuresByExec.foreach { case (exec, failuresInTaskSet) =>
    val appFailuresOnExecutor =
      executorIdToFailureList.getOrElseUpdate(exec, new ExecutorFailureList)
    appFailuresOnExecutor.addFailures(stageId, stageAttemptId, failuresInTaskSet)
    appFailuresOnExecutor.dropFailuresWithTimeoutBefore(now)
    val newTotal = appFailuresOnExecutor.numUniqueTaskFailures

    val expiryTimeForNewBlacklists = now + BLACKLIST_TIMEOUT_MILLIS
    if (newTotal >= MAX_FAILURES_PER_EXEC && !executorIdToBlacklistStatus.contains(exec)) {
      logInfo(s"Blacklisting executor id: $exec because it has $newTotal" +
        s" task failures in successful task sets")
      val node = failuresInTaskSet.node
      executorIdToBlacklistStatus.put(exec, BlacklistedExecutor(node, expiryTimeForNewBlacklists))
      listenerBus.post(SparkListenerExecutorBlacklisted(now, exec, newTotal))
      executorIdToFailureList.remove(exec)
      updateNextExpiryTime()
      killBlacklistedExecutor(exec)

      val blacklistedExecsOnNode = nodeToBlacklistedExecs.getOrElseUpdate(node, HashSet[String]())
      blacklistedExecsOnNode += exec
      if (blacklistedExecsOnNode.size >= MAX_FAILED_EXEC_PER_NODE &&
          !nodeIdToBlacklistExpiryTime.contains(node)) {
        logInfo(s"Blacklisting node $node because it has ${blacklistedExecsOnNode.size} " +
          s"executors blacklisted: ${blacklistedExecsOnNode}")
        nodeIdToBlacklistExpiryTime.put(node, expiryTimeForNewBlacklists)
        listenerBus.post(SparkListenerNodeBlacklisted(now, node, blacklistedExecsOnNode.size))
        _nodeBlacklist.set(nodeIdToBlacklistExpiryTime.keySet.toSet)
        killExecutorsOnBlacklistedNode(node)
      }
    }
  }
}
```
### 什么时候进行黑名单的判断

一个 stage 提交的调用链:

`TaskSchedulerImpl.submitTasks` ->
`CoarseGrainedSchedulerBackend.reviveOffers` ->
`CoarseGrainedSchedulerBackend.makeOffers` ->
`TaskSchedulerImpl.resourceOffers` ->
`TaskSchedulerImpl.resourceOfferSingleTaskSet` ->
`CoarseGrainedSchedulerBackend.launchTasks`

appliaction 级别的黑名单在 `TaskSchedulerImpl.resourceOffers` 中完成判断，stage/task 级别的黑名单在 `TaskSchedulerImpl.resourceOfferSingleTaskSet` 中完成判断。

### 如果所有的节点都被加入了黑名单？

如果将task的重试次数设置的比较高，有可能会出现这个问题，这个时候。将会中断这个 stage 的执行

TaskSchedulerImpl.resourceOffers

```scala
if (!launchedAnyTask) {
  taskSet.abortIfCompletelyBlacklisted(hostToExecutors)
}
```

## 结语

简单的来说，对于一个 application ，提供了三种级别的黑名单可以用于 executor/node： task blacklist -> stage blacklist -> application blacklist

通过这些黑名单的设置可以避免由于 task 反复调度在有问题的 `executor/node` （坏盘，磁盘满了，shuffle fetch 失败，环境错误等）上，进而导致整个 `Application` 运行失败的情况。

tips: `BlacklistTracker.updateBlacklistForFetchFailure` 存在BUG [SPARK-24021](https://github.com/apache/spark/commit/7fb11176f285b4de47e61511c09acbbb79e5c44c)，如果打开了 `spark.blacklist.application.fetchFailure.enabled` 配置将会受到影响。