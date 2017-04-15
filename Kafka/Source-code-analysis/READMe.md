# Kafka Source Code Analysis

## Mind Map

![mind map](./Kafka-mind.png)

## `kafka.scala.Kafka#main()`

```
def main(args: Array[String]): Unit = {
    try {
      // 读取配置文件内容到Properties对象
      val serverProps = getPropsFromArgs(args)
      // 创建KafkaServerStartable对象
      val kafkaServerStartable = KafkaServerStartable.fromProps(serverProps)

      // attach shutdown handler to catch control-c
      Runtime.getRuntime().addShutdownHook(new Thread("kafka-shutdown-hook") {
        override def run(): Unit = kafkaServerStartable.shutdown()
      })

      kafkaServerStartable.startup()
      kafkaServerStartable.awaitShutdown()
    }
    catch {
      case e: Throwable =>
        fatal(e)
        Exit.exit(1)
    }
    Exit.exit(0)
  }
```

## KafkaScheduler
1. Initialize a thread pool
2. Provide a method `schedule()` to create and execute one-shot or periodic actions