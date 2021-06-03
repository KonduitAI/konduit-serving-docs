# Creating a Graph Pipeline

Unlike Sequence Pipeline, a Graph Pipeline gives us more control in managing the pipeline's flow in parallel of steps. Same as the previous example, we'll use the `konduit config` command to configure graph pipeline steps and show the configuration file. In this example, we only focus on the steps with default inference configuration to demonstrate the basis of managing steps.

There are five type of steps in configuring the Graph Pipeline:

1. Pipeline steps
2. Switch step \(string\)
3. Switch step \(int\)
4. Merge step
5. Any step

Konduit-Serving provides flexibility to use multiple models in a single Graph Pipeline. The pipeline should be in single quote format such `'<output>=<type>(<input>)'` or `'[output]=<type>(<input>)'` for both switches step. The input must be continuously related to the previous output assigned before. If not, the Switches step is used to channel the input through the separate models.

Let's generate a configuration that logs the input\(1\), then flow them through two different frameworks models\(2 for tensorflow, 3 for dl4j\) and merge the output\(4\). The configuration file then saves to the JSON format.

```text
$ konduit config --pipeline '1=logging(input),2=tensorflow(1),3=dl4j(1),4=merge(2,3)' --output config.json
```

The configuration is as the following.

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

Same as the previous example, all the steps still need to configure the setting based on the input, model configuration like input and output layer name and the path to the model's directory. You'll need to modify the configuration based on your setting.

Let's try another command example for multiple integer inputs for different kinds of models in YAML configuration file format. In this case, we'll use Switch Step \(int\) to separate and channel the inputs to TensorFlow model and  DL4J model.

```text
$ konduit config --pipeline '1=logging(input),[2_1,2_2]=switch(int,select,1),3=tensorflow(2_1),4=dl4j(2_2),5=any(3,4)' --output graph_config.yaml --yaml
```

The YAML configuration file generated from above command.

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
  outputStep: "5"
  steps:
    "1":
      '@type': "LOGGING"
      '@input': "input"
      logLevel: "INFO"
      log: "KEYS_AND_VALUES"
    "3":
      '@type': "TENSORFLOW"
      '@input': "2_1"
      input_names:
      - "1"
      - "2"
      output_names:
      - "11"
      - "22"
      model_uri: "<path_to_model>"
    "2_1":
      '@type': "SWITCH_OUTPUT"
      '@input': "1_switch_9ed8d534"
      outputNum: 0
    "4":
      '@type': "DEEPLEARNING4J"
      '@input': "2_2"
      modelUri: "<path_to_model>"
      inputNames:
      - "1"
      - "2"
      outputNames:
      - "11"
      - "22"
    "2_2":
      '@type': "SWITCH_OUTPUT"
      '@input': "1_switch_9ed8d534"
      outputNum: 1
    "5":
      '@type': "ANY"
      '@input':
      - "3"
      - "4"
    "1_switch_9ed8d534":
      '@type': "SWITCH"
      '@input': "1"
      switchFn:
        '@type': "INT_SWITCH"
        numOutputs: 2
        fieldName: "select"
```

If the input is string type,we can change `[2_1,2_2]=switch(int,select,1)` to `[2_1,2_2]=switch(string,select,x:0,y:1,1)` in the command \(for Switch step \(string\). Same goes to three string elements such `[2_1,2_2,2_3]=switch(string,select,x:1,y:2,z:3,1)` ,then the input can passed to another three models.

The Graph Pipeline helps manage the inputs and multiple models that can be run in parallel, producing the output in instanced via Konduit-Serving.

