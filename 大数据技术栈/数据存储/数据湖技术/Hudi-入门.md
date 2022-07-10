尚硅谷大数据技术之Hudi

(作者：尚硅谷大数据研发部)

版本：V1.0 

# 第1章 Hudi介绍

## 1.1介绍

   Hudi将带来流式处理大数据， 提供新数据集，同时比传统批处理效率高一个数据量级。

![img](file:////tmp/wps-gree/ksohtml/wps5nugzO.jpg) 

 

## 1.2特性

（1）快速upsert,可插入索引

（2）以原子方式操作数据并具有回滚功能

（3）写入器之间的快照隔离

（4）savepoint用户数据恢复的保存点

（5）管理文件大小，使用统计数据布局

（6）数据行的异步压缩和柱状数据

（7）时间轴数据跟踪血统

# 第2章 Hudi快速构建

## 2.1 安装环境准备

Hadoop集群

Hive

Spark2.4.5(2.x)

### 2.1.1 Maven安装

（1）把apache-maven-3.6.1-bin.tar.gz上传到linux的/opt/software目录下

（2）解压apache-maven-3.6.1-bin.tar.gz到/opt/module/目录下面

[atguigu@hadoop102 software]$ tar -zxvf apache-maven-3.6.1-bin.tar.gz -C /opt/module/

（3）修改apache-maven-3.6.1的名称为maven

[atguigu@hadoop102 module]$ mv apache-maven-3.6.1/ maven

（4）添加环境变量到/etc/profile中

[atguigu@hadoop102 module]$ sudo vim /etc/profile

\#MAVEN_HOME

export MAVEN_HOME=/opt/module/maven

export PATH=$PATH:$MAVEN_HOME/bin

（5）测试安装结果

[atguigu@hadoop102 module]$ source /etc/profile

[atguigu@hadoop102 module]$ mvn -v

（6）修改setting.xml，指定为阿里云

[atguigu@hadoop102 maven]$ cd conf

[atguigu@hadoop102 maven]$ vim settings.xml

<!-- 添加阿里云镜像-->

<mirror>

​    <id>nexus-aliyun</id>

​    <mirrorOf>central</mirrorOf>

​    <name>Nexus aliyun</name>

​    <url>http://maven.aliyun.com/nexus/content/groups/public</url>

</mirror>

![img](file:////tmp/wps-gree/ksohtml/wpslTlLFQ.jpg) 

### 2.1.2 Git安装

[atguigu@hadoop102 software]$ sudo yum install git

[atguigu@hadoop102 software]$ git --version

### 2.1.3 构建hudi

[atguigu@hadoop102 software]$ cd /opt/module/

[atguigu@hadoop102 module]$ git clone https://github.com/apache/hudi.git && cd hudi

[atguigu@hadoop102 hudi]$ vim pom.xml 

​    <repository>

​    <id>nexus-aliyun</id>

​    <name>nexus-aliyun</name>

​    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>

​    <releases>

​      <enabled>true</enabled>

​    </releases>

​    <snapshots>

​      <enabled>false</enabled>

​    </snapshots>

  </repository>

[atguigu@hadoop102 hudi]$ mvn clean package -DskipTests -DskipITs

# 第3章 通过Spark-shell快速开始

## 3.1 Spark-shell启动

spark-shell启动,需要指定spark-avro模块，因为默认环境里没有，spark-avro模块版本好需要和spark版本对应，这里都是2.4.5。

[root@hadoop102 hudi]# spark-shell --packages org.apache.spark:spark-avro_2.11:2.4.5 --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' --jars /opt/module/hudi/packaging/hudi-spark-bundle/target/hudi-spark-bundle_2.11-0.6.0-SNAPSHOT.jar 

 

## 3.2 设置表名

设置表名，基本路径和数据生成器

scala> import org.apache.hudi.QuickstartUtils._

import org.apache.hudi.QuickstartUtils._

 

scala> import scala.collection.JavaConversions._

import scala.collection.JavaConversions._

 

scala> import org.apache.spark.sql.SaveMode._

import org.apache.spark.sql.SaveMode._

 

scala> import org.apache.hudi.DataSourceReadOptions._

import org.apache.hudi.DataSourceReadOptions._

 

scala> import org.apache.hudi.DataSourceWriteOptions._

import org.apache.hudi.DataSourceWriteOptions._

 

scala> import org.apache.hudi.config.HoodieWriteConfig._

import org.apache.hudi.config.HoodieWriteConfig._

 

scala> val tableName = "hudi_trips_cow"

tableName: String = hudi_trips_cow

 

scala> val basePath = "file:///tmp/hudi_trips_cow"

basePath: String = file:///tmp/hudi_trips_cow

 

scala> val dataGen = new DataGenerator

dataGen: org.apache.hudi.QuickstartUtils.DataGenerator = org.apache.hudi.QuickstartUtils$DataGenerator@5cdd5ff9

 

## 3.3 插入数据

新增数据，生成一些数据，将其加载到DataFrame中，然后将DataFrame写入Hudi表

scala> val inserts = convertToStringList(dataGen.generateInserts(10))

scala> val df = spark.read.json(spark.sparkContext.parallelize(inserts, 2))

scala> df.write.format("hudi").

   |  options(getQuickstartWriteConfigs).

   |  option(PRECOMBINE_FIELD_OPT_KEY, "ts").

   |  option(RECORDKEY_FIELD_OPT_KEY, "uuid").

   |  option(PARTITIONPATH_FIELD_OPT_KEY, "partitionpath").

   |  option(TABLE_NAME, tableName).

   |  mode(Overwrite).

   |  save(basePath)

Mode(overwrite)将覆盖重新创建表（如果已存在）。可以检查/tmp/hudi_trps_cow 路径下是否有数据生成。

[root@hadoop102 ~]# cd /tmp/hudi_trips_cow/

[root@hadoop102 hudi_trips_cow]# ls

americas  asia

## 3.4 查询数据

scala> val tripsSnapshotDF = spark.

   |  read.

   |  format("hudi").

   |  load(basePath + "////")

scala> tripsSnapshotDF.createOrReplaceTempView("hudi_trips_snapshot")

scala> spark.sql("select fare, begin_lon, begin_lat, ts from  hudi_trips_snapshot where fare > 20.0").show()

+------------------+-------------------+-------------------+---+

|        fare|      begin_lon|      begin_lat| ts|

+------------------+-------------------+-------------------+---+

| 64.27696295884016| 0.4923479652912024| 0.5731835407930634|0.0|

| 33.92216483948643| 0.9694586417848392| 0.1856488085068272|0.0|

| 27.79478688582596| 0.6273212202489661|0.11488393157088261|0.0|

| 93.56018115236618|0.14285051259466197|0.21624150367601136|0.0|

|  43.4923811219014| 0.8779402295427752| 0.6100070562136587|0.0|

| 66.62084366450246|0.03844104444445928| 0.0750588760043035|0.0|

|34.158284716382845|0.46157858450465483| 0.4726905879569653|0.0|

| 41.06290929046368| 0.8192868687714224|  0.651058505660742|0.0|

+------------------+-------------------+-------------------+---+

scala> spark.sql("select _hoodie_commit_time, _hoodie_record_key, _hoodie_partition_path, rider, driver, fare from  hudi_trips_snapshot").show()

+-------------------+--------------------+----------------------+---------+----------+------------------+

|_hoodie_commit_time|  _hoodie_record_key|_hoodie_partition_path|   rider|   driver|        fare|

+-------------------+--------------------+----------------------+---------+----------+------------------+

|   20200701105144|6007a624-d942-4e0...|  americas/united_s...|rider-213|driver-213| 64.27696295884016|

|   20200701105144|db7c6361-3f05-48d...|  americas/united_s...|rider-213|driver-213| 33.92216483948643|

|   20200701105144|dfd0e7d9-f10c-468...|  americas/united_s...|rider-213|driver-213|19.179139106643607|

|   20200701105144|e36365c8-5b3a-415...|  americas/united_s...|rider-213|driver-213| 27.79478688582596|

|   20200701105144|fb92c00e-dea2-48e...|  americas/united_s...|rider-213|driver-213| 93.56018115236618|

|   20200701105144|98be3080-a058-47d...|  americas/brazil/s...|rider-213|driver-213|  43.4923811219014|

|   20200701105144|3dd6ef72-4196-469...|  americas/brazil/s...|rider-213|driver-213| 66.62084366450246|

|   20200701105144|20f9463f-1c14-4e6...|  americas/brazil/s...|rider-213|driver-213|34.158284716382845|

|   20200701105144|1585ad3a-11c9-43c...|   asia/india/chennai|rider-213|driver-213|17.851135255091155|

|   20200701105144|d40daa90-cf1a-4d1...|   asia/india/chennai|rider-213|driver-213| 41.06290929046368|

+-------------------+--------------------+----------------------+---------+----------+------------------+

由于测试数据分区是 区域/国家/城市，所以load(basePath  “////”)

## 3.5 修改数据

类似于插入新数据，使用数据生成器生成新数据对历史数据进行更新。将数据加载到DataFrame中并将DataFrame写入Hudi表中

scala> val updates = convertToStringList(dataGen.generateUpdates(10))

scala> val df = spark.read.json(spark.sparkContext.parallelize(updates, 2))

scala> df.write.format("hudi").

   |  options(getQuickstartWriteConfigs).

   |  option(PRECOMBINE_FIELD_OPT_KEY, "ts").

   |  option(RECORDKEY_FIELD_OPT_KEY, "uuid").

   |  option(PARTITIONPATH_FIELD_OPT_KEY, "partitionpath").

   |  option(TABLE_NAME, tableName).

   |  mode(Append).

   |  save(basePath)

## 3.6 增量查询

Hudi还提供了获取自给定提交时间戳以来以更改记录流的功能。这可以通过使用Hudi的增量查询并提供开始流进行更改的开始时间来实现。

scala> spark.

   |  read.

   |  format("hudi").

   |  load(basePath + "////").

   |  createOrReplaceTempView("hudi_trips_snapshot")

scala> val commits = spark.sql("select distinct(_hoodie_commit_time) as commitTime from  hudi_trips_snapshot order by commitTime").map(k => k.getString(0)).take(50)

scala> val beginTime = commits(commits.length - 2)

beginTime: String = 20200701105144

scala> val tripsIncrementalDF = spark.read.format("hudi").

   |  option(QUERY_TYPE_OPT_KEY, QUERY_TYPE_INCREMENTAL_OPT_VAL).

   |  option(BEGIN_INSTANTTIME_OPT_KEY, beginTime).

   |  load(basePath)

scala> tripsIncrementalDF.createOrReplaceTempView("hudi_trips_incremental")

scala> spark.sql("select `_hoodie_commit_time`, fare, begin_lon, begin_lat, ts from  hudi_trips_incremental where fare > 20.0").show()

+-------------------+------------------+--------------------+-------------------+---+

|_hoodie_commit_time|        fare|      begin_lon|      begin_lat| ts|

+-------------------+------------------+--------------------+-------------------+---+

|   20200701110546|49.527694252432056|  0.5142184937933181| 0.7340133901254792|0.0|

|   20200701110546|  90.9053809533154| 0.19949323322922063|0.18294079059016366|0.0|

|   20200701110546|  98.3428192817987|  0.3349917833248327| 0.4777395067707303|0.0|

|   20200701110546| 90.25710109008239|  0.4006983139989222|0.08528650347654165|0.0|

|   20200701110546| 63.72504913279929|  0.888493603696927| 0.6570857443423376|0.0|

|   20200701110546| 29.47661370147079|0.010872312870502165| 0.1593867607188556|0.0|

+-------------------+------------------+--------------------+-------------------+---+

这将提供在beginTime提交后的数据，并且fare>20的数据

## 3.7 时间点查询

根据特定时间查询，可以将endTime指向特定时间，beginTime指向000（表示最早提交时间）

 

scala> val beginTime = "000"

beginTime: String = 000

 

scala> val endTime = commits(commits.length - 2)

endTime: String = 20200701105144

scala> val tripsPointInTimeDF = spark.read.format("hudi").

   |  option(QUERY_TYPE_OPT_KEY, QUERY_TYPE_INCREMENTAL_OPT_VAL).

   |  option(BEGIN_INSTANTTIME_OPT_KEY, beginTime).

   |  option(END_INSTANTTIME_OPT_KEY, endTime).

   |  load(basePath)

scala> tripsPointInTimeDF.createOrReplaceTempView("hudi_trips_point_in_time")

scala> spark.sql("select `_hoodie_commit_time`, fare, begin_lon, begin_lat, ts from hudi_trips_point_in_time where fare > 20.0").show()

+-------------------+------------------+-------------------+-------------------+---+

|_hoodie_commit_time|        fare|      begin_lon|      begin_lat| ts|

+-------------------+------------------+-------------------+-------------------+---+

|   20200701105144| 64.27696295884016| 0.4923479652912024| 0.5731835407930634|0.0|

|   20200701105144| 33.92216483948643| 0.9694586417848392| 0.1856488085068272|0.0|

|   20200701105144| 27.79478688582596| 0.6273212202489661|0.11488393157088261|0.0|

|   20200701105144| 93.56018115236618|0.14285051259466197|0.21624150367601136|0.0|

|   20200701105144|  43.4923811219014| 0.8779402295427752| 0.6100070562136587|0.0|

|   20200701105144| 66.62084366450246|0.03844104444445928| 0.0750588760043035|0.0|

|   20200701105144|34.158284716382845|0.46157858450465483| 0.4726905879569653|0.0|

|   20200701105144| 41.06290929046368| 0.8192868687714224|  0.651058505660742|0.0|

+-------------------+------------------+-------------------+-------------------+---+

 

## 3.8 删除数据

scala> spark.sql("select uuid, partitionPath from hudi_trips_snapshot").count()

res12: Long = 10

scala> val ds = spark.sql("select uuid, partitionPath from hudi_trips_snapshot").limit(2)

scala> val deletes = dataGen.generateDeletes(ds.collectAsList())

scala> val df = spark.read.json(spark.sparkContext.parallelize(deletes, 2));

scala> df.write.format("hudi").

   |  options(getQuickstartWriteConfigs).

   |  option(OPERATION_OPT_KEY,"delete").

   |  option(PRECOMBINE_FIELD_OPT_KEY, "ts").

   |  option(RECORDKEY_FIELD_OPT_KEY, "uuid").

   |  option(PARTITIONPATH_FIELD_OPT_KEY, "partitionpath").

   |  option(TABLE_NAME, tableName).

   |  mode(Append).

   |  save(basePath)

scala> val roAfterDeleteViewDF = spark.

   |  read.

   |  format("hudi").

   |  load(basePath + "////")

scala> roAfterDeleteViewDF.registerTempTable("hudi_trips_snapshot")

scala> spark.sql("select uuid, partitionPath from hudi_trips_snapshot").count()

res15: Long = 8

只有append模式，才支持删除功能

 

# 第4章 基础用例

## 4.1近实时摄取

将外部数据（例如事件日志，数据库，外部源）如何摄取到Hadoop Data Lake是一个众所周知的问题。在大多数Hadoop部署中，经常会以零碎的方式，使用多种摄取工具解决，这些数据对整个组织是最具有价值的。

对于RDBMS关系型的摄入，Hudi提供了更快的Upset操作。例如，你可以通过MySql binlog的形式或者Sqoop导入到hdfs上的对应的Hudi表中，这样操作比Sqoop批量合并job(Sqoop merge)和复杂合并工作流更加快速高效。

对于NoSql的数据库，比如Cassandra,Voldemort,Hbase,这种可以存储数十亿行的数据库。采用完全批量加载是根本不可行的，并且如果摄取数据要跟上通常较高的更新量，则需要更有效的方法。

即使对于像Kafka这样不可变数据库源，Hudi也会在HDFS	上强制执行最小文件大小，从而通过整体解决Hadoop领域中小文件过多问题，改善NameNode的运行状况。对于事件流尤为重要，因为事件流通常较大（例如：点击流），并且如果管理不善，可能会严重损害Hadoop集群。

 在所有来源中，Hudi都增加了急需的功能，即通过提交概念将新数据原子推送给消费者，避免摄入数据失败。

## 4.2近实时分析

通常，实时数据集市由专门的分析存储（例如Druid或Memsql,OpenTSDB）提供支持，对于较低规模的数据，这绝对是完美的，可实现亚秒级响应查询。但是通常这些数据库最终会因为交互性较低的查询而被滥用，导致利用率不足。

另一方面，Hadoop上交互式的SQL解决方案有Presto和Spark sql。将数据的更新时间缩短至几分钟，Hudi可以提供多种高效的替代方案，并可以对存储在DFS中的多个大小表进行实时分析，此外Hudi没有外部依赖，例如专用实时分析的专用HBase集群，因此可以在不增加操作开销的情况下，进行更快更新鲜的分析。

## 4.3增量处理管道

  Hadoop提供的一项基本功能是，通过表示为工作流的DAG来构建彼此衍生的表链。工作流通常取决于多个上游工作流输出的新数据，并且传统上，新数据的可用性由新的DFS文件夹/配置单元分区指示。让我们举一个具体的例子来说明这一点。上游工作流U可以每小时创建一个Hive分区，并在每小时的末尾（processing_time）包含该小时（event_time）的数据，从而提供1小时的有效刷新。然后，下游工作流D在U完成后立即开始，并在接下来的一个小时内进行自己的处理，从而将有效延迟增加到2个小时。

 

上述范例只是忽略了延迟到达的数据，即，processing_time和event_time分开时。不幸的是，在当今的后移动和物联网前世界中，间歇性连接的移动设备和传感器的最新数据是正常现象，而不是反常现象。在这种情况下，保证正确性的唯一补救方法是每小时一次又一次地重新处理最后几个小时的数据，这会严重损害整个生态系统的效率。例如想象在数百个工作流程中每小时重新处理价值TB的数据。

 

Hudi通过提供一种以记录粒度（而不是文件夹/分区）消耗上游Hudi表HU的新数据（包括最新数据），应用处理逻辑并有效地更新/协调最新数据的方式再次进行救援。下游护桌HD。在这里，HU和HD可以以更频繁的时间表（例如15分钟）连续进行调度，并在HD上提供30分钟的终端延迟。

 

为了实现这一目标，Hudi从流处理框架（如Spark Streaming），发布/订阅系统（如Kafka）或数据库复制技术（如Oracle XStream）中采用了类似的概念

 

## 4.4 DFS的数据分散

Hudi可以像Kafka一样，用于数据分散，将每个管道的数据输出到Hudi表中，然后将其递增尾部以获取新数据并写入服务存储中。

 

# 第5章 与其他数据库对比

## 5.1 与Kudu对比

  Apache Kudu是一个存储系统，其目标与Hudi类似，后者的目标是通过对Upserts的一流支持，对PB级数据进行实时分析。一个关键的区别是Kudu还试图充当OLTP工作负载的数据存储，而Hudi并不希望这样做。因此，Kudu不支持增量拉取（截至2017年初），Hudi这样做是为了启用增量处理用例。

 

Kudu与分布式文件系统抽象和HDFS完全不同，它自己的一组存储服务器通过RAFT相互通信。另一方面，Hudi旨在与底层Hadoop兼容文件系统（HDFS，S3或Ceph）一起使用，并且没有自己的存储服务器，而是依靠Apache Spark来完成繁重的工作。因此，就像其他Spark作业一样，Hudi可以轻松扩展，而Kudu则需要硬件和操作支持，这对于HBase或Vertica这样的数据存储来说是典型的。

## 5.2 与HBase对比

   HBase是OLTP工作负载的关键值存储，但鉴于与Hadoop的相似性，用户通常倾向于将HBase与分析相关联。 鉴于HBase经过严格的写优化，它支持开箱即用的亚秒级更新，Hive-on-HBase允许用户查询该数据。 但是，就分析工作负载的实际性能而言，混合列式存储格式（例如Parquet / ORC）可以轻松击败HBase，因为这些工作负载主要是读取繁重的工作。 Hudi弥补了更快的数据与具有分析性存储格式之间的差距。 从操作的角度来看，与仅管理分析用的大型HBase区域服务器场相比，为用户提供可提供更快数据的库更具可扩展性。 最终，HBase不像Hudi这样的一流公民来支持诸如提交时间，增量拉动之类的增量处理原语。

  Hudi拥有更好数据分析能力。

## 5.3与Streaming流式处理对比

  Hudi可以与当今的批处理（写时复制表）和流式（读时合并表）作业集成，以将计算结果存储在Hadoop中。对于Spark应用程序，这可以通过将Hudi库与Spark / Spark流式DAG直接集成来实现。在非Spark处理系统（例如Flink，Hive）的情况下，可以在各个系统中进行处理，然后通过Kafka主题/ DFS中间文件将其发送到Hudi表中。从概念上讲，数据处理管道仅由三个部分组成：源，处理，接收器，用户最终针对接收器运行查询以使用管道的结果。 Hudi可以充当将数据存储在DFS上的源或接收器。 Hudi在给定流处理管道上的适用性最终归结为Presto / SparkSQL / Hive对您的查询的适用性。

 

# 第6章 Hudi中的名称概念

  Apache Hudi通过Hadoop兼容存储提供以下流处理

（1）Update/Delete Records，更改表记录

（2）Change Streams,获取已更改记录

## 6.1 Timeline

  Hudi的核心是维护不同时间对表执行的所有操作的事件表，这有助于提供表的即时视图，同时还有效地支持按到达顺序进行数据检索。Hudi包含以下组件：

（1）Instant action:在表上的操作类型

（2）Instant time：操作开始的一个时间戳，该时间戳会按照开始时间顺序单调递增

（3）state:即时状态

 

Hudi保证在时间轴上执行的操作都是原先性的，所有执行的操作包括：

（1）commits:原子的写入一张表的操作

（2）cleans:后台消除了表中的旧版本数据，即表中不在需要的数据

（3）delta_commit:增量提交，将一批数据原子写入到MergeOnRead表中，并且只记录到增量日志中

（4）compaction:后台协调Hudi中的差异数据

（5）rollback:回滚，删除在写入过程中的数据

（6）savepoint:将某些文件标记“已保存”，以便清理数据时不会删除它们，一般用于表的还原，可以将数据还原到某个时间点

 

任何操作都可以处于以下状态

（1）Requested:表示已安排操作行为，但是尚未开始

（2）Inflight:表示正在执行当前操作

（3）Completed:表示已完成操作

## 6.2 File management

  Hudi将表组织成DFS上基本路径下的目录结构。表分位几个分区，与hive类似，每个分区均有唯一标示。

  在每个分区内，有多个数据组，每个数据组包含几个文件片，其中文件片包含基本文件和日志文件。Hudi采用MVCC设计，其中压缩操作将日志文件和基本数据文件合并成新的文件片，而清楚操作则将未使用的文件片去除。

## 6.3索引

  Hudi通过使用索引机制，生成hoodie密钥映射对应文件ID，从而提供高效upsert操作。

 

## 6.4表类型

（1）Copy on Write:仅使用列式存储，例如parquet。仅更新版本号，通过写入过程中执行同步合并来重写文件

（2）Merge on Read:基于列式存储（parquet）和行式存储(arvo)结合的文件更始进行存储。更新记录到增量文件，压缩同步和异步生成新版本的文件

 

以下是对比

![img](file:////tmp/wps-gree/ksohtml/wpsX1XwkQ.jpg) 

## 6.5查询类型

（1）快照查询（\Snapshot Queries\）：查询操作将查询最新快照的表数据。如果是Merge on Read类型的表，它将动态合并最新文件版本的基本数据和增量数据用于显示查询。如果是Copy On Write类型的表，它直接查询parquet表，同时提供upsert/delete操作.

（2）增量查询（\Incremental Queries\）：查询只能看到写入表的新数据。这有效的提供了change streams来启用增量数据管道。

（3）优化读查询（\Read Optimized Queries\）：查询将查看给定提交/压缩操作表的最新快照

 

 

以下是对比

![img](file:////tmp/wps-gree/ksohtml/wpsIqTrgM.jpg) 

# 第7章 Hudi代码测试

## 7.1 Hudi操作

### 7.1.1 pom.xml

<dependencies>    <dependency>        <groupId>org.apache.hudi</groupId>        <artifactId>hudi-client</artifactId>        <version>0.5.3</version>    </dependency>    <dependency>        <groupId>org.apache.hudi</groupId>        <artifactId>hudi-hive</artifactId>        <version>0.5.3</version>    </dependency>    <dependency>        <groupId>org.apache.hudi</groupId>        <artifactId>hudi-spark-bundle_2.11</artifactId>        <version>0.5.3</version>    </dependency>    <dependency>        <groupId>org.apache.hudi</groupId>        <artifactId>hudi-common</artifactId>        <version>0.5.3</version>    </dependency>    <dependency>        <groupId>org.apache.hudi</groupId>        <artifactId>hudi-hadoop-mr-bundle</artifactId>        <version>0.5.3</version>    </dependency>    <!-- Spark的依赖引入 -->    <dependency>        <groupId>org.apache.spark</groupId>        <artifactId>spark-core_2.11</artifactId>        <!--            <scope>provided</scope>-->        <version>2.4.5</version>    </dependency>    <dependency>        <groupId>org.apache.spark</groupId>        <artifactId>spark-sql_2.11</artifactId>        <!--            <scope>provided</scope>-->        <version>2.4.5</version>    </dependency>    <dependency>        <groupId>org.apache.spark</groupId>        <artifactId>spark-hive_2.11</artifactId>        <!--            <scope>provided</scope>-->        <version>2.4.5</version>    </dependency>    <dependency>        <groupId>org.apache.spark</groupId>        <artifactId>spark-avro_2.11</artifactId>        <version>2.4.5</version>    </dependency>    <dependency>        <groupId>org.scala-lang</groupId>        <artifactId>scala-library</artifactId>        <!--            <scope>provided</scope>-->        <version>${scala.version}</version>    </dependency>    <dependency>        <groupId>org.apache.hadoop</groupId>        <artifactId>hadoop-client</artifactId>        <version>2.7.2</version>    </dependency>    <dependency>        <groupId>com.alibaba</groupId>        <artifactId>fastjson</artifactId>        <version>1.2.47</version>    </dependency>    <dependency>        <groupId>org.apache.spark</groupId>        <artifactId>spark-hive_2.11</artifactId>        <!--            <scope>provided</scope>-->        <version>2.4.5</version>    </dependency>    <dependency>        <groupId>org.spark-project.hive</groupId>        <artifactId>hive-jdbc</artifactId>        <version>1.2.1.spark2</version>    </dependency></dependencies>

### 7.1.2 case class

package com.atguigu.bean

case class DwsMember(
           uid: Int,
           ad_id: Int,
           var fullname: String,
           iconurl: String,
           dt: String,
           dn: String
          )

### 7.1.3配置文件

将集群配置文件复制到，项目resource源码包下,使本地环境可以访问hadoop集群

![img](file:////tmp/wps-gree/ksohtml/wpskwBYhO.jpg) 

### 7.1.4 Hudi写入Hdfs

package com.atguigu.hudi.test

import com.atguigu.bean.DwsMember
import com.atguigu.hudi.util.ParseJsonData
import org.apache.spark.SparkConf
import org.apache.spark.sql.{SaveMode, SparkSession}

object HoodieDataSourceExample {
 def main(args: Array[String]): Unit = {
  System.setProperty("HADOOP_USER_NAME", "root")
  val sparkConf = new SparkConf().setAppName("dwd_member_import")
   .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
   .setMaster("local[]")
  val sparkSession = SparkSession.builder().config(sparkConf).enableHiveSupport().getOrCreate()
  val ssc = sparkSession.sparkContext
  ssc.hadoopConfiguration.set("fs.defaultFS", "hdfs://mycluster")
  ssc.hadoopConfiguration.set("dfs.nameservices", "mycluster")
//   insertData(sparkSession)
    queryData(sparkSession)
 }

 /
  \ 读取hdfs日志文件通过hudi写入hdfs
  
  \ @param sparkSession
  /
 def insertData(sparkSession: SparkSession) = {
  import org.apache.spark.sql.functions._
  import sparkSession.implicits._
  val commitTime = System.currentTimeMillis().toString //生成提交时间
  val df = sparkSession.read.text("/user/atguigu/ods/member.log")
   .mapPartitions(partitions => {
    partitions.map(item => {
     val jsonObject = ParseJsonData.getJsonData(item.getString(0))
     DwsMember(jsonObject.getIntValue("uid"), jsonObject.getIntValue("ad_id")
      , jsonObject.getString("fullname"), jsonObject.getString("iconurl")
      , jsonObject.getString("dt"), jsonObject.getString("dn"))
    })
   })
  val result = df.withColumn("ts", lit(commitTime)) //添加ts 时间戳列
   .withColumn("uuid", col("uid")) //添加uuid 列 如果数据中uuid相同hudi会进行去重
   .withColumn("hudipartition", concat_ws("/", col("dt"), col("dn"))) //增加hudi分区列
  result.write.format("org.apache.hudi")
   //    .options(org.apache.hudi.QuickstartUtils.getQuickstartWriteConfigs)
   .option("hoodie.insert.shuffle.parallelism", 12)
   .option("hoodie.upsert.shuffle.parallelism", 12)
   .option("PRECOMBINE_FIELD_OPT_KEY", "ts") //指定提交时间列
   .option("RECORDKEY_FIELD_OPT_KEY", "uuid") //指定uuid唯一标示列
   .option("hoodie.table.name", "testTable")
   //    .option(DataSourceWriteOptions.DEFAULT_PARTITIONPATH_FIELD_OPT_VAL, "dt") //  发现api方式不起作用 分区列
   .option("hoodie.datasource.write.partitionpath.field", "hudipartition") //分区列
   .mode(SaveMode.Overwrite)
   .save("/user/atguigu/hudi")
 }

 

测试发现，Hudi不能指定多分区列，所以代码上分区列采用两列拼接成一列的方式，提交操作时必须指定ts和uuid。写入成功后查看hadoop路径上的文件

 

![img](file:////tmp/wps-gree/ksohtml/wpsVizjNN.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpsMV4voQ.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpsiTtiyP.jpg) 

### 7.1.5查询Hdfs上Hudi数据


/
 \ 查询hdfs上的hudi数据
 
 \ @param sparkSession
 /
def queryData(sparkSession: SparkSession) = {
 val df = sparkSession.read.format("org.apache.hudi")
  .load("/user/atguigu/hudi//")
 df.show()
}

显示结果为

![img](file:////tmp/wps-gree/ksohtml/wpsFFtA1M.jpg) 

 

### 7.1.6修改Hdfs上Hudi数据

def updateData(sparkSession: SparkSession) = {
 import org.apache.spark.sql.functions._
 import sparkSession.implicits._
 val commitTime = System.currentTimeMillis().toString //生成提交时间
 val df = sparkSession.read.text("/user/atguigu/ods/member.log")
  .mapPartitions(partitions => {
   partitions.map(item => {
    val jsonObject = ParseJsonData.getJsonData(item.getString(0))
    DwsMember(jsonObject.getIntValue("uid"), jsonObject.getIntValue("ad_id")
     , jsonObject.getString("fullname"), jsonObject.getString("iconurl")
     , jsonObject.getString("dt"), jsonObject.getString("dn"))
   })
  })
 val result = df.withColumn("ts", lit(commitTime)) //添加ts 时间戳列
  .withColumn("uuid", col("uid")) //添加uuid 列 如果数据中uuid相同hudi会进行去重
  .withColumn("hudipartition", concat_ws("/", col("dt"), col("dn"))) //增加hudi分区列
 result.write.format("org.apache.hudi")
  //    .options(org.apache.hudi.QuickstartUtils.getQuickstartWriteConfigs)
  .option("hoodie.insert.shuffle.parallelism", 12)
  .option("hoodie.upsert.shuffle.parallelism", 12)
  .option("PRECOMBINE_FIELD_OPT_KEY", "ts") //指定提交时间列
  .option("RECORDKEY_FIELD_OPT_KEY", "uuid") //指定uuid唯一标示列
  .option("hoodie.table.name", "testTable")
  //    .option(DataSourceWriteOptions.DEFAULT_PARTITIONPATH_FIELD_OPT_VAL, "dt") //  发现api方式不起作用 分区列
  .option("hoodie.datasource.write.partitionpath.field", "hudipartition") //分区列
  .mode(SaveMode.Append)
  .save("/user/atguigu/hudi")
}

虽然代码操作和新增一样只是修改了插入模式为append,但是hudi会根据uid判断进行更新数据，操作完毕后，生成一份最新的修改后的数据。同时hdfs路径上写入一份数据。

![img](file:////tmp/wps-gree/ksohtml/wpsxgHhqP.jpg) 

提交时间发生了变化

![img](file:////tmp/wps-gree/ksohtml/wpsZ5kueO.jpg) 

数据条数为94175

![img](file:////tmp/wps-gree/ksohtml/wpsqY13pM.jpg) 

### 7.1.7增量查询

def incrementalQuery(sparkSession: SparkSession) = {
 val beginTime = 20200703130000l
 val df = sparkSession.read.format("org.apache.hudi")
  .option(DataSourceReadOptions.QUERY_TYPE_OPT_KEY, DataSourceReadOptions.QUERY_TYPE_INCREMENTAL_OPT_VAL) //指定模式为增量查询
  .option(DataSourceReadOptions.BEGIN_INSTANTTIME_OPT_KEY, beginTime) //设置开始查询的时间戳  不需要设置结束时间戳
  .load("/user/atguigu/hudi")
 df.show()
 println(df.count())
}

![img](file:////tmp/wps-gree/ksohtml/wpspy4QkP.jpg) 

根据haoodie_commit_time，时间进行查询，查询增量修改数据，注意参数begintime是和hadoop_commit_time对比而不是跟ts对比。如果beginitime填了比haoodie_commit_time大则会过滤所有数据。

 

![img](file:////tmp/wps-gree/ksohtml/wpsubRBdO.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpsXsDmiM.jpg) 

### 7.1.8指定特定时间查询

def updateData(sparkSession: SparkSession) = {
 import org.apache.spark.sql.functions._
 import sparkSession.implicits._
 val commitTime = System.currentTimeMillis().toString //生成提交时间
 val df = sparkSession.read.text("/user/atguigu/ods/member.log")
  .mapPartitions(partitions => {
   partitions.map(item => {
    val jsonObject = ParseJsonData.getJsonData(item.getString(0))
    DwsMember(jsonObject.getIntValue("uid"), jsonObject.getIntValue("ad_id")
     , jsonObject.getString("fullname"), jsonObject.getString("iconurl")
     , jsonObject.getString("dt"), jsonObject.getString("dn"))
   })
  }).limit(100)
 val result = df.withColumn("ts", lit(commitTime)) //添加ts 时间戳列
  .withColumn("uuid", col("uid")) //添加uuid 列 如果数据中uuid相同hudi会进行去重
  .withColumn("hudipartition", concat_ws("/", col("dt"), col("dn"))) //增加hudi分区列
 result.write.format("org.apache.hudi")
  //    .options(org.apache.hudi.QuickstartUtils.getQuickstartWriteConfigs)
  .option("hoodie.insert.shuffle.parallelism", 12)
  .option("hoodie.upsert.shuffle.parallelism", 12)
  .option("PRECOMBINE_FIELD_OPT_KEY", "ts") //指定提交时间列
  .option("RECORDKEY_FIELD_OPT_KEY", "uuid") //指定uuid唯一标示列
  .option("hoodie.table.name", "testTable")
  //    .option(DataSourceWriteOptions.DEFAULT_PARTITIONPATH_FIELD_OPT_VAL, "dt") //  发现api方式不起作用 分区列
  .option("hoodie.datasource.write.partitionpath.field", "hudipartition") //分区列
  .mode(SaveMode.Append)
  .save("/user/atguigu/hudi")
}

 

def pointInTimeQuery(sparkSession: SparkSession) = {
 val beginTime = 20200703150000l
 val endTime = 20200703160000l
 val df = sparkSession.read.format("org.apache.hudi")
  .option(DataSourceReadOptions.QUERY_TYPE_OPT_KEY, DataSourceReadOptions.QUERY_TYPE_INCREMENTAL_OPT_VAL) //指定模式为增量查询
  .option(DataSourceReadOptions.BEGIN_INSTANTTIME_OPT_KEY, beginTime) //设置开始查询的时间戳
  .option(DataSourceReadOptions.END_INSTANTTIME_OPT_KEY, endTime)
  .load("/user/atguigu/hudi")
 df.show()
 println(df.count())

}

演示，update时limit只修改100条数据，然后根据时间戳进行查询，只会查询出进行修改的100条符合时间的数据.

 

![img](file:////tmp/wps-gree/ksohtml/wpsJFTyiP.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpsFIBm5P.jpg) 

## 7.2 Hudi集成hive

### 7.2.1将编译好的hudi-adoop-mr的包复制到hive lib下

[root@hadoop102~]#cp ./packaging/hudi-hadoop-mr-bundle/hudi-hadoop-mr-bundle-0.6.0-SNAPSHOT.jar /opt/module/apache-hive-3.1.2-bin/lib

### 7.2.2建表

hudi会自动创建表，也可以提前手动建表

CREATE EXTERNAL TABLE `member_rt`(

 `_hoodie_commit_time` string, 

 `_hoodie_commit_seqno` string, 

 `_hoodie_record_key` string, 

 `_hoodie_partition_path` string, 

 `_hoodie_file_name` string, 

 `uid` int, 

 `ad_id` int, 

 `fullname` string, 

 `iconurl` string, 

 `ts` string, 

 `hudipartition` string)

PARTITIONED BY ( 

 `dt` string, 

 `dn` string)

ROW FORMAT SERDE 

 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 

STORED AS INPUTFORMAT 

 'org.apache.hudi.hadoop.realtime.HoodieParquetRealtimeInputFormat' 

OUTPUTFORMAT 

 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'

LOCATION

 'hdfs://mycluster/user/atguigu/hudi/hivetest'

### 7.2.3测试类

package com.atguigu.hudi.test

import com.atguigu.bean.DwsMember
import com.atguigu.hudi.util.ParseJsonData
import org.apache.hudi.DataSourceWriteOptions
import org.apache.hudi.config.HoodieIndexConfig
import org.apache.hudi.hive.MultiPartKeysValueExtractor
import org.apache.hudi.index.HoodieIndex
import org.apache.spark.SparkConf
import org.apache.spark.sql.{SaveMode, SparkSession}

object HudiTestHive {
 def main(args: Array[String]): Unit = {
  System.setProperty("HADOOP_USER_NAME", "root")
  val sparkConf = new SparkConf().setAppName("dwd_member_import")
   .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
   .setMaster("local[]")
  val sparkSession = SparkSession.builder().config(sparkConf).getOrCreate()
  val ssc = sparkSession.sparkContext
  ssc.hadoopConfiguration.set("fs.defaultFS", "hdfs://mycluster")
  ssc.hadoopConfiguration.set("dfs.nameservices", "mycluster")
  import org.apache.spark.sql.functions._
  import sparkSession.implicits._
  val commitTime = System.currentTimeMillis().toString //生成提交时间
  val df = sparkSession.read.text("/user/atguigu/ods/member.log")
   .mapPartitions(partitions => {
    partitions.map(item => {
     val jsonObject = ParseJsonData.getJsonData(item.getString(0))
     DwsMember(jsonObject.getIntValue("uid"), jsonObject.getIntValue("ad_id")
      , jsonObject.getString("fullname"), jsonObject.getString("iconurl")
      , jsonObject.getString("dt"), jsonObject.getString("dn"))
    })
   }).withColumn("ts", lit(commitTime))
   .withColumn("hudipartition", concat_ws("/", col("dt"), col("dn")))
  Class.forName("org.apache.hive.jdbc.HiveDriver");
  df.write.format("org.apache.hudi")
   .option(DataSourceWriteOptions.TABLE_TYPE_OPT_KEY, DataSourceWriteOptions.MOR_TABLE_TYPE_OPT_VAL) //选择表的类型 到底是MERGE_ON_READ 还是 COPY_ON_WRITE
   .option(DataSourceWriteOptions.RECORDKEY_FIELD_OPT_KEY, "uid") //设置主键
   .option(DataSourceWriteOptions.PRECOMBINE_FIELD_OPT_KEY, "ts") //数据更新时间戳的
   .option(DataSourceWriteOptions.PARTITIONPATH_FIELD_OPT_KEY, "hudipartition") //hudi分区列
   .option("hoodie.table.name", "member") //hudi表名
   .option(DataSourceWriteOptions.HIVE_URL_OPT_KEY, "jdbc:hive2://hadoop101:10000") //hiveserver2地址
   .option(DataSourceWriteOptions.HIVE_DATABASE_OPT_KEY, "default") //设置hudi与hive同步的数据库
   .option(DataSourceWriteOptions.HIVE_TABLE_OPT_KEY, "member") //设置hudi与hive同步的表名
   .option(DataSourceWriteOptions.HIVE_PARTITION_FIELDS_OPT_KEY, "dt,dn") //hive表同步的分区列
   .option(DataSourceWriteOptions.HIVE_PARTITION_EXTRACTOR_CLASS_OPT_KEY, classOf[MultiPartKeysValueExtractor].getName) // 分区提取器 按/ 提取分区
   .option(DataSourceWriteOptions.HIVE_SYNC_ENABLED_OPT_KEY, "true") //设置数据集注册并同步到hive
   .option(HoodieIndexConfig.BLOOM_INDEX_UPDATE_PARTITION_PATH, "true") //设置当分区变更时，当前数据的分区目录是否变更
   .option(HoodieIndexConfig.INDEX_TYPE_PROP, HoodieIndex.IndexType.GLOBAL_BLOOM.name()) //设置索引类型目前有HBASE,INMEMORY,BLOOM,GLOBAL_BLOOM 四种索引 为了保证分区变更后能找到必须设置全局GLOBAL_BLOOM
   .option("hoodie.insert.shuffle.parallelism", "12")
   .option("hoodie.upsert.shuffle.parallelism", "12")
   .mode(SaveMode.Append)
   .save("/user/atguigu/hudi/hivetest")

 }
}

 

跑完任务之后，去查询hive表,通过hive-shell查询

![img](file:////tmp/wps-gree/ksohtml/wpscKCWCP.jpg) 

 

使用spark-shell，spark on hive方式查询

![img](file:////tmp/wps-gree/ksohtml/wpsLzsFvO.jpg) 

Hudi集成hive，即Hudi同步数据到hive，供hive查询

## 7.3表类型对比

### 7.3.1生成数据

（1）针对copy_on_write表和merge_on_read表同时生成两份表数据，读取member日志数据各自生成两种类型的表

 def main(args: Array[String]): Unit = {
 System.setProperty("HADOOP_USER_NAME", "root")
 val sparkConf = new SparkConf().setAppName("dwd_member_import")
  .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
  .setMaster("local[]")
 val sparkSession = SparkSession.builder().config(sparkConf).enableHiveSupport().getOrCreate()
 val ssc = sparkSession.sparkContext
 ssc.hadoopConfiguration.set("fs.defaultFS", "hdfs://mycluster")
 ssc.hadoopConfiguration.set("dfs.nameservices", "mycluster")
 generateData(sparkSession)
 //   queryData(sparkSession)
 //   updateData(sparkSession)
 //     queryData(sparkSession)
}


def generateData(sparkSession: SparkSession) = {
 import org.apache.spark.sql.functions._
 import sparkSession.implicits._
 val commitTime = System.currentTimeMillis().toString //生成提交时间
 val df = sparkSession.read.text("/user/atguigu/ods/member.log")
  .mapPartitions(partitions => {
   partitions.map(item => {
    val jsonObject = ParseJsonData.getJsonData(item.getString(0))
    DwsMember(jsonObject.getIntValue("uid"), jsonObject.getIntValue("ad_id")
     , jsonObject.getString("fullname"), jsonObject.getString("iconurl")
     , jsonObject.getString("dt"), jsonObject.getString("dn"))
   })
  }).withColumn("ts", lit(commitTime))
  .withColumn("hudipartition", concat_ws("/", col("dt"), col("dn")))
 df.write.format("org.apache.hudi")
  .option(DataSourceWriteOptions.TABLE_TYPE_OPT_KEY, DataSourceWriteOptions.MOR_TABLE_TYPE_OPT_VAL) //指定表类型为MERGE_ON_READ
  .option("hoodie.insert.shuffle.parallelism", 12)
  .option("hoodie.upsert.shuffle.parallelism", 12)
  .option(DataSourceWriteOptions.RECORDKEY_FIELD_OPT_KEY, "uid") //指定记录键
  .option(DataSourceWriteOptions.PRECOMBINE_FIELD_OPT_KEY, "ts") //数据更新时间戳的
  .option(DataSourceWriteOptions.PARTITIONPATH_FIELD_OPT_KEY, "hudipartition") //hudi分区列
  .option("hoodie.table.name", "hudimembertest1") //hudi表名
  .option(DataSourceWriteOptions.HIVE_URL_OPT_KEY, "jdbc:hive2://hadoop101:10000") //hiveserver2地址
  .option(DataSourceWriteOptions.HIVE_DATABASE_OPT_KEY, "default") //设置hudi与hive同步的数据库
  .option(DataSourceWriteOptions.HIVE_TABLE_OPT_KEY, "member1") //设置hudi与hive同步的表名
  .option(DataSourceWriteOptions.HIVE_PARTITION_FIELDS_OPT_KEY, "dt,dn") //hive表同步的分区列
  .option(DataSourceWriteOptions.HIVE_PARTITION_EXTRACTOR_CLASS_OPT_KEY, classOf[MultiPartKeysValueExtractor].getName) // 分区提取器 按/ 提取分区
  .option(DataSourceWriteOptions.HIVE_SYNC_ENABLED_OPT_KEY, "true") //设置数据集注册并同步到hive
  .option(HoodieIndexConfig.BLOOM_INDEX_UPDATE_PARTITION_PATH, "true") //设置当分区变更时，当前数据的分区目录是否变更
  .option(HoodieIndexConfig.INDEX_TYPE_PROP, HoodieIndex.IndexType.GLOBAL_BLOOM.name()) //设置索引类型目前有HBASE,INMEMORY,BLOOM,GLOBAL_BLOOM 四种索引 为了保证分区变更后能找到必须设置全局GLOBAL_BLOOM
  .option("hoodie.insert.shuffle.parallelism", "12")
  .option("hoodie.upsert.shuffle.parallelism", "12")
  .mode(SaveMode.Overwrite)
  .save("/user/atguigu/hudi/hudimembertest1")

 df.write.format("org.apache.hudi")
  .option(DataSourceWriteOptions.TABLE_TYPE_OPT_KEY, DataSourceWriteOptions.COW_TABLE_TYPE_OPT_VAL) //指定表类型为COPY_ON_WRITE
  .option("hoodie.insert.shuffle.parallelism", 12)
  .option("hoodie.upsert.shuffle.parallelism", 12)
  .option(DataSourceWriteOptions.RECORDKEY_FIELD_OPT_KEY, "uid") //指定记录键
  .option(DataSourceWriteOptions.PRECOMBINE_FIELD_OPT_KEY, "ts") //数据更新时间戳的
  .option(DataSourceWriteOptions.PARTITIONPATH_FIELD_OPT_KEY, "hudipartition") //hudi分区列
  .option("hoodie.table.name", "hudimembertest2") //hudi表名
  .option(DataSourceWriteOptions.HIVE_URL_OPT_KEY, "jdbc:hive2://hadoop101:10000") //hiveserver2地址
  .option(DataSourceWriteOptions.HIVE_DATABASE_OPT_KEY, "default") //设置hudi与hive同步的数据库
  .option(DataSourceWriteOptions.HIVE_TABLE_OPT_KEY, "member2") //设置hudi与hive同步的表名
  .option(DataSourceWriteOptions.HIVE_PARTITION_FIELDS_OPT_KEY, "dt,dn") //hive表同步的分区列
  .option(DataSourceWriteOptions.HIVE_PARTITION_EXTRACTOR_CLASS_OPT_KEY, classOf[MultiPartKeysValueExtractor].getName) // 分区提取器 按/ 提取分区
  .option(DataSourceWriteOptions.HIVE_SYNC_ENABLED_OPT_KEY, "true") //设置数据集注册并同步到hive
  .option(HoodieIndexConfig.BLOOM_INDEX_UPDATE_PARTITION_PATH, "true") //设置当分区变更时，当前数据的分区目录是否变更
  .option(HoodieIndexConfig.INDEX_TYPE_PROP, HoodieIndex.IndexType.GLOBAL_BLOOM.name()) //设置索引类型目前有HBASE,INMEMORY,BLOOM,GLOBAL_BLOOM 四种索引 为了保证分区变更后能找到必须设置全局GLOBAL_BLOOM
  .option("hoodie.insert.shuffle.parallelism", "12")
  .option("hoodie.upsert.shuffle.parallelism", "12")
  .mode(SaveMode.Overwrite)
  .save("/user/atguigu/hudi/hudimembertest2")
}

生成数据后，查看对应数据块大小，没有太大差别

![img](file:////tmp/wps-gree/ksohtml/wpsyWIlcQ.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpsN4rJdO.jpg) 

查看hive,发现自动建好3张表，继续使用命令查看表结构

![img](file:////tmp/wps-gree/ksohtml/wpsw6ZhfP.jpg) 

hive (default)> show create table member1_ro;

OK

createtab_stmt

CREATE EXTERNAL TABLE `member1_ro`(

 `_hoodie_commit_time` string, 

 `_hoodie_commit_seqno` string, 

 `_hoodie_record_key` string, 

 `_hoodie_partition_path` string, 

 `_hoodie_file_name` string, 

 `uid` int, 

 `ad_id` int, 

 `fullname` string, 

 `iconurl` string, 

 `ts` string, 

 `hudipartition` string)

PARTITIONED BY ( 

 `dt` string, 

 `dn` string)

ROW FORMAT SERDE 

 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 

STORED AS INPUTFORMAT 

 'org.apache.hudi.hadoop.HoodieParquetInputFormat' 

OUTPUTFORMAT 

 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'

LOCATION

 'hdfs://mycluster/user/atguigu/hudi/hudimembertest1'

TBLPROPERTIES (

 'bucketing_version'='2', 

 'last_commit_time_sync'='20200706143356', 

 'transient_lastDdlTime'='1594017284')

Time taken: 0.097 seconds, Fetched: 27 row(s)

 

hive (default)> show create table member1_rt;

OK

createtab_stmt

CREATE EXTERNAL TABLE `member1_rt`(

 `_hoodie_commit_time` string, 

 `_hoodie_commit_seqno` string, 

 `_hoodie_record_key` string, 

 `_hoodie_partition_path` string, 

 `_hoodie_file_name` string, 

 `uid` int, 

 `ad_id` int, 

 `fullname` string, 

 `iconurl` string, 

 `ts` string, 

 `hudipartition` string)

PARTITIONED BY ( 

 `dt` string, 

 `dn` string)

ROW FORMAT SERDE 

 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 

STORED AS INPUTFORMAT 

 'org.apache.hudi.hadoop.realtime.HoodieParquetRealtimeInputFormat' 

OUTPUTFORMAT 

 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'

LOCATION

 'hdfs://mycluster/user/atguigu/hudi/hudimembertest1'

TBLPROPERTIES (

 'bucketing_version'='2', 

 'last_commit_time_sync'='20200706143356', 

 'transient_lastDdlTime'='1594017286')

Time taken: 0.048 seconds, Fetched: 27 row(s)

 

hive (default)> show create table member2;

OK

createtab_stmt

CREATE EXTERNAL TABLE `member2`(

 `_hoodie_commit_time` string, 

 `_hoodie_commit_seqno` string, 

 `_hoodie_record_key` string, 

 `_hoodie_partition_path` string, 

 `_hoodie_file_name` string, 

 `uid` int, 

 `ad_id` int, 

 `fullname` string, 

 `iconurl` string, 

 `ts` string, 

 `hudipartition` string)

PARTITIONED BY ( 

 `dt` string, 

 `dn` string)

ROW FORMAT SERDE 

 'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 

STORED AS INPUTFORMAT 

 'org.apache.hudi.hadoop.HoodieParquetInputFormat' 

OUTPUTFORMAT 

 'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'

LOCATION

 'hdfs://mycluster/user/atguigu/hudi/hudimembertest2'

TBLPROPERTIES (

 'bucketing_version'='2', 

 'last_commit_time_sync'='20200706143445', 

 'transient_lastDdlTime'='1594017317')

可以看到格式有两种HoodieParquetInputFormat和HoodieParquetRealtimeInputFormat，两种在hudi中是读优化视图和实时视图

![img](file:////tmp/wps-gree/ksohtml/wpshw4SsP.jpg) 

Merge_ON_READ建表时会自动创建两种视图，COPY_ON_WRITE建表时只会自动创建读优化视图

  查询表，都已同步数据

![img](file:////tmp/wps-gree/ksohtml/wpsMxchGM.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpsWbTPMO.jpg) 

 

### 7.3.2修改数据

（2）有了数据之后对表进行更新操作，对两张表分别只更新uid 0-9的10条数据

def updateData(sparkSession: SparkSession) = {
 import org.apache.spark.sql.functions._
 import sparkSession.implicits._
 val commitTime = System.currentTimeMillis().toString //生成提交时间
 val df = sparkSession.read.text("/user/atguigu/ods/member.log")
  .mapPartitions(partitions => {
   partitions.map(item => {
    val jsonObject = ParseJsonData.getJsonData(item.getString(0))
    DwsMember(jsonObject.getIntValue("uid"), jsonObject.getIntValue("ad_id")
     , jsonObject.getString("fullname"), jsonObject.getString("iconurl")
     , jsonObject.getString("dt"), jsonObject.getString("dn"))
   })
  }).where("uid>=0 and uid<=9")

 val result = df.map(item => {
  item.fullname = "testName"
  item
 }).withColumn("ts", lit(commitTime))
  .withColumn("hudipartition", concat_ws("/", col("dt"), col("dn")))
 //   result.show()
 result.write.format("org.apache.hudi")
  .option("hoodie.insert.shuffle.parallelism", 12)
  .option("hoodie.upsert.shuffle.parallelism", 12)
  .option(DataSourceWriteOptions.RECORDKEY_FIELD_OPT_KEY, "uid") //指定记录键
  .option(DataSourceWriteOptions.PRECOMBINE_FIELD_OPT_KEY, "ts") //数据更新时间戳的
  .option(DataSourceWriteOptions.PARTITIONPATH_FIELD_OPT_KEY, "hudipartition") //hudi分区列
  .option("hoodie.table.name", "hudimembertest1") //hudi表名1
  .option(DataSourceWriteOptions.HIVE_URL_OPT_KEY, "jdbc:hive2://hadoop101:10000") //hiveserver2地址
  .option(DataSourceWriteOptions.HIVE_DATABASE_OPT_KEY, "default") //设置hudi与hive同步的数据库
  .option(DataSourceWriteOptions.HIVE_TABLE_OPT_KEY, "member1") //设置hudi与hive同步的表名
  .option(DataSourceWriteOptions.HIVE_PARTITION_FIELDS_OPT_KEY, "dt,dn") //hive表同步的分区列
  .option(DataSourceWriteOptions.HIVE_PARTITION_EXTRACTOR_CLASS_OPT_KEY, classOf[MultiPartKeysValueExtractor].getName) // 分区提取器 按/ 提取分区
  .option(DataSourceWriteOptions.HIVE_SYNC_ENABLED_OPT_KEY, "true") //设置数据集注册并同步到hive
  .option(HoodieIndexConfig.BLOOM_INDEX_UPDATE_PARTITION_PATH, "true") //设置当分区变更时，当前数据的分区目录是否变更
  .option(HoodieIndexConfig.INDEX_TYPE_PROP, HoodieIndex.IndexType.GLOBAL_BLOOM.name()) //设置索引
  .option("hoodie.insert.shuffle.parallelism", "12")
  .option("hoodie.upsert.shuffle.parallelism", "12")
  .mode(SaveMode.Append)
  .save("/user/atguigu/hudi/hudimembertest1")

 result.write.format("org.apache.hudi")
  .option("hoodie.insert.shuffle.parallelism", 12)
  .option("hoodie.upsert.shuffle.parallelism", 12)
  .option(DataSourceWriteOptions.RECORDKEY_FIELD_OPT_KEY, "uid") //指定记录键
  .option(DataSourceWriteOptions.PRECOMBINE_FIELD_OPT_KEY, "ts") //数据更新时间戳的
  .option(DataSourceWriteOptions.PARTITIONPATH_FIELD_OPT_KEY, "hudipartition") //hudi分区列
  .option("hoodie.table.name", "hudimembertest2") //hudi表名
  .option(DataSourceWriteOptions.HIVE_URL_OPT_KEY, "jdbc:hive2://hadoop101:10000") //hiveserver2地址
  .option(DataSourceWriteOptions.HIVE_DATABASE_OPT_KEY, "default") //设置hudi与hive同步的数据库
  .option(DataSourceWriteOptions.HIVE_TABLE_OPT_KEY, "member2") //设置hudi与hive同步的表名
  .option(DataSourceWriteOptions.HIVE_PARTITION_FIELDS_OPT_KEY, "dt,dn") //hive表同步的分区列
  .option(DataSourceWriteOptions.HIVE_PARTITION_EXTRACTOR_CLASS_OPT_KEY, classOf[MultiPartKeysValueExtractor].getName) // 分区提取器 按/ 提取分区
  .option(DataSourceWriteOptions.HIVE_SYNC_ENABLED_OPT_KEY, "true") //设置数据集注册并同步到hive
  .option(HoodieIndexConfig.BLOOM_INDEX_UPDATE_PARTITION_PATH, "true") //设置当分区变更时，当前数据的分区目录是否变更
  .option(HoodieIndexConfig.INDEX_TYPE_PROP, HoodieIndex.IndexType.GLOBAL_BLOOM.name()) //设置索引
  .option("hoodie.insert.shuffle.parallelism", "12")
  .option("hoodie.upsert.shuffle.parallelism", "12")
  .mode(SaveMode.Append)
  .save("/user/atguigu/hudi/hudimembertest2")
}

修改完毕后，查看对应hdfs路径下的文件变化

![img](file:////tmp/wps-gree/ksohtml/wpsFsmfVM.jpg) 

![img](file:////tmp/wps-gree/ksohtml/wpskC1SVQ.jpg) 

可以看到对于新增数据，Merger_On_Read（读时合并表）采用增量日志的方式修改数据，而Copy_On_Write（写实时表）采用了全量更新出一份全新文件方式进行修改功能。

两者对比

![img](file:////tmp/wps-gree/ksohtml/wpss5KqOM.jpg) 

### 7.3.3查询数据

最后修改完毕之后，查询对应hive表，修改数据操作代码中将fullname字段的值改成了testName

  ![img](file:////tmp/wps-gree/ksohtml/wpsNKJveN.jpg)

  可以发现ro结尾的读优化视图没有发生变化。查询rt表

![img](file:////tmp/wps-gree/ksohtml/wpsUy2QON.jpg) 

rt表的数据发生了变化，实时视图是查询基础数据和日志数据的合并视图。最后查询member2的实时视图

![img](file:////tmp/wps-gree/ksohtml/wpsKVKl3P.jpg) 

### 7.3.4总结

由此可以理解为MERGE_ON_READ的表，是以增量的形式来记录表的数据，修改操作都以日志的形式保存。而COPY_ON_READ的表是以全量覆盖的方式进行保存数据，每次有修改操作那么会和历史数据重新合并生产一份新的数据文件，并且历史数据不会删除。

两种表视图，HoodieParquetInputFormat格式的只支持查询表的原始数据，HoodieParquetRealtimeInputFormat格式支持查询表和日志的合并视图提供最新的数据。

## 7.4问题

跑hive yarn模式会出现以下错误，

![img](file:////tmp/wps-gree/ksohtml/wpsSAS2EO.jpg) 

[root@hadoop101 lib]# cp hudi-hadoop-mr-bundle-0.6.0-SNAPSHOT.jar /opt/module/hadoop-3.1.3/share/hadoop/common/lib/

将hudi mr的jar包放到hadoop下，再次运行，跑rt读实时视图仍会报错

![img](file:////tmp/wps-gree/ksohtml/wps4wJ1pQ.jpg) 

但是读ro读优化视图就可以了

![img](file:////tmp/wps-gree/ksohtml/wpsChlawM.jpg) 

  所以建议采用spark on hive方式，通过spark去读hive表