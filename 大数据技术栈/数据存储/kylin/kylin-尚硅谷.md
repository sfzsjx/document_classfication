# 第1章 概述

## 1.1 Kylin定义。

Apache Kylin是一个开源的分布式分析引擎，提供Hadoop/Spark 之上的SQL查询接口多维分析(OLAP)能力以支持超大规模数据，最初由eBay开发并贡献至开源社区。它能在亚秒内查询巨大的Hive表。

**Data Warehouse(数据仓库)**
数据仓库是一个各种数据（包括历史数据和当前数据).的中心存储系统( **business intelligence**，商业智能）的核心部件。这里所谈的数据包括来自企业业务系统的订单、库存、.交易账目、客户和供应商等来自企业所处行业和竞争对手的数据以及来自企业所处的其他外部环境中
的各种数据。

**Business Intelligence(商业智能)**
商业智能通常被理解为将企业中现有的数据转化为知识)帮助企业做出明智的业务经营决策的工具。为了将数据转化为知识，需要利用数据仓库，联机分析处理(OLAP)工具和数据挖掘等技术。

**OLAP(online analytical processing)**
OLAP(online analytical processing）是一种软件技术，它使分析人员能够迅速、一致、交互地从各个方面观察信息，以达到深入理解数据的目的。从各方面观察信息，也就是从不同的维度分析数据，因此OLAP也成为`多维分析`。

![image-20210124203216095](kylin-尚硅谷.assets/image-20210124203216095.png)

![image-20210124203339411](kylin-尚硅谷.assets/image-20210124203339411.png)

**OLAP Cube**
MOLAP基于多维数据集，一个多维数据集称为一个OLAP Cube

![image-20210124203535898](kylin-尚硅谷.assets/image-20210124203535898.png)

![image-20210124203858707](kylin-尚硅谷.assets/image-20210124203858707.png)

**Cuboid**: 长方体

**Star Schema(星型模型)**

![image-20210124204222563](kylin-尚硅谷.assets/image-20210124204222563.png)



![image-20210124204352420](kylin-尚硅谷.assets/image-20210124204352420.png)

 ![image-20210124204423825](kylin-尚硅谷.assets/image-20210124204423825.png)

**维度:分析数据的数据的角度**

**度量:被分析的指标**



## 1.2 Kylin架构

![image-20210124205204522](kylin-尚硅谷.assets/image-20210124205204522.png)



**1) REST Server** 
	REST Server是一套面向应用程序开发的入口点，旨在实现针对Kylin平台的应用开发工作。此类应用程序可以提供查询、获取结果、触发cube构建任务、获取元数据以及获取用户权限等等。另外可以通过Restful接口实现SQL查询。

**2〉查询引擎(Queryngine）** 
	当cube 准备就绪后，查询引擎就能够获取并解析用户查询。它随后会与系统中的其它组件进行交互，从而向用户返回对应的结果。·

**3)路由器（Routing）**
	在最初设计时曾考虑过将Kylin不能执行的查询引导去Hive中继续执行，但在实践后发现Hive 与Kylin的速度差异过大，导致用户无法对查询的速度有一致的期望，很可能大多数查询几秒内就返回结果了，而有些查询则要等几分钟到几十分钟，因此体验非常糟糕。最后这个路由功能在发行版中默认关闭。。

**4）元数据管理工具(Metadata）**
	Kylin是一款元数据驱动型应用程序。元数据管理工具是一大关键性组件，用于对保存在Kylin当中的所有元数据进行管理，其中包括最为重要的cube元数据。其它全部组件的正常运作都需以元数据管理工具为基础。Kylin的元数据存储在 hbase中。.

**5)任务引擎（Cube Build Engine）** 
	这套引擎的设计目的在于处理所有离线任务，其中包括shell脚本、Java API 以及 MapReduce任务等等。任务引擎对Kylin当中的全部任务加以管理与协调，从而确保每一项任务都能得到切实执行并解决其间出现的故障。

## 1.3 Kylin特点

Kylin 的主要特点包括支持SQL接口、支持超大规模数据集、亚秒级响应、可伸缩性、高吞吐率、BI工具集成等。·

