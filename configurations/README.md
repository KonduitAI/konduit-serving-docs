---
description: Konduit-Serving supports defining server configurations as JSON or YAML files.
---

# Configurations

The configuration is essential to serve a Machine Learning or Deep Learning model in Konduit-Serving. The complete format contains inference configuration and pipeline steps to deploy the server to serve the model in details.

A Konduit-Serving configuration file has two top-level items: 

1. Inference configuration
2. Pipeline

## Inference Configuration

A sample below is the inference configuration, which contains many setups for the server to run on Konduit-Serving. For explanation purpose, the YAML format is used like the following.

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
```

As the setting on the above, there are a lot of keys used in the inference configuration. The inference configuration takes the following arguments:

* `host` : specify the port number
* `port` : the host of the Konduit-Serving. Default is 'localhost'.
* `use_ssl` : Enable SSL for internet connection security data protection. Default is 'false'.
* `protocol` : protocol use with the server. Default is 'HTTP'.
* `static_content_root` : root directory use to search for a folder in serving static content
* `static_content_url` : URL that will link to the static contents specified in the `static_content_root` __key.
* `static_content_index_page` : index file name. Default is 'index.html'.
* `kafka_configuration` : Configuration for Kafka message queue when `protocol` key is `KAFKA`
* `mqtt_configuration` : configuration if 'MQTT' is used as protocol.
* `custom_endpoints` : Custom created endpoints by implementing the `ai.konduit.serving.endpoint.HttpEndpoints` interface.

This setting can be generated using the `CLI` command, and generally, this configuration file is a must to deploy any Pipeline Steps on Konduit-Serving. 

## Pipeline

A pipeline consists of steps on how the server should treat the data and the model. We can use many steps in the pipeline to serve the model, including the input pre-processing step and output post-processing step. The simple application makes Konduit-Serving more convenient for all level experience to use our platform. Below is the example of Pipeline Steps which only consists of serving the model and do post-processing by classifying the output product from the last layer of the model.

```text
pipeline:
  steps:
  - '@type': "DEEPLEARNING4J"
    modelUri: "dl4j_iris_model.zip"
    inputNames:
    - "input"
    outputNames:
    - "output"
  - '@type': "CLASSIFIER_OUTPUT"
    input_names: "layer2"
    labels:
      - Sentosa
      - Versicolor
      - Virginica
```

These steps can be added as many as possible based on the requirement for custom endpoints. Among the steps that can be used in this pipeline are:

* Sequences pipeline steps:
  * crop\_grid
  * crop\_fixed_\__grip
  * dl4j \(use in above example\)
  * keras
  * draw\_bounding\_box
  * draw\_fixed_\__grid
  * draw\_segmentation
  * extract\_bounding\_box
  * camera\_frame\_capture
  * video\_frame\_capture
  * image\_to\_ndarray
  * logging
  * ssd\_to\_bounding\_box
  * samediff
  * show\_image
  * tensorflow
  * nd4jtensorflow
  * python
  * onnx
  * classifier\_output \(use in above example\)
* Graphs pipeline steps:
  * pipeline steps
  * merge step
  * switch step
  * any step

The complete inference configuration files can be in:

{% page-ref page="json.md" %}

or

{% page-ref page="yaml.md" %}

