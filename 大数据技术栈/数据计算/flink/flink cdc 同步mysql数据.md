# Flink cdc

> CDC (change data capture) 主要用于同步数据，它可以捕获源数据变化，实时同步到多个目标数据源，保证数据的一致性。
>
> 业界主要分为两种同步方案：
>
> ​	1.基于查询的CDC（主要每次捕获变更发起的select查询进行全表扫描，过滤查询之间变更的数据）
>
> ​	2.基于日志的CDC（读取数据存储系统的log，例如mysql里面的binlog持续监控）

flink cdc mysql

```sql
-- creates a mysql cdc table source
CREATE TABLE mysql_binlog (
 id INT NOT NULL,
 name STRING,
 description STRING,
 weight DECIMAL(10,3),
 PRIMARY KEY(id) NOT ENFORCED
) WITH (
 'connector' = 'mysql-cdc',
 'hostname' = 'localhost',
 'port' = '3306',
 'username' = 'flinkuser',
 'password' = 'flinkpw',
 'database-name' = 'inventory',
 'table-name' = 'products'
);

-- read snapshot and binlog data from mysql, and do some transformation, and show on the client
SELECT id, UPPER(name), description, weight FROM mysql_binlog;
```

