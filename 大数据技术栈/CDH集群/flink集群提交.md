```java
 System.setProperty("java.security.auth.login.config","./conf/jaas.conf");
        System.setProperty("java.security.krb5.conf", "./conf/krb5.conf");

 System.setProperty("java.security.auth.login.config","E:\\WorkSpace\\jaas.conf");   
System.setProperty("java.security.krb5.conf", "E:\\WorkSpace\\krb5.conf");
        StreamExecutionEnvironment env = InitFlinkUtils.getEnv();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.setParallelism(1);

        //1.读取数据源kafka

        Properties properties = new Properties();

        ParameterTool parameterTool = ParameterTool.fromArgs(args);
        String paramProperties = parameterTool.get("paramProperties");

        ///加载类路径
        try {
            properties.load(new FileInputStream(paramProperties));
        } catch (IOException e) {
            e.printStackTrace();
        }

///        Properties kafkaProperties = new Properties();
//        kafkaProperties.setProperty("bootstrap.servers",properties.getProperty("bootstrap.servers"));
//        kafkaProperties.setProperty("group.id",properties.getProperty("group.id"));
//        kafkaProperties.setProperty("topic",properties.getProperty("topic"));
//        kafkaProperties.setProperty("security.protocol",properties.getProperty("security.protocol"));
//        kafkaProperties.setProperty("sasl.kerberos.service.name",properties.getProperty("sasl.kerberos.service.name"));


//        FlinkKafkaConsumer<String> kafkaTopicBluetooth =
//                new FlinkKafkaConsumer<>(properties.getProperty("topic"), new SimpleStringSchema(), kafkaProperties);
//
//        kafkaTopicBluetooth.setStartFromEarliest();
//
//        DataStreamSource<String> sourceStream = env.addSource(kafkaTopicBluetooth);

        //=========================== 通过socket 模拟kafka数据 ===================

        DataStreamSource<String> sourceStream = env.socketTextStream("localhost", 8888);



        //2.数据处理

        //将数据转成信标的坐标实体
        SingleOutputStreamOperator<BaseStationLocationInfo> labelLocation = sourceStream.map(labelLocationString -> {
            BaseStationLocationInfo baseStationLocationInfo = new BaseStationLocationInfo();
            try {
                Gson gson = new Gson();
                baseStationLocationInfo = gson.fromJson(labelLocationString, BaseStationLocationInfo.class);
            } catch (JsonSyntaxException e) {
                e.printStackTrace();
            }
            return baseStationLocationInfo;
        });

        labelLocation.print();
        //通过滚动窗口，按照信标分组，计算出10秒中信号强度最大的基站对应的数据

        SingleOutputStreamOperator<LabelLocation> labelLocationSingleOutputStreamOperator = labelLocation.
                assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<BaseStationLocationInfo>
                        (Time.seconds(20)) {
            @Override
            public long extractTimestamp(BaseStationLocationInfo baseStationLocationInfo) {
                return Timestamp.valueOf(baseStationLocationInfo.getRecDatetime()).getTime();
            }
        }).keyBy(BaseStationLocationInfo::getLabelId)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                .process(new LabelTimeLocationProcessWindow());

        //关联基站的基础信息数据

//        SingleOutputStreamOperator<BaseStationAndLabel> baseStationAndLabel =
//                labelLocationSingleOutputStreamOperator.map(new BaseStationLocationInfoMap());
//
//
//        //计算信标在基站停留的时间，每换一个基站，记录上个基站的时间
//
//        SingleOutputStreamOperator<LabelStayTimeInfo> labelStayTimeInfoSingleOutputStreamOperator =
//                labelLocationSingleOutputStreamOperator
//                .keyBy(LabelLocation::getLabelId)
//                .flatMap(new LabelStayTimeFlatMap());


        //3.sink

```

```sh
security.kerberos.login.use-ticket-cache: false
security.kerberos.login.keytab: ./conf/260164.keytab
security.kerberos.login.principal: 260164@GREE.IO
security.kerberos.login.contexts: Client,KafkaClient
```

```sh
#!/bin/bash
env -i FLINK_CONF_DIR=./conf /lvm/data3/flink/flink-1.8.1/bin/flink run -m yarn-cluster -yn 4 -ytm 2g -ys 4 -yqu root.dev1 -ynm person-location   ./location-data-analysis.jar --paramProperties ./conf/location-param-test.properties
```

linux 上 jaas.conf

```properties
KafkaClient {  
com.sun.security.auth.module.Krb5LoginModule required  
doNotPrompt=true  
useKeyTab=true  
storeKey=false  
renewTicket=true 
keyTab="./conf/260164.keytab"  
principal="260164@GREE.IO";
};
Client {  
com.sun.security.auth.module.Krb5LoginModule required  
useKeyTab=true 
storeKey=false
keyTab="./conf/260164.keytab"  
principal="260164@GREE.IO";
};
```

window jaas.conf

```properties
KafkaClient {  
com.sun.security.auth.module.Krb5LoginModule required  
doNotPrompt=true  
useKeyTab=true 
storeKey=true 
renewTicket=true 
keyTab="E:/WorkSpace/bluetooth-location-dynamics/location-data-analysis/src/main/resources/260164.keytab"  
principal="260164@GREE.IO";
};
Client {  
com.sun.security.auth.module.Krb5LoginModule required  
useKeyTab=true 
storeKey=true 
keyTab="E:/WorkSpace/bluetooth-location-dynamics/location-data-analysis/src/main/resources/260164.keytab"  
principal="260164@GREE.IO";
};
```





```properties
bootstrap.servers=kafka01:30001,kafka02:30002,kafka03:30003,kafka04:30004,kafka05:30005,kafka06:30006,kafka07:30007,kafka08:30008,kafka09:30009
group.id=bluetooth-person-location440
auto.offset.reset=earliesttopic=bluetooth-person-location
security.protocol=SASL_PLAINTEXT
sasl.kerberos.service.name=kafka
application.name=people-location-analysis
minimumResidenceTimes=0.5
redis.host=10.2.21.71
redis.password=123456
```



## spark 写入kafka

```scala
wxgWangdianInfo 
.toJSON 
.selectExpr("CAST(value as STRING)")  
.write
.format("kafka")      .option("kafka.bootstrap.servers","kafka01:30001,kafka02:30002,kafka03:30003,kafka04:30004,kafka05:30005,kafka06:30006,kafka07:30007,kafka08:30008,kafka09:30009")  
.option("topic","dispatchInstaller")  
.option("kafka.security.protocol","SASL_PLAINTEXT" ) 
.option("kafka.sasl.kerberos.service.name","kafka" )  
.save()
```





