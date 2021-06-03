---
description: In JSON format
---

# JSON

Below is the example of a default configuration file in JSON format that needs to serve the model in Konduit-Serving. This configuration file may need simple editing to set up your setup, especially in Pipeline Steps.

Example default Inference Configuration with Sequence Pipeline Steps:

```text
{
  "host" : "localhost",
  "port" : 0,
  "useSsl" : false,
  "protocol" : "HTTP",
  "staticContentRoot" : "static-content",
  "staticContentUrl" : "/static-content",
  "staticContentIndexPage" : "/index.html",
  "kafkaConfiguration" : {
    "startHttpServerForKafka" : true,
    "httpKafkaHost" : "localhost",
    "httpKafkaPort" : 0,
    "consumerTopicName" : "inference-in",
    "consumerKeyDeserializerClass" : "io.vertx.kafka.client.serialization.JsonObjectDeserializer",
    "consumerValueDeserializerClass" : "io.vertx.kafka.client.serialization.JsonObjectDeserializer",
    "consumerGroupId" : "konduit-serving-consumer-group",
    "consumerAutoOffsetReset" : "earliest",
    "consumerAutoCommit" : "true",
    "producerTopicName" : "inference-out",
    "producerKeySerializerClass" : "io.vertx.kafka.client.serialization.JsonObjectSerializer",
    "producerValueSerializerClass" : "io.vertx.kafka.client.serialization.JsonObjectSerializer",
    "producerAcks" : "1"
  },
  "mqttConfiguration" : { },
  "customEndpoints" : [ ],
  "pipeline" : {
    "steps" : [ {
      "@type" : "DEEPLEARNING4J",
      "modelUri" : "<path_to_model>",
      "inputNames" : [ "1", "2" ],
      "outputNames" : [ "11", "22" ]
    }, {
      "@type" : "LOGGING",
      "logLevel" : "INFO",
      "log" : "KEYS_AND_VALUES"
    } ]
  }
}
```

Example default Inference Configuration with Graph Pipeline Steps:

```text
{
  "host" : "localhost",
  "port" : 0,
  "useSsl" : false,
  "protocol" : "HTTP",
  "staticContentRoot" : "static-content",
  "staticContentUrl" : "/static-content",
  "staticContentIndexPage" : "/index.html",
  "kafkaConfiguration" : {
    "startHttpServerForKafka" : true,
    "httpKafkaHost" : "localhost",
    "httpKafkaPort" : 0,
    "consumerTopicName" : "inference-in",
    "consumerKeyDeserializerClass" : "io.vertx.kafka.client.serialization.JsonObjectDeserializer",
    "consumerValueDeserializerClass" : "io.vertx.kafka.client.serialization.JsonObjectDeserializer",
    "consumerGroupId" : "konduit-serving-consumer-group",
    "consumerAutoOffsetReset" : "earliest",
    "consumerAutoCommit" : "true",
    "producerTopicName" : "inference-out",
    "producerKeySerializerClass" : "io.vertx.kafka.client.serialization.JsonObjectSerializer",
    "producerValueSerializerClass" : "io.vertx.kafka.client.serialization.JsonObjectSerializer",
    "producerAcks" : "1"
  },
  "mqttConfiguration" : { },
  "customEndpoints" : [ ],
  "pipeline" : {
    "outputStep" : "4",
    "steps" : {
      "1" : {
        "@type" : "LOGGING",
        "@input" : "input",
        "logLevel" : "INFO",
        "log" : "KEYS_AND_VALUES"
      },
      "2" : {
        "@type" : "TENSORFLOW",
        "@input" : "1",
        "inputNames" : [ "1", "2" ],
        "outputNames" : [ "11", "22" ],
        "modelUri" : "<path_to_model>"
      },
      "3" : {
        "@type" : "DEEPLEARNING4J",
        "@input" : "1",
        "modelUri" : "<path_to_model>",
        "inputNames" : [ "1", "2" ],
        "outputNames" : [ "11", "22" ]
      },
      "4" : {
        "@type" : "MERGE",
        "@input" : [ "2", "3" ]
      }
    }
  }
}
```

For more details on how to create the configuration file, please refer to the examples:

{% page-ref page="../examples/cli/use-cases/creating-a-sequence-pipeline.md" %}

{% page-ref page="../examples/cli/use-cases/creating-a-graph-pipeline.md" %}

