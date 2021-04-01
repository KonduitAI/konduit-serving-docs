---
description: Example of sequence pipeline
---

# Sequence Pipeline

In this example, you'll create the configuration for a server. The same as the previous example, you'll print the configuration to demonstrate the step in details that will help you see the difference and notice the configuration contents.

* Let's create a configuration by adding logging step in a sequence pipeline and print the output to JSON:

```java
SequencePipeline sequencePipelineWithLoggingStep = SequencePipeline
                .builder()
                .add(new LoggingStep()) //add logging step into pipeline
                .build();
                
System.out.format("----------%n" +
                        "Pipeline with a Logging step output%n" +
                        "------------%n" +
                        "%s%n" +
                        "------------%n%n",
                sequencePipelineWithLoggingStep.toJson());
```

* Call for default or empty inference configuration and print the output to JSON:

```java
InferenceConfiguration defaultInferenceConfiguration = new InferenceConfiguration();

        System.out.format("----------%n" +
                        "Default inference configuration%n" +
                        "------------%n" +
                        "%s%n" +
                        "------------%n%n",
                defaultInferenceConfiguration.toJson());
```

* Combine both to create complete configuration for a server:

```java
InferenceConfiguration inferenceConfigurationWithPipeline = new InferenceConfiguration();
        inferenceConfigurationWithPipeline.pipeline(sequencePipelineWithLoggingStep);

        // Printing InferenceConfiguration in YAML
        System.out.format("----------%n" +
                        "Inference Configuration in YAML%n" +
                        "------------%n" +
                        "%s%n" +
                        "------------%n%n",
                inferenceConfigurationWithPipeline.toYaml());
```

You'll see the output similar to:

```aspnet
----------
Pipeline with a Logging step output
------------
{
  "steps" : [ {
    "@type" : "LOGGING",
    "logLevel" : "INFO",
    "log" : "KEYS"
  } ]
}
------------

----------
Default inference configuration
------------
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
  "customEndpoints" : [ ]
}
------------

----------
Inference Configuration in YAML
------------
---
host: "localhost"
port: 0
use_ssl: false
protocol: "HTTP"
static_content_root: "static-content"
static_content_url: "/static-content"
static_content_index_page: "/index.html"
kafka_configuration:
  start_http_server_for_kafka: true
  http_kafka_host: "localhost"
  http_kafka_port: 0
  consumer_topic_name: "inference-in"
  consumer_key_deserializer_class: "io.vertx.kafka.client.serialization.JsonObjectDeserializer"
  consumer_value_deserializer_class: "io.vertx.kafka.client.serialization.JsonObjectDeserializer"
  consumer_group_id: "konduit-serving-consumer-group"
  consumer_auto_offset_reset: "earliest"
  consumer_auto_commit: "true"
  producer_topic_name: "inference-out"
  producer_key_serializer_class: "io.vertx.kafka.client.serialization.JsonObjectSerializer"
  producer_value_serializer_class: "io.vertx.kafka.client.serialization.JsonObjectSerializer"
  producer_acks: "1"
mqtt_configuration: {}
custom_endpoints: []
pipeline:
  steps:
  - '@type': "LOGGING"
    logLevel: "INFO"
    log: "KEYS"

------------


Process finished with exit code 0
```

