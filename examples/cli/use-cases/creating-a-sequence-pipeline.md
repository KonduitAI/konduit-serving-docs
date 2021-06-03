---
description: To create boilerplate configurations
---

# Creating a Sequence Pipeline

A Sequence Pipeline is used to treat the data and Machine Learning or Deep Learning model in a series of steps from pre-processing to model serving and post-processing on the output product. In this example, the `CLI` command specifies on `konduit config` is used to configure the configuration file to serve the models on `Konduit-Serving`. You'll be able to follow this example on your local terminal on any directory.

If deploying the model does not need pre- nor post-processing, only one step, a deep learning model is needed. This configuration is defined using a single Step to serve a model, and the command for creating the configuration file is like the following.

```text
$ konduit config --pipeline dl4j --output config_dl4j.yaml --yaml
```

The YAML configuration is as follows.

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
```

The Steps are included in the `--pipeline` based on the model's requirement and how output should represent. For example, the model fetches an image input, so the `image_to_ndarray` should be pre-processing step to convert the image into an array. The table below shows all steps that can be used in the Sequence Pipeline.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Pre-processing Step</th>
      <th style="text-align:left">Model/Python Step</th>
      <th style="text-align:left">Post-processing Step</th>
      <th style="text-align:left">Logging</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <ul>
          <li>image_to_ndarray</li>
        </ul>
      </td>
      <td style="text-align:left">
        <p></p>
        <ul>
          <li>dl4j</li>
          <li>keras</li>
          <li>tensorflow</li>
          <li>nd4jtensorflow</li>
          <li>onnx</li>
          <li>samediff</li>
          <li>python</li>
        </ul>
      </td>
      <td style="text-align:left">
        <ul>
          <li>crop_grid</li>
          <li>crop_fixed<em>_</em>grip</li>
          <li>draw_bounding_box</li>
          <li>draw_fixed<em>_</em>grid</li>
          <li>draw_segmentation</li>
          <li>extract_bounding_box</li>
          <li>camera_frame_capture</li>
          <li>video_frame_capture</li>
          <li>ssd_to_bounding_box</li>
          <li>show_image</li>
          <li>classifier_output</li>
        </ul>
      </td>
      <td style="text-align:left">
        <p></p>
        <ul>
          <li>logging</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

Here is another example with a series of step in Sequence Pipeline \(`image_to_ndarray` to `nd4jtensorflow` to `classifier_output`\). The input image needs to convert into an _n-_D array before feeding into the model and produce the classification output. A command likes below:

```text
$ konduit config -p image_to_ndarray,nd4jtensorflow,classifier_output -o config.json
```

The command would give the configuration file in JSON with the complete Pipeline Steps.

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
      "@type" : "IMAGE_TO_NDARRAY",
      "config" : {
        "height" : 100,
        "width" : 100,
        "dataType" : "FLOAT",
        "includeMinibatchDim" : true,
        "aspectRatioHandling" : "CENTER_CROP",
        "format" : "CHANNELS_FIRST",
        "channelLayout" : "RGB",
        "normalization" : {
          "type" : "SCALE"
        },
        "listHandling" : "NONE"
      },
      "keys" : [ "key1", "key2" ],
      "outputNames" : [ "output1", "output2" ],
      "keepOtherValues" : true,
      "metadata" : false,
      "metadataKey" : "@ImageToNDArrayStepMetadata"
    }, {
      "@type" : "ND4JTENSORFLOW",
      "inputNames" : [ "1", "2" ],
      "outputNames" : [ "11", "22" ],
      "modelUri" : "<path_to_model>"
    }, {
      "@type" : "CLASSIFIER_OUTPUT",
      "inputName" : "inputName (optional)",
      "returnLabel" : true,
      "returnIndex" : true,
      "returnProb" : true,
      "labelName" : "label",
      "indexName" : "index",
      "probName" : "prob",
      "labels" : [ "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" ],
      "allProbabilities" : false
    } ]
  }
}
```

{% hint style="info" %}
Here is the example which use almost similar steps. You can find the JSON file on [https://github.com/ShamsUlAzeem/konduit-serving-demo/blob/master/demos/4-tensorflow-mnist/tensorflow.json](https://github.com/ShamsUlAzeem/konduit-serving-demo/blob/master/demos/4-tensorflow-mnist/tensorflow.json)
{% endhint %}

Every Step in the Pipeline needs to modify based on the input characteristics, model configurations and how output should looks like in the end. Using a configuration file allows you to serve the model with Konduit-Serving. 

