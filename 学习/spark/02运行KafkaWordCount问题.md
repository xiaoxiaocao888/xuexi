
| **问题现象​**​                                  | ​**​根本原因​**​                                 | ​**​解决方案​**​                                  |
| ------------------------------------------- | -------------------------------------------- | --------------------------------------------- |
| `NoClassDefFoundError: ByteArraySerializer` | Kafka 客户端依赖未正确加载（路径错误或版本冲突）                  | 硬链接 Ivy 缓存 JAR 到 Spark 路径，强制指定 `--jars` 参数    |
| `Failed to create new KafkaAdminClient`     | Kafka 连接配置错误（`bootstrap.servers` 地址无效或网络不可达） | 修正 `kafka.bootstrap.servers` 配置，确保 Kafka 服务可达 |
| `SLF4J: No providers` 日志警告                  | 日志框架依赖未正确初始化（次要问题，通常不影响运行）                   | 添加 `slf4j-simple` 依赖或忽略（生产环境需统一日志框架）          |

**代码配置**
```
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "lqh425:9092")  # ✅ 必须真实IP/端口
    .option("subscribe", "your_topic")                            # ✅ 已存在的 Topic
    .option("startingOffsets", "latest") \
    .load()
```

**spark提交命令**
```
/usr/local/spark/bin/spark-submit \
  --master local[2] \
  --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.0 \
  --jars "/usr/local/spark/jars/kafka-clients-3.3.2.jar,/usr/local/spark/jars/spark-sql-kafka-0-10_2.12-3.4.0.jar" \
  --conf "spark.sql.extensions=org.apache.spark.sql.kafka010.KafkaSparkExtensions" \
  --conf "spark.driver.extraClassPath=/usr/local/spark/jars/*:/tmp/.ivy/jars/*" \
  --conf "spark.executor.extraClassPath=/usr/local/spark/jars/*:/tmp/.ivy/jars/*" \
  --conf "spark.jars.ivy=/tmp/.ivy" \
  /usr/local/spark/mycode/streaming/kafka/KafkaWordCount.py
```

清理旧缓存依赖
```
rm -rf /tmp/.ivy/cache ~/.ivy2/cache
```

###### 启动kafaka
1.启动zookeeper服务
```
cd /usr/local/kafka
./bin/zookeeper-server-start.sh config/zookeeper.properties
```
2.打开第二个终端，启动kafka服务
```
 ./bin/kafka-server-start.sh  config/server.properties
```
3.使用kafka创建主题
```
./bin/kafka-topics.sh --create \
  --bootstrap-server lqh425:9092 \
  --replication-factor 1 --partitions 1 --topic zhutiming
```
4.创建生产者
```
./bin/kafka-console-producer.sh  --broker-list  lqh425:9092 \
>  --topic  zhutiming
```
5.创建消费者
```
./bin/kafka-console-consumer.sh  --  lqh425:9092  \
> --topic  zhutiming  --from-beginning
```