1）`标准SQL接口`：Kylin是以标准的SQL作为对外服务的接口。

2〉`支持超大数据集`:Kylin对于大数据的支撑能力可能是目前所有技术中最为领先的。早在2015年 eBay 的生产环境中就能支持百亿记录的秒级查询，之后在移动的应用场景中又有了千亿记录秒级查询的案例。·

3)`亚秒级响应`:Kylin拥有优异的查询相应速度,这点得益于预计算，很多复杂的计算，比如连接、聚合,在离线的预计算过程中就已经完成,这大大降低了查询时刻所需的计算量，提高了响应速度。·

4）`可伸缩性和高吞吐率`:单节点Kylin可实现每秒70个查询，还可以搭建Kylin的集群。

5)`BI工具集成`：Kylin可以与现有的BI工具集成，具体包括如下内容。

ODBC:与Tableau、Excel、PowerBI等工具集成

JDBC:与 Saiku、BIRT等Java工具集成。

RestAPI:与JavaScript、Web网页集成·

Kylin开发团队还贡献了Zepplin的插件，也可以使用Zepplin来访问Kylin服务。

# 第2章Kylin环境搭建。

## 2.1安装地址

1）官网地址

http://kylin.apache.org/cn/ 

2）官方文档

http://lkylin.apache.org/cn/docs/

3）下载地址。

http://kylin.apache.orglcn/download/ 

## 2.2安装部署

1）将apache-kylin-2.5.1-bin-hbase1x.tar.gz 上传到Linuxw

2）解压apache-kylin-2.5.1-bin-hbase1x.tar.gz到/opt/module

 ```sh
[atguigu@hadoop102 sorfware]$ tar -zxvf apache-kylin-2.5.1-bin-hbase1x.tar.gz -c / opt/module/ 
 ```

注意:需要在/etc/profile文件中配置 HADOOP_HOME，HIVE_HOME，HBASE_HOME 并
source使其生效。

3）启动。

```sh
[atguigu@hadoop102 kylin]$ bin/kylin.sh start
```

启动之后查看各个节点进程:·

```sh
-————----- hadoop102 ---------
3360 JobHistoryserver
31425 HMaster
3282 NodeManager
3026 DataNode
53283 Jps
2886 NameNode
44007 RunJar
2728 QuorumPeerMain
31566 HRegionservere
----------- hadoop103 ----------
5040 HMaster
2864 ResourceManager
9729 Jps
2657 QuorumPeerMain
4946 HRegionserver
2979 NodeManager
2727 DataNode 
------------------- hadoop104 ------------
4688 HRegionserver
2900 NodeManager
9848 Jps
2636 QuorumPeerMain
2700 DataNode
2815 SecondaryNameNode
```

注意:启动Kylin之前要保证HDFS，YARN，ZK，HBASE相关进程是正常运行的。

在 http://hadoop102:7070/kylin查看Web页面

![image-20210124211120853](kylin-尚硅谷.assets/image-20210124211120853.png)

**用户名为:ADMIN，密码为:KYLIN（系统已填)。**

4）关闭,

```sh
[atguiguhadoop102 kylin]$ bin/ kylin.sh stop
```

# 第3章快速入门

需求:实现按照维度（工作地点）统计员工信息

## 3.1数据准备

在Hive中创建数据，分别创建部门和员工外部表，并向表中导入数据。

(1) 原始数据

 ![image-20210124211258436](kylin-尚硅谷.assets/image-20210124211258436.png)

