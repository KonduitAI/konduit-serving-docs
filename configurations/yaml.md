---
description: In YAML format
---

# YAML

Apart from JSON, the configuration file also can be in YAML format. It is identical to JSON but more human-readable data representation and pretty straightforward. Difference to JSON, YAML's hierarchy is denoted by using double space characters as an example below.

```text
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
  - '@type': "DEEPLEARNING4J"
    modelUri: "<path_to_model>"
    inputNames:
    - "1"
    - "2"
    outputNames:
    - "11"
    - "22"
  - '@type': "LOGGING"
    logLevel: "INFO"
    log: "KEYS_AND_VALUES"
```

Example of default YAML configuration file with custom Graph Pipeline Steps: 

```text
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
  outputStep: "4"
  steps:
    "1":
      '@type': "LOGGING"
      '@input': "input"
      logLevel: "INFO"
      log: "KEYS_AND_VALUES"
    "2":
      '@type': "TENSORFLOW"
      '@input': "1"
      input_names:
      - "1"
      - "2"
      output_names:
      - "11"
      - "22"
      model_uri: "<path_to_model>"
    "3":
      '@type': "DEEPLEARNING4J"
      '@input': "1"
      modelUri: "<path_to_model>"
      inputNames:
      - "1"
      - "2"
      outputNames:
      - "11"
      - "22"
    "4":
      '@type': "MERGE"
      '@input':
      - "2"
      - "3"
```

For more details on how to create the configuration file, please refer to the examples:

{% page-ref page="../examples/cli/use-cases/creating-a-sequence-pipeline.md" %}

{% page-ref page="../examples/cli/use-cases/creating-a-graph-pipeline.md" %}

