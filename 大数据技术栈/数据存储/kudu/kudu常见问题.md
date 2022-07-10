

## 1. kudu owner can not empry or null

```scala
val impala_driver = "com.cloudera.impala.jdbc41.Driver"
val sslpath = "cm-auto-in_cluster_truststore.jks" 
val IMPALA_KRBS_URL: String = "jdbc:impala://cdh-master03:25004/cdm_erp_view_419;AuthMech=3;SSL=1;SSLTrustStore=" + sslpath +";request_pool=dev1" 
```

关键在与AuthMech=3;SSL=1

## 2. kudu impala 建表

```sql
CREATE TABLE my_first_table
(id BIGINT,
name STRING,
PRIMARY KEY(id)
 )
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES (
'kudu.master addresses' = 'node1:7051, node2:7051, node3:7051'
);
```

