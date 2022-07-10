



bin/kafka-topics.sh --zookeeper localhost:2181 --list



bin/kafka-console-consumer.sh --bootstrap-server lyxbdw-01:9092 --topic topic0619



## kafka 常用命令kafka-console-consumer.sh

kafka-console-consumer.sh 脚本是一个简易的消费者控制台。该 shell 脚本的功能通过调用 kafka.tools 包下的 `ConsoleConsumer` 类，并将提供的命令行参数全部传给该类实现。

### 1.手动插入数据

```sh
kafka-console-producer.sh --broker-list 192.168.104.91:9092,192.168.104.92:9092,192.168.104.93:9092 --topic capture_test
```

### 2. 消息消费

```sh
bin/kafka-console-consumer.sh --bootstrap-server 192.168.104.91:9092,192.168.104.92:9092,192.168.104.93:9092 --topic topicName
bin/kafka-console-consumer.sh --zookeeper 192.168.104.91:2181,192.168.104.92:2181,192.168.104.93:2181 --topic topicName
表示从 latest 位移位置开始消费该主题的所有分区消息，即仅消费正在写入的消息。
```

### 3.从开始位置消费

```sh
bin/kafka-console-consumer.sh --bootstrap-server 192.168.104.91:9092,192.168.104.92:9092,192.168.104.93:9092 --from-beginning --topic topicName
bin/kafka-console-consumer.sh --zookeeper 192.168.104.91:2181,192.168.104.92:2181,192.168.104.93:2181 --from-beginning --topic topicName
```

### 4.显示key消费

```sh
bin/kafka-console-consumer.sh --bootstrap-server 192.168.104.91:9092,192.168.104.92:9092,192.168.104.93:9092 --property print.key=true --topic topicName
bin/kafka-console-consumer.sh --zookeeper 192.168.104.91:2181,192.168.104.92:2181,192.168.104.93:2181 --property print.key=true  --topic topicName
消费出的消息结果将打印出消息体的 key 和 value。
```

| 参数                      | 值类型  | 说明                                                         | 有效值                                                       |
| ------------------------- | :-----: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| --topic                   | String  | 被消费的topic                                                |                                                              |
| --whitelist               | string  | 正则表达式，指定要包含以供使用的主题的白名单                 |                                                              |
| --partition               | integer | 指定分区 除非指定’–offset’，否则从分区结束(latest)开始消费   |                                                              |
| --offset                  | string  | 执行消费的起始offset位置 默认值:latest                       | latest earliest <offset>                                     |
| --consumer-property       | string  | 将用户定义的属性以key=value的形式传递给使用者                |                                                              |
| --consumer.config         | string  | 消费者配置属性文件 请注意，[consumer-property]优先于此配置   |                                                              |
| --formatter               | string  | 用于格式化kafka消息以供显示的类的名称 默认值:kafka.tools.DefaultMessageFormatter | kafka.tools.DefaultMessageFormatter kafka.tools.LoggingMessageFormatter kafka.tools.NoOpMessageFormatter kafka.tools.ChecksumMessageFormatter |
| --property                | string  | 初始化消息格式化程序的属性                                   | print.timestamp=true\|false print.key=true\|false print.value=true\|false key.separator=<key.separator> line.separator=<line.separator> key.deserializer=<key.deserializer> value.deserializer=<value.deserializer> |
| --from-beginning          |         | 从存在的最早消息开始，而不是从最新消息开始                   |                                                              |
| --max-messages            | integer | 消费的最大数据量，若不指定，则持续消费下去                   |                                                              |
| --timeout-ms              | integer | 在指定时间间隔内没有消息可用时退出                           |                                                              |
| --skip-message-on-error   |         | 如果处理消息时出错，请跳过它而不是暂停                       |                                                              |
| --bootstrap-server        | string  | 必需(除非使用旧版本的消费者)，要连接的服务器                 |                                                              |
| --key-deserializer        | string  |                                                              |                                                              |
| --value-deserializer      | string  |                                                              |                                                              |
| --enable-systest-events   |         | 除记录消费的消息外，还记录消费者的生命周期 (用于系统测试)    |                                                              |
| --isolation-level         | string  | 设置为read_committed以过滤掉未提交的事务性消息 设置为read_uncommitted以读取所有消息 默认值:read_uncommitted |                                                              |
| --group                   | string  | 指定消费者所属组的ID                                         |                                                              |
| --blacklist               | string  | 要从消费中排除的主题黑名单                                   |                                                              |
| --csv-reporter-enabled    |         | 如果设置，将启用csv metrics报告器                            |                                                              |
| --delete-consumer-offsets |         | 如果指定，则启动时删除zookeeper中的消费者信息                |                                                              |
| --metrics-dir             | string  | 输出csv度量值 需与[csv-reporter-enable]配合使用              |                                                              |
| --zookeeper               | string  | 必需(仅当使用旧的使用者时)连接zookeeper的字符串。 可以给出多个URL以允许故障转移 |                                                              |











































