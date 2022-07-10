

# Flink 双流 Join

> Flink DataStream API 为用户提供了3个算子来实现双流 join，分别是：
>
> - join()
>
> - coGroup()
>
> - intervalJoin()
>
>   数据流
>
>   ```java
>     
>   DataStream<String> clickSourceStream = env
>     .addSource(new FlinkKafkaConsumer011<>(
>       "ods_analytics_access_log",
>       new SimpleStringSchema(),
>       kafkaProps
>     ).setStartFromLatest());
>   DataStream<String> orderSourceStream = env
>     .addSource(new FlinkKafkaConsumer011<>(
>       "ods_ms_order_done",
>       new SimpleStringSchema(),
>       kafkaProps
>     ).setStartFromLatest());
>     
>   DataStream<AnalyticsAccessLogRecord> clickRecordStream = clickSourceStream
>     .map(message -> JSON.parseObject(message, AnalyticsAccessLogRecord.class));
>   DataStream<OrderDoneLogRecord> orderRecordStream = orderSourceStream
>     .map(message -> JSON.parseObject(message, OrderDoneLogRecord.class));
>   ```

## 1. join算子

>  join() 算子提供的语义为"Window join"，即按照指定字段和（滚动/滑动/会话）窗口进行 inner join，支持处理时间和事件时间两种时间特征。
>
>以下示例以10秒滚动窗口，将两个流通过商品 ID 关联，取得订单流中的售价相关字段。
>
>![image-20201127160806918](flilnk%E5%8F%8C%E6%B5%81join.assets/image-20201127160806918.png)
>
>```java
>clickRecordStream
>  .join(orderRecordStream)
>  .where(record -> record.getMerchandiseId())
>  .equalTo(record -> record.getMerchandiseId())
>  .window(TumblingProcessingTimeWindows.of(Time.seconds(10)))
>  .apply(new JoinFunction<AnalyticsAccessLogRecord, OrderDoneLogRecord, String>() {
>    @Override
>    public String join(AnalyticsAccessLogRecord accessRecord, OrderDoneLogRecord orderRecord) throws Exception {
>      return StringUtils.join(Arrays.asList(
>        accessRecord.getMerchandiseId(),
>        orderRecord.getPrice(),
>        orderRecord.getCouponMoney(),
>        orderRecord.getRebateAmount()
>      ), '\t');
>    }
>  })
>  .print().setParallelism(1);
>```
>
>

## 2. coGroup()算子

> 只有 inner join 肯定还不够，如何实现 left/right outer join 呢？答案就是利用 coGroup() 算子。它的调用方式类似于 join() 算子，也需要开窗，但是 CoGroupFunction 比 JoinFunction 更加灵活，可以按照用户指定的逻辑匹配左流和/或右流的数据并输出。
>
> 以下的例子就实现了点击流 left join 订单流的功能，是很朴素的 nested loop join 思想（二重循环）。
>
> ```java
> clickRecordStream
>   .coGroup(orderRecordStream)
>   .where(record -> record.getMerchandiseId())
>   .equalTo(record -> record.getMerchandiseId())
>   .window(TumblingProcessingTimeWindows.of(Time.seconds(10)))
>   .apply(new CoGroupFunction<AnalyticsAccessLogRecord, OrderDoneLogRecord, Tuple2<String, Long>>() {
>     @Override
>     public void coGroup(Iterable<AnalyticsAccessLogRecord> accessRecords, Iterable<OrderDoneLogRecord> orderRecords, Collector<Tuple2<String, Long>> collector) throws Exception {
>       for (AnalyticsAccessLogRecord accessRecord : accessRecords) {
>         boolean isMatched = false;
>         for (OrderDoneLogRecord orderRecord : orderRecords) {
>           // 右流中有对应的记录
>           collector.collect(new Tuple2<>(accessRecord.getMerchandiseName(), orderRecord.getPrice()));
>           isMatched = true;
>         }
>         if (!isMatched) {
>           // 右流中没有对应的记录
>           collector.collect(new Tuple2<>(accessRecord.getMerchandiseName(), null));
>         }
>       }
>     }
>   })
>   .print().setParallelism(1);
> ```
>
> 

## 3.interval Join()算子

> join() 和 coGroup() 都是基于窗口做关联的。但是在某些情况下，两条流的数据步调未必一致。例如，订单流的数据有可能在点击流的购买动作发生之后很久才被写入，如果用窗口来圈定，很容易 join 不上。所以 Flink 又提供了"Interval join"的语义，按照指定字段以及右流相对左流偏移的时间区间进行关联，即：
>
> > right.timestamp ∈ [left.timestamp + lowerBound; left.timestamp + upperBound]
>
> ![image-20201127161724873](flilnk%E5%8F%8C%E6%B5%81join.assets/image-20201127161724873.png)
>
> interval join 也是 inner join，虽然不需要开窗，但是需要用户指定偏移区间的上下界，并且只支持事件时间。
>
> 示例代码如下。注意在运行之前，需要分别在两个流上应用 assignTimestampsAndWatermarks() 方法获取事件时间戳和水印。
>
> ```java
> 
> clickRecordStream
>   .keyBy(record -> record.getMerchandiseId())
>   .intervalJoin(orderRecordStream.keyBy(record -> record.getMerchandiseId()))
>   .between(Time.seconds(-30), Time.seconds(30))
>   .process(new ProcessJoinFunction<AnalyticsAccessLogRecord, OrderDoneLogRecord, String>() {
>     @Override
>     public void processElement(AnalyticsAccessLogRecord accessRecord, OrderDoneLogRecord orderRecord, Context context, Collector<String> collector) throws Exception {
>       collector.collect(StringUtils.join(Arrays.asList(
>         accessRecord.getMerchandiseId(),
>         orderRecord.getPrice(),
>         orderRecord.getCouponMoney(),
>         orderRecord.getRebateAmount()
>       ), '\t'));
>     }
>   })
>   .print().setParallelism(1);
> ```
>
> 







参考：1. [Flink双流join的3中操作示例](https://mp.weixin.qq.com/s?__biz=MzU3Mzg4OTMyNQ==&mid=2247489831&idx=1&sn=e11a7f228d824d6ff79160e82128714c&chksm=fd3b9765ca4c1e73fb9f40168db85a0ef3f835e847bbc28d016c08b828171a5a4e981d7168e4&scene=126&sessionid=1606437164&key=49bbee8ef4005d858d83627c06b54ff8483db160e049ee429a2be72944156164462f6c57828b9a7ea6ce9d28f7454701c5e4f43a732a7b5af5c101eae360a345aaf2f0d8b88282b7b6cec011790eda4d3e938e7ae272066ea81378e10ee4387a893eb32fb8f8336116c856b89f9239679d961738922b7f73ed7f2171d1c6c794&ascene=1&uin=MjE0OTczMjUzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=A1oWd0X8ReQU31s1eAq2JJU%3D&pass_ticket=EqTcxJXW%2F22CqP0v1PsKrgw6dTgvSo5t10vD8rlgy8Cl5soyEi4lhdX5E2bu%2FLVs&wx_header=0)

