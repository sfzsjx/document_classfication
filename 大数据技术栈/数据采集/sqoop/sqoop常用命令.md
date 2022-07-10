# Sqooop

## 1. 简介

>​	Sqoop是一种旨在在Hadoop与关系数据库或大型机之间传输数据的工具。您可以使用Sqoop将数据从MySQL或Oracle等关系数据库管理系统（RDBMS）或大型机导入Hadoop分布式文件系统（HDFS），在Hadoop MapReduce中转换数据，然后将数据导出回RDBMS 。

## 2. Sqoop import

mysql -> hive

```sh
#!bin/sh
srcConnect="jdbc:mysql://host:3306/db?user=user&password=password&characterEncoding=utf-8"
dstDbPath=hdfs://nameservice1/extwarehouse/gree_product_analysisall/
####################talbe start##################################

dstTabName=test
jobName=job_test
#校验列
checkColumn="module"

srcQuery="select * from ${dstTabName} where \$CONDITIONS"

####################talbe end#################################################################

############################### execution part #############################################

   #tempTabName=${dstTabName}_temp
   #表数据目录
   TabPath=${dstDbPath}/${dstTabName}
   #表临时数据目录
   tempTabPath=${TabPath}_temp


   #将对应目录下的文件移动到临时文件目录下
   hadoop fs -mv $TabPath $tempTabPath 1>/dev/null 2>&1

   #删除正式文件
   hadoop fs -rm -f -r $TabPath 1>/dev/null 2>&1

   #删除sqoop任务
   sqoop job --meta-connect jdbc:hsqldb:hsql://hostname:16000/sqoop -delete $jobName

   #创建sqoop任务
   sqoop job -D mapreduce.job.queuename=dev1 --meta-connect jdbc:hsqldb:hsql://hostname:16000/sqoop --create $jobName -- import --connect "$srcConnect" --query "$srcQuery" --split-by "$checkColumn" --target-dir $TabPath --lines-terminated-by '\n' --fields-terminated-by '\t' --null-string '\\N' --null-non-string '\\N' -m 1

   #执行sqoop任务
   sqoop job --exec $jobName --meta-connect jdbc:hsqldb:hsql://hostname:16000/sqoop

   #判断上面的语句是否执行成功，执行成功则删除重命名的hsdf文件，否则则恢复为正式文件
   if [ $? -eq "0"]
   then
   	hadoop fs -rm -f -r $tempTabPath
   else
   	hadoop fs -mv $tempTabPath $TabPath  1>/dev/null 2>&1
   	echo "HDFS数据导入失败"
   fi
   echo "************************ 数据导入完成 ***************************"
```



sqlserver -> hive

```sh
#!bin/sh
   srcConnect="jdbc:sqlserver://host;username=user;password=password;database=db;characterEncoding=utf-8"
   dstDbPath=hdfs://nameservice1/extwarehouse/gree_product_analysisall/
   ####################talbe start##################################

   dstTabName=test
   jobName=job_test
   #校验列
   checkColumn="module"

   srcQuery="select * from ${dstTabName} where \$CONDITIONS"

   ####################talbe end#####################################################################


   ############################### execution part #############################################

   #tempTabName=${dstTabName}_temp
   #表数据目录
   TabPath=${dstDbPath}/${dstTabName}
   #表临时数据目录
   tempTabPath=${TabPath}_temp


   #将对应目录下的文件移动到临时文件目录下
   hadoop fs -mv $TabPath $tempTabPath 1>/dev/null 2>&1

   #删除正式文件
   hadoop fs -rm -f -r $TabPath 1>/dev/null 2>&1

   #删除sqoop任务
   sqoop job --meta-connect jdbc:hsqldb:hsql://hostname:16000/sqoop -delete $jobName

   #创建sqoop任务
   sqoop job -D mapreduce.job.queuename=dev1 --meta-connect jdbc:hsqldb:hsql://hostname:16000/sqoop --create $jobName -- import --connect "$srcConnect" --query "$srcQuery" --split-by "$checkColumn" --target-dir $TabPath --lines-terminated-by '\n' --fields-terminated-by '\t' --null-string '\\N' --null-non-string '\\N' -m 1

   #执行sqoop任务
   sqoop job --exec $jobName --meta-connect jdbc:hsqldb:hsql://hostname:16000/sqoop

   #判断上面的语句是否执行成功，执行成功则删除重命名的hsdf文件，否则则恢复为正式文件
   if [ $? -eq "0"]
   then
   	hadoop fs -rm -f -r $tempTabPath
   else
   	hadoop fs -mv $tempTabPath $TabPath  1>/dev/null 2>&1
   	echo "HDFS数据导入失败"
   fi
   echo "************************ 数据导入完成 ***************************"
```

