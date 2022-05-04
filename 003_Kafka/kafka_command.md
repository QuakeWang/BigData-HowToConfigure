# KAFKA 常用命令



### （1）查看 Kafka Topic 列表

`[quakewang@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181/kafka --list`

### （2）创建 Kafka Topic

进入到 /opt/module/kafka/ 目录下创建日志主题

`[quakewang@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --replication-factor 2 --partitions 3 --topic topic_log`

### （3）阐述 Kafka Topic

`[quakewang@hadoop102 kafka]$ bin/kafka-topics.sh --delete --bootstrap-server hadoop102:9092 --topic topic_log`

### （4）Kafka 生产消息

```bash
[quakewang@hadoop102 kafka]$ bin/kafka-console-producer.sh \
--broker-list hadoop102:9092 --topic topic_log
>hello world
>apache kafka
```

### （5）Kafka 消费消息

```bash
[quakewang@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
--bootstrap-server hadoop102:9092 --from-beginning --topic topic_log 
```

--from-beginning：会把主题中以往所有的数据都读取出来。根据业务场景选择是否增加该配置。

### （6） 查看 Kafka Topic 详情

```bash
[quakewang@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181/kafka \
--describe --topic topic_log
```