(2）建表语句,

**创建部门表**

```sql
create external table if not exists default.dept
(deptno int, 
dname string,
 loc int) row format delimited fields terminated by '\t';
```

 **创建员工表**

```sql
create external table if not exists default.emp ( empno int, 
ename string ,
job string,mgr int, 
hiredate string,sal double,
comm double, deptno int) row format delimited fields terminated by '\t';
```

(3)查看创建的表

```sh
hive (default)> show tables;
OK
tab_namew
dept
emp
```

(4）向外部表中导入数据导入数据

```sh
hive (default)> load data local inpath 'lopt/module/datas/dept.txt'into table default.dept; 
hive (default)> load data local inpath '/opt/module/datas/emp.txt'into table default.emp; 
```

查询结果

```sh
hive (default)> select * from emp;
hive (default)> select * from dept; 
```

## 3.2创建项目

### 3.2.1登录系统

![image-20210124211958828](kylin-尚硅谷.assets/image-20210124211958828.png)

![image-20210124212024455](kylin-尚硅谷.assets/image-20210124212024455.png)



### 3.2.2创建工程

1) 点击图上所示“+”号

![image-20210125200722679](kylin-尚硅谷.assets/image-20210125200722679.png)

2）填入项目名及描述点击Submit

![image-20210125200809118](kylin-尚硅谷.assets/image-20210125200809118.png)





### 3.2.3选择数据源

1）选择加载数据源方式

![image-20210125200900501](kylin-尚硅谷.assets/image-20210125200900501.png)

2）输入要作为数据源的表

![image-20210125200932206](kylin-尚硅谷.assets/image-20210125200932206.png)

3）查看数据源

![image-20210125200959279](kylin-尚硅谷.assets/image-20210125200959279.png)





## 3.3创建Model

1）回到Models页面

![image-20210125201035462](kylin-尚硅谷.assets/image-20210125201035462.png)

2）点击New按钮后点击New Model

![image-20210125201103183](kylin-尚硅谷.assets/image-20210125201103183.png)

3）填写Model名称及描述后Next

![image-20210125201126893](kylin-尚硅谷.assets/image-20210125201126893.png)

4）选择事实表

![image-20210125205032587](kylin-尚硅谷.assets/image-20210125205032587.png)

5）添加维度表

![image-20210125205102482](kylin-尚硅谷.assets/image-20210125205102482.png)

6）选择添加的维度表及join字段

![image-20210125205123410](kylin-尚硅谷.assets/image-20210125205123410.png)

![image-20210125205137931](kylin-尚硅谷.assets/image-20210125205137931.png)

7）选择维度信息

![image-20210125205202377](kylin-尚硅谷.assets/image-20210125205202377.png)

8）选择度量信息

![image-20210125205229425](kylin-尚硅谷.assets/image-20210125205229425.png)



9）添加分区信息及过滤条件之后“Save”

![image-20210125205252219](kylin-尚硅谷.assets/image-20210125205252219.png)

10）创建Model完成

![image-20210125205311506](kylin-尚硅谷.assets/image-20210125205311506.png)



## 3.4创建Cube

1）点击New按钮然后选择New Cube

![image-20210125205339737](kylin-尚硅谷.assets/image-20210125205339737.png)

2）选择Model及填写Cube Name

![image-20210125205357089](kylin-尚硅谷.assets/image-20210125205357089.png)

3）添加维度

![image-20210125205414914](kylin-尚硅谷.assets/image-20210125205414914.png)

![image-20210125205430297](kylin-尚硅谷.assets/image-20210125205430297.png)



4）添加需要做预计算的内容

![image-20210125205452123](kylin-尚硅谷.assets/image-20210125205452123.png)

![image-20210125205502810](kylin-尚硅谷.assets/image-20210125205502810.png)

5）动态更新相关（默认）

![image-20210125205522435](kylin-尚硅谷.assets/image-20210125205522435.png)

6）高阶模块（默认）

![image-20210125205542921](kylin-尚硅谷.assets/image-20210125205542921.png)

7）需要修改的配置

![image-20210125205601857](kylin-尚硅谷.assets/image-20210125205601857.png)

8）Cube信息展示

![image-20210125205621145](kylin-尚硅谷.assets/image-20210125205621145.png)

9）Cube配置完成

![image-20210125205638242](kylin-尚硅谷.assets/image-20210125205638242.png)

10）触发预计算

![image-20210125205655714](kylin-尚硅谷.assets/image-20210125205655714.png)

11）查看Build进度

![image-20210125205720747](kylin-尚硅谷.assets/image-20210125205720747.png)

12）构建Cube完成

![image-20210125205741745](kylin-尚硅谷.assets/image-20210125205741745.png)

## 3.5 Hive和Kylin性能对比

需求：根据部门名称[dname]统计员工薪资总数[sum（sal）]



### 3.5.1 Hive查询

```sh
hive> select dname,sum(sal) from emp e join dept d on e.deptno = d.deptno group by dname;
Query ID = atguigu_20181210104140_4931b735-5bad-4a4f-bce6-67985b8fe30a
Total jobs = 1
SLF4J: Class path contains multiple SLF4J bindings.
… …
Stage-Stage-2: Map: 1  Reduce: 1   Cumulative CPU: 3.95 sec   HDFS Read: 13195 HDFS Write: 48 SUCCESS
Total MapReduce CPU Time Spent: 3 seconds 950 msec
OK
ACCOUNTING      3750.0
RESEARCH        10875.0
SALES   9400.0
Time taken: 23.893 seconds, Fetched: 3 row(s)
hive>
```



### 3.5.2 Kylin查询

1）进入Insight页面

![image-20210125205913498](kylin-尚硅谷.assets/image-20210125205913498.png)

2）在New Query中输入查询语句并Submit

![image-20210125205932258](kylin-尚硅谷.assets/image-20210125205932258.png)

3）数据图表展示及导出

![image-20210125205954393](kylin-尚硅谷.assets/image-20210125205954393.png)

4）图表展示之条形图

![image-20210125210011466](kylin-尚硅谷.assets/image-20210125210011466.png)

4）图表展示之饼图

![image-20210125210033801](kylin-尚硅谷.assets/image-20210125210033801.png)

# 第4章Cube构建原理

## 4.1 Cube构建流程



![image-20210125210236089](kylin-尚硅谷.assets/image-20210125210236089.png)

## 4.2 Cube构建算法



### 4.2.1逐层构建算法

![image-20210125210327651](kylin-尚硅谷.assets/image-20210125210327651.png)

我们知道，一个N维的Cube，是由1个N维子立方体、N个(N-1)维子立方体、N*(N-1)/2个(N-2)维子立方体、......、N个1维子立方体和1个0维子立方体构成，总共有2^N个子立方体组成，在逐层算法中，按维度数逐层减少来计算，每个层级的计算（除了第一层，它是从原始数据聚合而来），是基于它上一层级的结果来计算的。比如，[Group by A, B]的结果，可以基于[Group by A, B, C]的结果，通过去掉C后聚合得来的；这样可以减少重复计算；当 0维度Cuboid计算出来的时候，整个Cube的计算也就完成了。每一轮的计算都是一个MapReduce任务，且串行执行；一个N维的Cube，至少需要N次MapReduce Job。

![image-20210125210413112](kylin-尚硅谷.assets/image-20210125210413112.png)

算法优点：

1）此算法充分利用了MapReduce的优点，处理了中间复杂的排序和shuffle工作，故而算法代码清晰简单，易于维护；

2）受益于Hadoop的日趋成熟，此算法非常稳定，即便是集群资源紧张时，也能保证最终能够完成。



算法缺点：

1）当Cube有比较多维度的时候，所需要的MapReduce任务也相应增加；由于Hadoop的任务调度需要耗费额外资源，特别是集群较庞大的时候，反复递交任务造成的额外开销会相当可观；

2）由于Mapper逻辑中并未进行聚合操作，所以每轮MR的shuffle工作量都很大，导致效率低下。

3）对HDFS的读写操作较多：由于每一层计算的输出会用做下一层计算的输入，这些Key-Value需要写到HDFS上；当所有计算都完成后，Kylin还需要额外的一轮任务将这些文件转成HBase的HFile格式，以导入到HBase中去；

总体而言，该算法的效率较低，尤其是当Cube维度数较大的时候。

### 4.2.2快速构建算法

![image-20210125210504289](kylin-尚硅谷.assets/image-20210125210504289.png)

也被称作“逐段”(By Segment) 或“逐块”(By Split) 算法，从1.5.x开始引入该算法，该算法的主要思想是，每个Mapper将其所分配到的数据块，计算成一个完整的小Cube 段（包含所有Cuboid）。每个Mapper将计算完的Cube段输出给Reducer做合并，生成大Cube，也就是最终结果。如图所示解释了此流程。

![image-20210125210529903](kylin-尚硅谷.assets/image-20210125210529903.png)

与旧算法相比，快速算法主要有两点不同：

1） Mapper会利用内存做预聚合，算出所有组合；Mapper输出的每个Key都是不同的，这样会减少输出到Hadoop MapReduce的数据量，Combiner也不再需要；

2）一轮MapReduce便会完成所有层次的计算，减少Hadoop任务的调配。

# 第5章Cube构建优化

从之前章节的介绍可以知道，在没有采取任何优化措施的情况下，Kylin会对每一种维度的组合进行预计算，每种维度的组合的预计算结果被称为Cuboid。假设有4个维度，我们最终会有24 =16个Cuboid需要计算。

但在现实情况中，用户的维度数量一般远远大于4个。假设用户有10 个维度，那么没有经过任何优化的Cube就会存在210 =1024个Cuboid；而如果用户有20个维度，那么Cube中总共会存在220 =1048576个Cuboid。虽然每个Cuboid的大小存在很大的差异，但是单单想到Cuboid的数量就足以让人想象到这样的Cube对构建引擎、存储引擎来说压力有多么巨大。因此，在构建维度数量较多的Cube时，尤其要注意Cube的剪枝优化（即减少Cuboid的生成）。

## 5.1使用衍生维度（derived dimension）

衍生维度用于在有效维度内将维度表上的非主键维度排除掉，并使用维度表的主键（其实是事实表上相应的外键）来替代它们。Kylin会在底层记录维度表主键与维度表其他维度之间的映射关系，以便在查询时能够动态地将维度表的主键“翻译”成这些非主键维度，并进行实时聚合。

![image-20210125210712063](kylin-尚硅谷.assets/image-20210125210712063.png)

虽然衍生维度具有非常大的吸引力，但这也并不是说所有维度表上的维度都得变成衍生维度，如果从维度表主键到某个维度表维度所需要的聚合工作量非常大，则不建议使用衍生维度。



## 5.2使用聚合组（Aggregation group）

聚合组（Aggregation Group）是一种强大的剪枝工具。聚合组假设一个Cube的所有维度均可以根据业务需求划分成若干组（当然也可以是一个组），由于同一个组内的维度更可能同时被同一个查询用到，因此会表现出更加紧密的内在关联。每个分组的维度集合均是Cube所有维度的一个子集，不同的分组各自拥有一套维度集合，它们可能与其他分组有相同的维度，也可能没有相同的维度。每个分组各自独立地根据自身的规则贡献出一批需要被物化的Cuboid，所有分组贡献的Cuboid的并集就成为了当前Cube中所有需要物化的Cuboid的集合。不同的分组有可能会贡献出相同的Cuboid，构建引擎会察觉到这点，并且保证每一个Cuboid无论在多少个分组中出现，它都只会被物化一次。

对于每个分组内部的维度，用户可以使用如下三种可选的方式定义，它们之间的关系，具体如下。

1）`强制维度（Mandatory）`，如果一个维度被定义为强制维度，那么这个分组产生的所有Cuboid中每一个Cuboid都会包含该维度。每个分组中都可以有0个、1个或多个强制维度。如果根据这个分组的业务逻辑，则相关的查询一定会在过滤条件或分组条件中，因此可以在该分组中把该维度设置为强制维度。

![image-20210125210826144](kylin-尚硅谷.assets/image-20210125210826144.png)

2）`层级维度（Hierarchy）`，每个层级包含两个或更多个维度。假设一个层级中包含D1，D2…Dn这n个维度，那么在该分组产生的任何Cuboid中， 这n个维度只会以（），（D1），（D1，D2）…（D1，D2…Dn）这n+1种形式中的一种出现。每个分组中可以有0个、1个或多个层级，不同的层级之间不应当有共享的维度。如果根据这个分组的业务逻辑，则多个维度直接存在层级关系，因此可以在该分组中把这些维度设置为层级维度。

![image-20210125210917151](kylin-尚硅谷.assets/image-20210125210917151.png)

3）`联合维度（Joint）`，每个联合中包含两个或更多个维度，如果某些列形成一个联合，那么在该分组产生的任何Cuboid中，这些联合维度要么一起出现，要么都不出现。每个分组中可以有0个或多个联合，但是不同的联合之间不应当有共享的维度（否则它们可以合并成一个联合）。如果根据这个分组的业务逻辑，多个维度在查询中总是同时出现，则可以在该分组中把这些维度设置为联合维度。

![image-20210125210944009](kylin-尚硅谷.assets/image-20210125210944009.png)

这些操作可以在Cube Designer的Advanced Setting中的Aggregation Groups区域完成，如下图所示。

![image-20210125211005809](kylin-尚硅谷.assets/image-20210125211005809.png)

聚合组的设计非常灵活，甚至可以用来描述一些极端的设计。假设我们的业务需求非常单一，只需要某些特定的Cuboid，那么可以创建多个聚合组，每个聚合组代表一个Cuboid。具体的方法是在聚合组中先包含某个Cuboid所需的所有维度，然后把这些维度都设置为强制维度。这样当前的聚合组就只能产生我们想要的那一个Cuboid了。

再比如，有的时候我们的Cube中有一些基数非常大的维度，如果不做特殊处理，它就会和其他的维度进行各种组合，从而产生一大堆包含它的Cuboid。包含高基数维度的Cuboid在行数和体积上往往非常庞大，这会导致整个Cube的膨胀率变大。如果根据业务需求知道这个高基数的维度只会与若干个维度（而不是所有维度）同时被查询到，那么就可以通过聚合组对这个高基数维度做一定的“隔离”。我们把这个高基数的维度放入一个单独的聚合组，再把所有可能会与这个高基数维度一起被查询到的其他维度也放进来。这样，这个高基数的维度就被“隔离”在一个聚合组中了，所有不会与它一起被查询到的维度都没有和它一起出现在任何一个分组中，因此也就不会有多余的Cuboid产生。这点也大大减少了包含该高基数维度的Cuboid的数量，可以有效地控制Cube的膨胀率。



## 5.4 Row Key优化

Kylin会把所有的维度按照顺序组合成一个完整的Rowkey，并且按照这个Rowkey升序排列Cuboid中所有的行。

设计良好的Rowkey将更有效地完成数据的查询过滤和定位，减少IO次数，提高查询速度，维度在rowkey中的次序，对查询性能有显著的影响。

Row key的设计原则如下：

**1）被用作where过滤的维度放在前边。**

**Row Key调优**

![image-20210125211140144](kylin-尚硅谷.assets/image-20210125211140144.png)

**2）基数大的维度放在基数小的维度前边。**

![image-20210125211227297](kylin-尚硅谷.assets/image-20210125211227297.png)







## 5.3并发粒度优化

当Segment中某一个Cuboid的大小超出一定的阈值时，系统会将该Cuboid的数据分片到多个分区中，以实现Cuboid数据读取的并行化，从而优化Cube的查询速度。具体的实现方式如下：构建引擎根据Segment估计的大小，以及参数“kylin.hbase.region.cut”的设置决定Segment在存储引擎中总共需要几个分区来存储，如果存储引擎是HBase，那么分区的数量就对应于HBase中的Region数量。kylin.hbase.region.cut的默认值是5.0，单位是GB，也就是说对于一个大小估计是50GB的Segment，构建引擎会给它分配10个分区。用户还可以通过设置kylin.hbase.region.count.min（默认为1）和kylin.hbase.region.count.max（默认为500）两个配置来决定每个Segment最少或最多被划分成多少个分区。

![image-20210125211304239](kylin-尚硅谷.assets/image-20210125211304239.png)

由于每个Cube的并发粒度控制不尽相同，因此建议在Cube Designer 的Configuration Overwrites（上图所示）中为每个Cube量身定制控制并发粒度的参数。假设将把当前Cube的kylin.hbase.region.count.min设置为2，kylin.hbase.region.count.max设置为100。这样无论Segment的大小如何变化，它的分区数量最小都不会低于2，最大都不会超过100。相应地，这个Segment背后的存储引擎（HBase）为了存储这个Segment，也不会使用小于两个或超过100个的分区。我们还调整了默认的kylin.hbase.region.cut，这样50GB的Segment基本上会被分配到50个分区，相比默认设置，我们的Cuboid可能最多会获得5倍的并发量。











# 第6章BI工具集成

可以与Kylin结合使用的可视化工具很多，例如：

ODBC：与Tableau、Excel、PowerBI等工具集成

JDBC：与Saiku、BIRT等Java工具集成

RestAPI：与JavaScript、Web网页集成

Kylin开发团队还贡献了Zepplin的插件，也可以使用Zepplin来访问Kylin服务。



## 4.1 JDBC

1）新建项目并导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.kylin</groupId>
        <artifactId>kylin-jdbc</artifactId>
        <version>2.5.1</version>
    </dependency>
</dependencies>
```

2）编码

```java
package com.atguigu;

import java.sql.*;

public class TestKylin {

    public static void main(String[] args) throws Exception {

        //Kylin_JDBC 驱动
        String KYLIN_DRIVER = "org.apache.kylin.jdbc.Driver";

        //Kylin_URL
        String KYLIN_URL = "jdbc:kylin://hadoop102:7070/FirstProject";

        //Kylin的用户名
        String KYLIN_USER = "ADMIN";

        //Kylin的密码
        String KYLIN_PASSWD = "KYLIN";

        //添加驱动信息
        Class.forName(KYLIN_DRIVER);

        //获取连接
        Connection connection = DriverManager.getConnection(KYLIN_URL, KYLIN_USER, KYLIN_PASSWD);

        //预编译SQL
        PreparedStatement ps = connection.prepareStatement("SELECT sum(sal) FROM emp group by deptno");

        //执行查询
        ResultSet resultSet = ps.executeQuery();

        //遍历打印
        while (resultSet.next()) {
            System.out.println(resultSet.getInt(1));
        }
    }
}
```

3）结果展示

![image-20210125211600972](kylin-尚硅谷.assets/image-20210125211600972.png)

## 4.2 Zepplin

### 4.2.1 Zepplin安装与启动



1）将zeppelin-0.8.0-bin-all.tgz上传至Linux

2）解压zeppelin-0.8.0-bin-all.tgz之/opt/module

```sh
[atguigu@hadoop102 sorfware]$ tar -zxvf zeppelin-0.8.0-bin-all.tgz -C /opt/module/
```

3）修改名称

```sh
[atguigu@hadoop102 module]$ mv zeppelin-0.8.0-bin-all/ zeppelin
```

4）启动

```sh
[atguigu@hadoop102 zeppelin]$ bin/zeppelin-daemon.sh start
```

可登录网页查看，web默认端口号为8080

http://hadoop102:8080

![image-20210125211730472](kylin-尚硅谷.assets/image-20210125211730472.png)

### 4.2.2 配置Zepplin支持Kylin

1）点击右上角anonymous选择Interpreter

![image-20210125211757583](kylin-尚硅谷.assets/image-20210125211757583.png)

2）搜索Kylin插件并修改相应的配置

![image-20210125211817369](kylin-尚硅谷.assets/image-20210125211817369.png)

3）修改完成点击Save完成

![image-20210125211836137](kylin-尚硅谷.assets/image-20210125211836137.png)

### 4.3.3 案例实操

需求：查询员工详细信息，并使用各种图表进行展示

1）点击Notebook创建新的note

![image-20210125211902361](kylin-尚硅谷.assets/image-20210125211902361.png)

2）填写Note Name点击Create

![image-20210125211923976](kylin-尚硅谷.assets/image-20210125211923976.png)

![image-20210125211936174](kylin-尚硅谷.assets/image-20210125211936174.png)

3）执行查询

![image-20210125211952190](kylin-尚硅谷.assets/image-20210125211952190.png)

4）结果展示

![image-20210125212010524](kylin-尚硅谷.assets/image-20210125212010524.png)

5）其他图表格式

![image-20210125212032201](kylin-尚硅谷.assets/image-20210125212032201.png)

![image-20210125212045041](kylin-尚硅谷.assets/image-20210125212045041.png)

![image-20210125212058587](kylin-尚硅谷.assets/image-20210125212058587.png)

![image-20210125212109303](kylin-尚硅谷.assets/image-20210125212109303.png)

![image-20210125212119716](kylin-尚硅谷.assets/image-20210125212119716.png)