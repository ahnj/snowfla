---
title: "Kafka Connect auto.offset.reset"
date: 2020-11-01T09:09:54-07:00
draft: true

description: "kafka retention data in reverse"
featured_image: "/images/reverse.png"
tags: ["kafka connect", "auto.offset.reset", "offset management"]
---

I'm a big fan of Robin Moffatt's writings and speaking content.  I owe him a beer or two from all I have learned just from his prolific work on Kafka Connect.  However, when I first encountered his [article](https://rmoff.net/2019/08/09/starting-a-kafka-connect-sink-connector-at-the-end-of-a-topic/) about configuring a sink to play a topic from the the end, rather than from the beginning, I didn't understand why on earth anyone would want to do so.  Well, now I do and let me share one possible scenario where this could save the day.  

As you already know, Kafka does not guarantee order.  But it does guarantee [exactly-once semantics](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/).  This means that when a sink that's newly born or resurrected from a long winter hibernation, it might take a really long time to consume and work off all the lagged offsets down to zero.  This is especially true when there's a topic with a very large retention size.  This is problematic if your sink destination is relatively a slower ingest destination like Snowflake (for example, we observed maximum insertion rate of ~20 million kafka records per hour with a `task.max` set to value `1`.  and relatively small record payload size).  Therefore, your lag might take multiple hours or even days to fully get ingested into Snowflake.  This means the latest data being published into the topic will get stuck in the back of the msg queue.  This is bad news for downstream processes who goes starved of recent data while this ingestion is being resolved.  Like in the old CFA days, the default Kafka Connect sink behavior is first-in-first out, FIFO. However, by setting the sink configuration value `consumer.override.auto.offset.reset` to `latest`, sink's behavior is modified to last-in-first-out, LIFO. 

Below is an example Snowflake sink connector configuration set to LIFO behavior:
```
{
  "consumer.override.auto.offset.reset": "latest",
  "name": "snowflake-sink",
  "value.converter.schema.registry.url": "http://schemaregistry.company.com/",
  "connector.class": "com.snowflake.kafka.connector.SnowflakeSinkConnector",
  "tasks.max": "1",
  "key.converter": "org.apache.kafka.connect.storage.StringConverter",
  "value.converter": "com.snowflake.kafka.connector.records.SnowflakeAvroConverter",
  "errors.tolerance": "all",
  "errors.log.enable": "true",
  "errors.log.include.messages": "true",
  "topics": ["topic_with_super_long_retention_period"],
  "errors.deadletterqueue.topic.name": "event-some-data-snowflake-dlt",
  "errors.deadletterqueue.topic.replication.factor": "3",
  "errors.deadletterqueue.context.headers.enable": "true",
  "snowflake.url.name": "https://*******.snowflakecomputing.com/",
  "snowflake.user.name": "SNOWFLAKE_CONNECT_USER",
  "snowflake.private.key": "*******",
  "snowflake.database.name": "SF_KAFKA_INGEST_DB",
  "snowflake.schema.name": "KAFKA_SCHEMA",
  "snowflake.topic2table.map": "topic_with_super_big_retention_period:twsbrp_table",
  "buffer.flush.time": "10"
}
```

and environment variable set to:

```
CONNECT_CONNECTOR_CLIENT_CONFIG_OVERRIDE_POLICY=All
```

 Kafka now allows infinate retention period, this might even be a more commonly recurring issue for those Connects in the wild.

