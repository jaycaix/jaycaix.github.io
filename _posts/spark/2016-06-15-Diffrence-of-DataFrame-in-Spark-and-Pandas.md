---
layout: post
title: "Spark与Pandas中DataFrame对比"
subtitle:   " Diffrence of DataFrame in Spark and Pandas "
date:       2016-06-15 11:00:00 +0800
author: "Jay Cai"
header-style: text
catalog: true
tags:
  - Spark
  - Pandas
  - Python
---


## Spark与Pandas中DataFrame对比
<table width="1704"><tbody><tr><td width="188">&nbsp;</td>
<td width="638">Pandas</td>
<td width="878">Spark</td>
</tr><tr><td>工作方式</td>
<td width="638">单机single machine tool，没有并行机制parallelism<br>
不支持Hadoop，处理大量数据有瓶颈</td>
<td width="878">分布式并行计算框架，内建并行机制parallelism，所有的数据和操作自动并行分布在各个集群结点上。以处理in-memory数据的方式处理distributed数据。<br>
支持Hadoop，能处理大量数据</td>
</tr><tr><td>延迟机制</td>
<td>not lazy-evaluated</td>
<td>lazy-evaluated</td>
</tr><tr><td>内存缓存</td>
<td>单机缓存</td>
<td width="878">persist() or cache()将转换的RDDs保存在内存</td>
</tr><tr><td>DataFrame可变性</td>
<td>Pandas中DataFrame是可变的</td>
<td width="878">Spark中RDDs是不可变的，因此DataFrame也是不可变的</td>
</tr><tr><td rowspan="6">创建</td>
<td>从spark_df转换：pandas_df = spark_df.toPandas()</td>
<td width="878">从pandas_df转换：spark_df = SQLContext.createDataFrame(pandas_df)<br>
另外，createDataFrame支持从list转换spark_df，其中list元素可以为tuple，dict，rdd</td>
</tr><tr><td>list，dict，ndarray转换</td>
<td>已有的RDDs转换</td>
</tr><tr><td>CSV数据集读取</td>
<td>结构化数据文件读取</td>
</tr><tr><td>HDF5读取</td>
<td>JSON数据集读取</td>
</tr><tr><td>EXCEL读取</td>
<td>Hive表读取</td>
</tr><tr><td>&nbsp;</td>
<td>外部数据库读取</td>
</tr><tr><td>index索引</td>
<td>自动创建</td>
<td>没有index索引，若需要需要额外创建该列</td>
</tr><tr><td>行结构</td>
<td>Series结构，属于Pandas DataFrame结构</td>
<td>Row结构，属于Spark DataFrame结构</td>
</tr><tr><td>列结构</td>
<td>Series结构，属于Pandas DataFrame结构</td>
<td width="878">Column结构，属于Spark DataFrame结构，如：DataFrame[name: string]</td>
</tr><tr><td>列名称</td>
<td>不允许重名</td>
<td width="878">允许重名<br>
修改列名采用alias方法</td>
</tr><tr><td>列添加</td>
<td>df[“xx”] = 0</td>
<td width="878">df.withColumn(“xx”, 0).show() 会报错<br>
from pyspark.sql import functions<br>
df.withColumn(“xx”, functions.lit(0)).show()</td>
</tr><tr><td>列修改</td>
<td>原来有df[“xx”]列，df[“xx”] = 1</td>
<td width="878">原来有df[“xx”]列，df.withColumn(“xx”, 1).show()</td>
</tr><tr><td rowspan="4">显示</td>
<td>&nbsp;</td>
<td width="878">df 不输出具体内容，输出具体内容用show方法<br>
输出形式：DataFrame[age: bigint, name: string]</td>
</tr><tr><td width="638">df 输出具体内容</td>
<td>df.show() 输出具体内容</td>
</tr><tr><td>没有树结构输出形式</td>
<td width="878">以树的形式打印概要：df.printSchema()</td>
</tr><tr><td>&nbsp;</td>
<td width="878">df.collect()</td>
</tr><tr><td rowspan="2">排序</td>
<td>df.sort_index() 按轴进行排序</td>
<td width="878">&nbsp;</td>
</tr><tr><td>df.sort() 在列中按值进行排序</td>
<td width="878">df.sort() 在列中按值进行排序</td>
</tr><tr><td rowspan="8">选择或切片</td>
<td>df.name 输出具体内容</td>
<td width="878">df[] 不输出具体内容，输出具体内容用show方法<br>
df[“name”] 不输出具体内容，输出具体内容用show方法</td>
</tr><tr><td width="638">df[] 输出具体内容，<br>
df[“name”] 输出具体内容</td>
<td width="878">df.select() 选择一列或多列<br>
df.select(“name”)<br>
切片 df.select(df[‘name’], df[‘age’]+1)</td>
</tr><tr><td width="638">df[0]<br>
df.ix[0]</td>
<td>df.first()</td>
</tr><tr><td width="638">df.head(2)</td>
<td width="878">df.head(2)或者df.take(2)</td>
</tr><tr><td width="638">df.tail(2)</td>
<td width="878">&nbsp;</td>
</tr><tr><td width="638">切片 df.ix[:3]或者df.ix[:”xx”]或者df[:”xx”]</td>
<td>&nbsp;</td>
</tr><tr><td width="638">df.loc[] 通过标签进行选择</td>
<td>&nbsp;</td>
</tr><tr><td width="638">df.iloc[] 通过位置进行选择</td>
<td>&nbsp;</td>
</tr><tr><td>过滤</td>
<td>df[df[‘age’]&gt;21]</td>
<td>df.filter(df[‘age’]&gt;21) 或者 df.where(df[‘age’]&gt;21)</td>
</tr><tr><td>整合</td>
<td width="638">df.groupby(“age”)<br>
df.groupby(“A”).avg(“B”)</td>
<td width="878">df.groupBy(“age”)<br>
df.groupBy(“A”).avg(“B”).show() 应用单个函数<br>
from pyspark.sql import functions<br>
df.groupBy(“A”).agg(functions.avg(“B”), functions.min(“B”), functions.max(“B”)).show() 应用多个函数</td>
</tr><tr><td rowspan="2">统计</td>
<td>df.count() 输出每一列的非空行数</td>
<td>df.count() 输出总行数</td>
</tr><tr><td width="638">df.describe() 描述某些列的count, mean, std, min, 25%, 50%, 75%, max</td>
<td width="878">df.describe() 描述某些列的count, mean, stddev, min, max</td>
</tr><tr><td rowspan="4">合并</td>
<td>Pandas下有concat方法，支持轴向合并</td>
<td>&nbsp;</td>
</tr><tr><td width="638">Pandas下有merge方法，支持多列合并<br>
同名列自动添加后缀，对应键仅保留一份副本</td>
<td width="878">Spark下有join方法即df.join()<br>
同名列不自动添加后缀，只有键值完全匹配才保留一份副本</td>
</tr><tr><td>df.join() 支持多列合并</td>
<td>&nbsp;</td>
</tr><tr><td>df.append() 支持多行合并</td>
<td>&nbsp;</td>
</tr><tr><td rowspan="3">缺失数据处理</td>
<td>对缺失数据自动添加NaNs</td>
<td>不自动添加NaNs，且不抛出错误</td>
</tr><tr><td>fillna函数：df.fillna()</td>
<td>fillna函数：df.na.fill()</td>
</tr><tr><td>dropna函数：df.dropna()</td>
<td>dropna函数：df.na.drop()</td>
</tr><tr><td rowspan="2">SQL语句</td>
<td rowspan="2" width="638">import sqlite3<br>
pd.read_sql(“SELECT name, age FROM people WHERE age &gt;= 13 AND age &lt;= 19″)</td>
<td width="878">表格注册：把DataFrame结构注册成SQL语句使用类型<br>
df.registerTempTable(“people”) 或者 sqlContext.registerDataFrameAsTable(df, “people”)<br>
sqlContext.sql(“SELECT name, age FROM people WHERE age &gt;= 13 AND age &lt;= 19″)</td>
</tr><tr><td width="878">功能注册：把函数注册成SQL语句使用类型<br>
sqlContext.registerFunction(“stringLengthString”, lambda x: len(x))<br>
sqlContext.sql(“SELECT stringLengthString(‘test’)”)</td>
</tr><tr><td>两者互相转换</td>
<td>pandas_df = spark_df.toPandas()</td>
<td>spark_df = sqlContext.createDataFrame(pandas_df)</td>
</tr><tr><td>函数应用</td>
<td>df.apply(f）将df的每一列应用函数f</td>
<td width="878">df.foreach(f) 或者 df.rdd.foreach(f) 将df的每一列应用函数f<br>
df.foreachPartition(f) 或者 df.rdd.foreachPartition(f) 将df的每一块应用函数f</td>
</tr><tr><td>map-reduce操作</td>
<td>map(func, list)，reduce(func, list) 返回类型seq</td>
<td>df.map(func)，df.reduce(func) 返回类型seqRDDs</td>
</tr><tr><td>diff操作</td>
<td>有diff操作，处理时间序列数据（Pandas会对比当前行与上一行）</td>
<td>没有diff操作（Spark的上下行是相互独立，分布式存储的）</td>
</tr></tbody></table>

## 原文链接
[Spark与Pandas中DataFrame对比](http://www.lining0806.com/spark%E4%B8%8Epandas%E4%B8%ADdataframe%E5%AF%B9%E6%AF%94/)