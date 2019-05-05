---
layout:     post
title:      "Spark Web UI详解"
subtitle:   " understanding your spark application through visualization"
date:       2017-01-10 10:00:00 +0800
author:     "Jay Cai"
header-style: text
catalog: true
tags:
    - Spark
---

# Spark Web UI详解

![](/img/posts/sparkwebui/ApacheSparkLogo.png)

SparkのWeb UIに記載されている項目の意味について（日本語で）まとまっている情報がなかったのでまとめてみました。(Spark 1.6ベース）
Spark 2.xへの対応と、SparkSQL、SparkStreamingは別途記載する予定。
間違いを見つけたらコメントお願いします。


## 1\. メイン画面

アプリケーションに関する情報を表示
![sparkui1](/img/posts/sparkwebui/sparkui1.png)

<style>
table th:first-of-type {
    width: 80px;
}
th:nth-of-type(2) {
    width: 200px;
}
</style>

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | Jobs | ジョブ一覧（このページ）へのリンク |
| 2 | Stages | ステージ一覧へのリンク |
| 3 | Storage | ストレージ（persistの情報）へのリンク |
| 4 | Environment | 環境(Environment)情報へのリンク |
| 5 | Executors | エグゼキュータ情報へのリンク |
| 6 | – | このアプリケーションの名前(spark.app.name) |
| 7 | ? | ジョブについての説明をポップアップで表示 |
| 8 | Total Uptime: | このジョブの稼働時間（実際の処理時間ではない） |
| 9 | Scheduling Mode: | スケジューリングモード：FAIR / FIFO / NONE のいずれか。 (spark.scheduler.mode) |
| 10 | Active Jobs / Completed Jobs / Failed Jobs | それぞれの数と一覧へのリンク |
| 11 | Event Timeline | 時系列でイベントを表示（クリックで表示） |
| 12 | Job Id: | ジョブのID |
| 13 | Description: | ジョブの詳細へのリンク。最後のステージのアクション名が表示される |
| 14 | Submitted: | ジョブ投入日時 |
| 15 | Duration: | 各ステージにかかった時間の合計 |
| 16 | Stages: Succeeded/Total: | このジョブで成功したステージ数/合計ステージ数 |
| 17 | Tasks (for all stages): Succeeded/Total | このジョブで成功したタスク数/合計タスク数 |

#### 1.1\. ジョブ全体のイベントタイムライン

![eventtimline](/img/posts/sparkwebui/eventtimline.png)このジョブのエグゼキュータおよびジョブの開始、かかった時間、失敗、成功など情報を時系列で表示。チェックボックスにチェックすることで、タイムラインの拡大/縮小も可能。
この図では、Executor/Driverが登録され、その後２つのジョブが順次実行されていることがわかる。

![eventtimeline2](/img/posts/sparkwebui/eventtimeline2.png)また、Enable zoomingを有効にした状態で、イベントタイムラインでジョブにマウスカーソルを合わせるとジョブの情報がポップアップされ、該当するJob Idの行が緑にハイライト表示される
イベントタイムラインの詳細は参考資料 <sup>[1]</sup> を参照

## 2\. ジョブの詳細

ジョブに関する情報を表示
![sparkui_job_details](/img/posts/sparkwebui/sparkui_job_details.png)

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | Details for Job | ジョブ番号の詳細 |
| 2 | Status | ジョブの状態 (RUNNING / SUCCEEDED / FAILED / UNKNOWN ) |
| 3 | Active Stages: \<n\>  Pending Stages: \<n\>  Completed Stages: \<n\>  Skipped Stages: \<n\>  Failed Stages: \<n\> | 各状態 (Active / Pending / Completed / Skipped / Failed) でのそれぞれのステージの数 |
| 4 | Event Timeline | 時系列でイベントを表示（クリックで表示） |
| 5 | DAG Visualization | DAG(有向非循環グラフ）を可視化して表示 |
| 6 | Active Stages (n)  Completed Stages (n)  Skipped Stages (n)  Failed Stages (n) | 各状態（Active / Completed / Skipped / Failed）におけるステージ一覧。カッコ内の n はステージ数 |
| 7 | Stage Id | ステージのID |
| 8 | Description | ステージ詳細のリンク。アクションが表示される |
| 9 | +details | そのステージの詳細を表示 |
| 10 | Submitted | ジョブ投入時間 |
| 11 | Duration | 経過時間 |
| 12 | Tasks: Succeeded/Total | 成功したタスク数/合計タスク数 |
| 13 | Input | 入力サイズ（各タスクの合計）。中間データからの入力は含まない。<br>内訳は、メモリ（永続化されたRDDからの入力）、ディスク（に永続化されたRDDからの入力）、Hadoop (textFile()などからの入力)、Network (Networkとはリモートのブロックマネージャーから読み込まれたデータ）<sup>[4]</sup> |
| 14 | Output | 出力サイズ（各タスクの合計）。中間データへの出力は含まない。<br>内訳はHadoop (saveAsTextFile()などへの出力）<sup>[5]</sup> |
| 15 | Shuffle Read | シャッフルで読み込んだデータのサイズ（各タスクの合計） |
| 16 | Shuffle Write | シャッフルで書き出したデータのサイズ（各タスクの合計） |

#### 2.1\. イベントタイムライン

そのジョブでのエグゼキュータおよびジョブの開始、かかった時間、失敗、成功など情報を時系列で表示。チェックボックスにチェックすることで、タイムラインの拡大/縮小も可能。

#### 2.2\. DAGのビジュアル化

そのジョブのDAG（有向非循環グラフ）を表示。各ステージにおけるRDDの関連性がわかる。

![spark_eventtimeline](/img/posts/sparkwebui/spark_eventtimeline.png)

イベントタイムラインの「イベント」、またはDAG Vidualizationのボックスをクリックすると、各ステージの詳細画面に遷移する。

なお、DAG Visualizationでグラフの中心が緑色のドットになっているRDDは「永続化」されているという意味。（下記の図を参照）

![sparkurl_persist1](/img/posts/sparkwebui/sparkurl_persist1.png)

イベントタイムラインの詳細は参考資料 <sup>[1]</sup> を参照

#### 2.3\. +details

各ステージのスタックトレースを表示/非表示

![sparkurl_stages](/img/posts/sparkwebui/sparkurl_stages.png)

## 3\. ステージの詳細

ステージに関する情報を表示
![sparkui_stage_details](/img/posts/sparkwebui/sparkui_stage_details.png)

| 号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | Details for Stage \<n\> (Attempt <m>) | ステージ \<n\> の詳細。試行ID m |
| 2 | Total Time Across All Tasks: | 全タスクに渡っての合計時間 |
| 3 | Locality Level Summary | 各タスクのローカリティレベル（とその数）。例： Node local: 18, Process local 3  ローカリティレベルは、Process local / Node local / Rack local / Any のいずれか。 |
| 4 | Input Size / Records: | 入力サイズとレコード数（全タスクの合計） |
| 5 | Shuffle Read:  <br>Shuffle Write: | シャッフルでの読み込みとレコード数　（スクリーンショットでは非表示）  シャッフルでの書き込みとレコード数 |
| 6 | DAG Visualization | DAG （有向非循環グラフ）を可視化して表示 |
| 7 | Show Additional Metircs | 追加のメトリクスを表示（後述） |
| 8 | Event Timeline | 時系列でイベントを表示（クリックで表示） |
| 9 | Summary Metrics for <n> Completed Tasks | 完了した n 個のタスクのメトリクス情報の要約。最小 / 25パーセンタイル / 平均 / 75パーセンタイル / 最大（後述） |
| 10 | Aggregated Metrics by Executor | エグゼキュータによる集約したメトリクス情報　（後述） |
| 11 | Tasks | 各タスクごとのメトリクス情報（後述） |

#### 3.1\. DAGのビジュアル化

そのステージでのDAG（有向非循環グラフ）を表示。各ステージにおけるRDDのリネージの関連性がわかる。toDebugStringのビジュアル化したものと考えると判りやすいかな。

#### 3.2 追加のメトリクスの表示

追加メトリクスを表示するためのチェックボックスがある。追加できるメトリクスの種類には、スケジューリングの遅延、タスクの非シリアル化にかかった時間、結果のシリアル化にかかった時間、結果を取得するのにかかった時間、ピークの実行時メモリを表示する、がある

#### 3.3\. イベントタイムライン

そのステージで、スケジュールの遅延、実行などにかかった時間を色分けして時系列に表示。Enable Scalingのチェックボックスにチェックをすることで、タイムラインの拡大/縮小も可能。

![sparkui_event_timeline](/img/posts/sparkwebui/sparkui_event_timeline.png)

#### 3.4\. タスクのメトリクスのサマリ

当該ステージでのタスクのメトリクスに関する情報。(Summary Metrics for n Completed Tasks)
[![sparkui_task_metrics](/img/posts/sparkwebui/sparkui_task_metrics.png)
※サマリとタスクには同一の項目がある

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| a | Min | 最小 |
| b | 25th percentile | 25パーセンタイル |
| c | Median | 中央値 |
| d | 75th percentile | 75パーセンタイル |
| e | Max | 最大 |

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | Duration | エグゼキュタ上でタスクの実行にかかった時間 (タスクのexecutorRunTimeメトリクスに基づく) |
| 2 | Scheduler Delay | スケジューラからエグザキュータにタスクを送るのにかかった時間と、エグゼキュータからスケジューラにタスクの結果を送るのにかかった時間。<br>ここにかかっている時間が長い場合、タスクのサイズまたはタスクの結果のサイズを減らすことを考慮する |
| 3 | Task Deserialization Time | エグゼキュータ上でタスクのでシリアライズにかかった時間。executorDeserializeTimeメトリクスに基づいた値 |
| 4 | GC Time | タスク実行中にエグゼキュータがJavaのガベージコレクションで一時停止に費やした時間。タスクのjvmGCTimeメトリクスに基づく |
| 5 | Result Serialization Time | タスクの結果をドライバに返す前に、エグゼキュータでタスクの結果をシリアライズするのにかかった時間。resultSerializationTimeメトリクスに基づく |
| 6 | Getting Result Time | ドライバーがWorker群からのタスクの結果をフェッチするのにかかった時間。ここにかかっている時間が長い場合、各タスクから返されるデータ量を減らすことを考慮する |
| 7 | Peak Execution Memory | シャッフル、集約、ジョインの間に作成された内部データ構造の最大サイズの合計。<br>peakExecutionMemoryメトリクスに基づく。SQLのジョブでは、すべてのunsafeな操作、ブロードキャストジョイン、外部ソートのみを追跡。 |
| 8 | Input Size / Records | ステージに入力がある場合、HadoopやSparkのストレージ (sc.textFileなど。HDFSやローカルファイルシステム）から読み込んだバイト数とレコード数。inputMetrics.bytesReadとinputMetrics.recordsReadメトリクスに基づく。 |
| 9 | Shuffle Write Size / Records | ステージで出力がある場合、HadoopやSparkのストレージ (sc.saveAsTextFileなど。HDFSやローカルファイルシステム）に書き込んだバイト数とレコード数。outputMetrics.bytesReadとoutputMetrics.recordsReadメトリクスに基づく。 |

#### 3.5\. エグゼキュータで集計したメトリクス

当該ステージでの各エグゼキュータで集計したメトリクスの情報。(Aggregated Metrics by Executor)
※この例はローカルモードで実行しているので分散環境の表示とは若干異なる。近いうちに更新予定。
![sparkui_aggregated_metrics](/img/posts/sparkwebui/sparkui_aggregated_metrics.png)

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | Executor ID | エグゼキュータのID。通常は0、1のような数字になる |
| 2 | Address | エグゼキュータを実行しているホストのアドレス。（例: 192.168.1.10:60233) |
| 3 | Task Time |
| 4 | Total Tasks | 当該エグゼキュータ上で実行されたタスク数の合計 |
| 5 | Failed Tasks | 失敗したタスク数 |
| 6 | Succeeded Tasks | 成功したタスク数 |
| 7 | Input Sizes / Records | ステージに入力がある場合、当該エグゼキュータ上にてHadoopやSparkのストレージ (sc.textFileなど。HDFSやローカルファイルシステム）から読み込んだバイト数とレコード数。<br>inputMetrics.bytesReadとinputMetrics.recordsReadメトリクスに基づく。   |
| 8 | Shuffle Write Size / Records | ステージにシャッフルで書き込むステージがある場合、ローカルへの書き込みとリモートエグゼキュータへの書き込みを含んだ、シャッフルのバイト数とレコード数の合計。<br>shuffleWriteMetrics.bytesWrittenとshuffleWriteMetrics.recordsWrittenに基づく |

#### 3.6\. タスク毎のメトリクス

当該ステージでの各タスクのメトリクスの情報。
①〜⑨までは3.4、タスクのメトリクスのサマリと同じ。
![sparkui_task_details](/img/posts/sparkwebui/sparkui_task_details.png)

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1〜9 | --- | **3.4. タスクのメトリクスのサマリを参照** |
| 10 | Index | 項番 |
| 11 | ID | 当該ステージ中でのユニークなタスクID |
| 12 | Attempt | 当該ステージ中でのユニークなタスク試行ID |
| 5 | Status | タスクのステータス (SUCCESS / GET RESULT / RUNNING / FAILED  / KILLED / UNKOWN のいずれか<sup>[7]</sup>) |
| 6 | Locality Level | 本タスクのローカリティレベル (PROCESS_LOCAL / NODE_LOCAL / RACK_LOCAL / ANY のいずれか) |
| 7 | Executor ID / Host | エグゼキュータID / ホスト。分散環境では 0 / 192.168.1.10 のようになる |
| 8 | Input Size / Records | ステージに入力がある場合、HadoopやSparkのストレージ (sc.textFileなど。HDFSやローカルファイルシステム）から読み込んだバイト数とレコード数。<br>inputMetrics.bytesReadとinputMetrics.recordsReadメトリクスに基づく。 |
| 9 | Write Time | シャッフルの書き込みにかかった時間 <sup>[8]</sup> |
| 10 | Errors | エラーがあった場合エラーの最初の行が表示される。クリックすると展開される（?）<sup>[9]</sup> |

## 4\. 永続化に関する情報 (Storage)

永続化に関する情報を表示。
DFSReadWriteTestは永続化を行わないため、下記のようなコードを書いて実行した。
```scala
val r1 = sc.textFile(“hosts”)
r1.cache()
val r2 = sc.textFile(“group”)
r2.persist(org.apache.spark.storage.StorageLevel.DISK_ONLY_2)
val r3 = sc.parallelize(Array(1,2,3))
r3.persist(org.apache.spark.storage.StorageLevel.MEMORY_ONLY_SER)
r1.union(r2.union(r3)).count()
r1.count()
r2.count()
r3.count()
```
![sparkui_storage](/img/posts/sparkwebui/sparkui_storage.png)

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | RDD Name | RDDの名前 |
| 2 | Storage Level | ストレージレベル (MEMORYまたはDISK、およびシリアライズされているか否か) と複製数 |
| 3 | Cached Partitions | キャッシュしたパーティションの数 |
| 4 | Fraction Cached | RDDのパーティション数に対するキャッシュしたパーティション数の割合。rdd.numCachedPartitions * 100.0 / rdd.numPartitions |
| 5 | Size in Memory | メモリにキャッシュしたサイズ |
| 6 | Size in ExternalBlockStore | External Block store (Tachyonなど）にキャッシュしたサイズ。オフヒープの設定がされていればStorageLevel.OFF_HEAPで表示される模様（未検証） |
| 7 | Size on Disk | ディスク上にキャッシュしたサイズ |

#### 4.1\. RDD毎のストレージ情報

永続化で一覧されたRDDのうち、指定された特定のRDDのストレージ情報
ほとんどの項目の意味は永続化に関する情報と同じ。

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | Storage Level | ストレージレベル (MEMORYまたはDISK、およびシリアライズされているか否か) と複製数 |
| 2 | Cached Partitions | RDDの中でキャッシュされたパーティションの数 |
| 3 | Total Partitions | RDD全体のパーティション数 |
| 4 | Memory Size | メモリにキャッシュしたサイズ |
| 5 | Disk Size | ディスク上にキャッシュしたサイズ |
| 6 | Data Distribution on \<n\> Executors | データが \<n\> 個のエグゼキュタ上に分散しているという説明 |
| 7 | Host | エグゼキュータのホスト名とポート番号 |
| 8 | Memory Usage | メモリ上のキャッシュの使用量（残容量） |
| 9 | Disk Usage | ディスク上のキャッシュの使用量 |
| 10 | <n> Partitions | キャッシュした（？）<n> 個のパーティションがあるという説明 |
| 11 | Block Name | ブロックの名前。ディスクに永続化した場合はファイル名になる |
| 12 | Storage Level | ストレージレベル (MEMORYまたはDISK、およびシリアライズされているか否か) と複製数 |
| 13 | Size in Memory | メモリにキャッシュしたサイズ |
| 14 | Size on Disk | ディスク上にキャッシュしたサイズ |
| 15 | Executors | 保持しているエグゼキュータ |

## 5\. 環境（Environment）

環境（JDKのバージョンやプロパティ）に関する情報を表示。
[![sparkui_env](/img/posts/sparkwebui/sparkui_env.png)

## 6\. エグゼキュータ（Executors）

エグゼキュータのサマリと各エグゼキュータの情報を表示。
この例はpersist()時にOFF_HEAPを指定して失敗させた結果が表示されている。
[![sparkui_executors](/img/posts/sparkwebui/sparkui_executors.png)

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| a | Active (n) | アクティブなエグゼキュータ（の数） |
| b | Dead (n) | 死んでいるエグゼキュータ（の数） |
| c | Total (n) | Active + Dead なエグゼキュータ（の数） |

| 番号 | 画面での表示 | 説明 |
| --- | --- | --- |
| 1 | RDD Blocks | RDDブロックの数 |
| 2 | Storage Memory | メモリ使用量（と全体の容量） |
| 3 | Disk Used | ディスク使用量 |
| 4 | Core | CPUコア |
| 5 | Active Tasks | アクティブなタスク数 |
| 6 | Failed Tasks | 失敗したタスク数 |
| 7 | Complete Tasks | 完了したタスク数 |
| 8 | Total Tasks | タスク数の合計 |
| 9 | Task Time (GC Time) | タスク実行時間（Javaのガベージコレクションで一時停止に費やした時間） |
| 10 | Input | HDFSなどから入力の際に読み込んだバイト数 |
| 11 | Shuffle Read | シャッフルで読み込んだデータのサイズ |
| 12 | Shuffle Write | シャッフルで書き出したデータのサイズ |
| 13 | Executor ID | エグゼキュータのID。分散環境の場合は0, 1のような数字になる |
| 14 | Address | エグゼキュータを実行しているホストのアドレス。（例: 192.168.1.10:60233) |
| 15 | Status | エグゼキュータのステータス（Active / Dead) |
| 16 | Thread Dump | スレッドダンプ（後述）へのリンク |

#### 6.1 エグゼキュータのスレッドダンプ

エグゼキュータのスレッドダンプを表示
[![sparkui_executor_threaddump](/img/posts/sparkwebui/sparkui_executor_threaddump.png)

## 参考資料

[1] [https://databricks.com/blog/2015/06/22/understanding-your-spark-application-through-visualization.html](https://databricks.com/blog/2015/06/22/understanding-your-spark-application-through-visualization.html)
[2] [https://github.com/apache/spark/blob/f2ceb2abe9357942a51bd643683850efd1fc9df7/core/src/main/scala/org/apache/spark/ui/jobs/AllJobsPage.scala](https://github.com/apache/spark/blob/f2ceb2abe9357942a51bd643683850efd1fc9df7/core/src/main/scala/org/apache/spark/ui/jobs/AllJobsPage.scala)
[3] [https://github.com/apache/spark/blob/f2ceb2abe9357942a51bd643683850efd1fc9df7/core/src/main/scala/org/apache/spark/ui/jobs/StageTable.scala](https://github.com/apache/spark/blob/f2ceb2abe9357942a51bd643683850efd1fc9df7/core/src/main/scala/org/apache/spark/ui/jobs/StageTable.scala)
[4] [https://github.com/apache/spark/blob/39e2bad6a866d27c3ca594d15e574a1da3ee84cc/core/src/main/scala/org/apache/spark/executor/InputMetrics.scala](https://github.com/apache/spark/blob/39e2bad6a866d27c3ca594d15e574a1da3ee84cc/core/src/main/scala/org/apache/spark/executor/InputMetrics.scala)
[5] [https://github.com/apache/spark/blob/39e2bad6a866d27c3ca594d15e574a1da3ee84cc/core/src/main/scala/org/apache/spark/executor/OutputMetrics.scala](https://github.com/apache/spark/blob/39e2bad6a866d27c3ca594d15e574a1da3ee84cc/core/src/main/scala/org/apache/spark/executor/OutputMetrics.scala)
[6] [https://github.com/apache/spark/blob/dbf842b7a8479f9566146192ffc04421591742d5/core/src/main/scala/org/apache/spark/ui/jobs/StagePage.scala](https://github.com/apache/spark/blob/dbf842b7a8479f9566146192ffc04421591742d5/core/src/main/scala/org/apache/spark/ui/jobs/StagePage.scala)
[7] [https://github.com/apache/spark/blob/3a710b94b0c853a2dd4c40dca446ecde4e7be959/core/src/main/scala/org/apache/spark/scheduler/TaskInfo.scala](https://github.com/apache/spark/blob/3a710b94b0c853a2dd4c40dca446ecde4e7be959/core/src/main/scala/org/apache/spark/scheduler/TaskInfo.scala)
[8] [https://github.com/apache/spark/blob/39e2bad6a866d27c3ca594d15e574a1da3ee84cc/core/src/main/java/org/apache/spark/storage/TimeTrackingOutputStream.java#L45](https://github.com/apache/spark/blob/39e2bad6a866d27c3ca594d15e574a1da3ee84cc/core/src/main/java/org/apache/spark/storage/TimeTrackingOutputStream.java#L45)
[9] [https://github.com/apache/spark/blob/dbf842b7a8479f9566146192ffc04421591742d5/core/src/main/scala/org/apache/spark/ui/jobs/StagePage.scala#L1335](https://github.com/apache/spark/blob/dbf842b7a8479f9566146192ffc04421591742d5/core/src/main/scala/org/apache/spark/ui/jobs/StagePage.scala#L1335)
[10] [https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-webui-StagePage.html](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-webui-StagePage.html)
