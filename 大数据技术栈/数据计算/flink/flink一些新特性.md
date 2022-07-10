## 1. flink watermark 设置

```java
//注意泛型SourceDataSchema
//设置水印
                .assignTimestampsAndWatermarks(WatermarkStrategy.<SourceDataSchema>forBoundedOutOfOrderness(Duration.ofSeconds(5))
                .withTimestampAssigner((element, recordTimestamp) -> element.getTime()))
```





## 2. flink 从checkpoint中重启

> flink run --fromSavepoint /xxx/xxx/chk-14  xxx.jar
>
> 可以从webui界面查看checpoint是否是从checkpoint中restore的
>
> **savepoint**
>
> flink run -s /tmp/state.backend/s1/savepoint-17b840-2cfe3bd5bc0c -c flink.HelloWorld target/scala-flink-0.1.jar
>
> **checkpoint**
>
> flink run -s /tmp/state.backend/17b840a3d2221b1400ec03f7e3949b17/chk-960 -c flink.HelloWorld target/scala-flink-0.1.jar





参考：[FLILNK CHECKPOINT 和 SavePOINT](https://blog.csdn.net/c13232906050/article/details/83342131?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)



## 3. FLINK 任务部署

### 3.1 flink 本地部署

```sh
1. 首先，本地搭建flink，启动flink
```

![1621559198557](flink一些新特性.assets/1621559198557.png)

![1621559308947](flink一些新特性.assets/1621559308947.png)

```sh
打包flink程序，执行命令
#!bin/bash
cd /opt/flink/test
rm -rf intelligent_transportation_system/
git clone -b dev http://sfz:sfzsjx12@119.23.107.222/smart-commuting/intelligent_transportation_system.git

cd /opt/flink/test/intelligent_transportation_system/transportation-track/

mvn clean package


nohup flink run -p 4 -c com.leayun.track.TransportationTrack    /opt/flink/test/intelligent_transportation_system/transportation-track/target/transportation-track-1.0-SNAPSHOT-jar-with-dependencies.jar  --paramProperties /opt/flink/test/intelligent_transportation_system/transportation-track/src/main/resources/flink-test.properties  > /opt/flink/test/transport-track-test.log 2>&1  &
```



### 3.2 docker 部署flink程序

> 1. 安装docker模式flink应用，详见 基于docker模式安装部署flink应用
> 2. 打包flink应用，上传jar包到服务器上，并将jar包放入docker映射目录下
> 3. docker exec -it jobmanager bash
> 4. cd bin
> 5. flink run /opt/xxx/xxx.jar



### 3.3 基于hue网页提交flink任务

> 1. 登陆hue页面
> 2. 上传jar到hdfs上，放入用户目录下
> 3. 下载flink配置文件，并上传到conf目录下
> 4. 创建workflow任务，编写shell脚本
> 5. 设置jar包和配置文件目录,在属性中设置队列 mapred.job.queue.name = dev1，保存并执行任务

```sh
#!/bin/bash
env -i FLINK_CONF_DIR=./conf /lvm/data3/flink/flink-1.8.1/bin/flink run -m yarn-cluster -yn 4 -ytm 2g -ys 4 -yqu root.dev1 -ynm dubhe-flink-A   ./commutator-1.0-SNAPSHOT-jar-with-dependencies.jar  --queueName A_newline
```



## 4. flink sql

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.leayun</groupId>
    <artifactId>flink-sql-learn</artifactId>
    <version>1.0-SNAPSHOT</version>


    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_2.11</artifactId>
            <version>1.10.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-csv</artifactId>
            <version>1.10.1</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.5.5</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.leayun.track.TransportationTrack</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.6.0.1398</version>
            </plugin>
        </plugins>
    </build>
</project>
```

```java
package com.leayun;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.DataTypes;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.java.StreamTableEnvironment;
import org.apache.flink.table.descriptors.Csv;
import org.apache.flink.table.descriptors.Kafka;
import org.apache.flink.table.descriptors.Schema;

/**
 * @author 260164
 */
public class FlinkSqlLearn {
    public static void main(String[] args) {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        String tableName = "kafkaInputTable";
        tableEnv.connect(
                new Kafka()
                .version("2.1.0")
                .topic("sensor")
                .property("bootstrap.servers", "localhost:9092"))
                .withFormat(new Csv())
                .withSchema(new Schema().field("id", DataTypes.STRING()))
                .createTemporaryTable(tableName);
        Table res = tableEnv.from(tableName).select("id");

        res.insertInto(tableName);


    }
}
```



## 5.flink cdc 

1. 下载 `docker-compose.yml`

   ```yml
   version: '2.1'
   services:
     postgres:
       image: debezium/example-postgres:1.1
       ports:
         - "5432:5432"
       environment:
         - POSTGRES_PASSWORD=1234
         - POSTGRES_DB=postgres
         - POSTGRES_USER=postgres
         - POSTGRES_PASSWORD=postgres
     mysql:
       image: debezium/example-mysql:1.1
       ports:
         - "3306:3306"
       environment:
         - MYSQL_ROOT_PASSWORD=123456
         - MYSQL_USER=mysqluser
         - MYSQL_PASSWORD=mysqlpw
     elasticsearch:
       image: elastic/elasticsearch:7.6.0
       environment:
         - cluster.name=docker-cluster
         - bootstrap.memory_lock=true
         - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
         - discovery.type=single-node
       ports:
         - "9200:9200"
         - "9300:9300"
       ulimits:
         memlock:
           soft: -1
           hard: -1
         nofile:
           soft: 65536
           hard: 65536
     kibana:
       image: elastic/kibana:7.6.0
       ports:
         - "5601:5601"
     zookeeper:
       image: wurstmeister/zookeeper:3.4.6
       ports:
         - "2181:2181"
     kafka:
       image: wurstmeister/kafka:2.12-2.2.1
       ports:
         - "9092:9092"
         - "9094:9094"
       depends_on:
         - zookeeper
       environment:
         - KAFKA_ADVERTISED_LISTENERS=INSIDE://:9094,OUTSIDE://localhost:9092
         - KAFKA_LISTENERS=INSIDE://:9094,OUTSIDE://:9092
         - KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
         - KAFKA_INTER_BROKER_LISTENER_NAME=INSIDE
         - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
         - KAFKA_CREATE_TOPICS="user_behavior:1:1"
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock
   ```

   

2. 进入 mysql 容器，初始化数据：

   ```sh
   docker-compose exec mysql mysql -uroot -p123456
   ```

   ```sql
   -- MySQL
   CREATE DATABASE mydb;
   USE mydb;
   CREATE TABLE products (
     id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(255) NOT NULL,
     description VARCHAR(512)
   );
   ALTER TABLE products AUTO_INCREMENT = 101;
   
   INSERT INTO products
   VALUES (default,"scooter","Small 2-wheel scooter"),
          (default,"car battery","12V car battery"),
          (default,"12-pack drill bits","12-pack of drill bits with sizes ranging from #40 to #3"),
          (default,"hammer","12oz carpenter's hammer"),
          (default,"hammer","14oz carpenter's hammer"),
          (default,"hammer","16oz carpenter's hammer"),
          (default,"rocks","box of assorted rocks"),
          (default,"jacket","water resistent black wind breaker"),
          (default,"spare tire","24 inch spare tire");
   
   CREATE TABLE orders (
     order_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
     order_date DATETIME NOT NULL,
     customer_name VARCHAR(255) NOT NULL,
     price DECIMAL(10, 5) NOT NULL,
     product_id INTEGER NOT NULL,
     order_status BOOLEAN NOT NULL -- 是否下单
   ) AUTO_INCREMENT = 10001;
   
   INSERT INTO orders
   VALUES (default, '2020-07-30 10:08:22', 'Jark', 50.50, 102, false),
          (default, '2020-07-30 10:11:09', 'Sally', 15.00, 105, false),
          (default, '2020-07-30 12:00:30', 'Edward', 25.25, 106, false);
   
   
   ```

   

3. 进入postgres 容器，初始化数据：

   ```sh
   docker-compose exec postgres psql -h localhost -U postgres
   ```

   ```sql
   -- PG
   CREATE TABLE shipments (
     shipment_id SERIAL NOT NULL PRIMARY KEY,
     order_id SERIAL NOT NULL,
     origin VARCHAR(255) NOT NULL,
     destination VARCHAR(255) NOT NULL,
     is_arrived BOOLEAN NOT NULL
   );
   ALTER SEQUENCE public.shipments_shipment_id_seq RESTART WITH 1001;
   ALTER TABLE public.shipments REPLICA IDENTITY FULL;
   
   INSERT INTO shipments
   VALUES (default,10001,'Beijing','Shanghai',false),
          (default,10002,'Hangzhou','Shanghai',false),
          (default,10003,'Shanghai','Hangzhou',false);
   
   ```

   

4. 下载以下 jar 包到 `<FLINK_HOME>/lib/`:

- [flink-sql-connector-elasticsearch7_2.11-1.11.1.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-elasticsearch7_2.11/1.11.1/flink-sql-connector-elasticsearch7_2.11-1.11.1.jar)
- [flink-sql-connector-mysql-cdc-1.0.0.jar](https://repo1.maven.org/maven2/com/alibaba/ververica/flink-sql-connector-mysql-cdc/1.0.0/flink-sql-connector-mysql-cdc-1.0.0.jar)
- [flink-sql-connector-postgres-cdc-1.0.0.jar](https://repo1.maven.org/maven2/com/alibaba/ververica/flink-sql-connector-postgres-cdc/1.0.0/flink-sql-connector-postgres-cdc-1.0.0.jar)

5. 然后启动 Flink 集群，再启动 SQL CLI.

   ```sh
   --FlinkSQL
   CREATE TABLE products (
     id INT,
     name STRING,
     description STRING
   ) WITH (
     'connector' = 'mysql-cdc',
     'hostname' = 'localhost',
     'port' = '3306',
     'username' = 'root',
     'password' = '123456',
     'database-name' = 'mydb',
     'table-name' = 'products'
   );
   
   CREATE TABLE orders (
     order_id INT,
     order_date TIMESTAMP(0),
     customer_name STRING,
     price DECIMAL(10, 5),
     product_id INT,
     order_status BOOLEAN
   ) WITH (
     'connector' = 'mysql-cdc',
     'hostname' = 'localhost',
     'port' = '3306',
     'username' = 'root',
     'password' = '123456',
     'database-name' = 'mydb',
     'table-name' = 'orders'
   );
   
   CREATE TABLE shipments (
     shipment_id INT,
     order_id INT,
     origin STRING,
     destination STRING,
     is_arrived BOOLEAN
   ) WITH (
     'connector' = 'postgres-cdc',
     'hostname' = 'localhost',
     'port' = '5432',
     'username' = 'postgres',
     'password' = 'postgres',
     'database-name' = 'postgres',
     'schema-name' = 'public',
     'table-name' = 'shipments'
   );
   
   CREATE TABLE enriched_orders (
     order_id INT,
     order_date TIMESTAMP(0),
     customer_name STRING,
     price DECIMAL(10, 5),
     product_id INT,
     order_status BOOLEAN,
     product_name STRING,
     product_description STRING,
     shipment_id INT,
     origin STRING,
     destination STRING,
     is_arrived BOOLEAN,
     PRIMARY KEY (order_id) NOT ENFORCED
   ) WITH (
       'connector' = 'elasticsearch-7',
       'hosts' = 'http://localhost:9200',
       'index' = 'enriched_orders'
   );
   
   INSERT INTO enriched_orders
   SELECT o.*, p.name, p.description, s.shipment_id, s.origin, s.destination, s.is_arrived
   FROM orders AS o
   LEFT JOIN products AS p ON o.product_id = p.id
   LEFT JOIN shipments AS s ON o.order_id = s.order_id;
   
   ```

   

6. 修改 mysql 和 postgres 里面的数据，观察 elasticsearch 里的结果。

```sh
--MySQL
INSERT INTO orders
VALUES (default, '2020-07-30 15:22:00', 'Jark', 29.71, 104, false);

--PG
INSERT INTO shipments
VALUES (default,10004,'Shanghai','Beijing',false);

--MySQL
UPDATE orders SET order_status = true WHERE order_id = 10004;

--PG
UPDATE shipments SET is_arrived = true WHERE shipment_id = 1004;

--MySQL
DELETE FROM orders WHERE order_id = 10004;

```

7. Kafka changelog json format

```sql
--Flink SQL
CREATE TABLE kafka_gmv (
  day_str STRING,
  gmv DECIMAL(10, 5)
) WITH (
    'connector' = 'kafka',
    'topic' = 'kafka_gmv',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'localhost:9092',
    'format' = 'changelog-json'
);

INSERT INTO kafka_gmv
SELECT DATE_FORMAT(order_date, 'yyyy-MM-dd') as day_str, SUM(price) as gmv
FROM orders
WHERE order_status = true
GROUP BY DATE_FORMAT(order_date, 'yyyy-MM-dd');

-- 读取 Kafka 的 changelog 数据，观察 materialize 后的结果
SELECT * FROM kafka_gmv;

```

```sh
docker-compose exec kafka bash -c 'kafka-console-consumer.sh --topic kafka_gmv --bootstrap-server kafka:9094 --from-beginning'

```

更新 orders 数据，观察SQL CLI 和 kafka console 的输出

```sh
-- MySQL
UPDATE orders SET order_status = true WHERE order_id = 10001;
UPDATE orders SET order_status = true WHERE order_id = 10002;
UPDATE orders SET order_status = true WHERE order_id = 10003;

INSERT INTO orders
VALUES (default, '2020-07-30 17:33:00', 'Timo', 50.00, 104, true);

UPDATE orders SET price = 40.00 WHERE order_id = 10005;

DELETE FROM orders WHERE order_id = 10005;

```



参考文章：https://github.com/ververica/flink-cdc-connectors/wiki/中文教程



## 6.flink中文社区

https://flink-learning.org.cn/