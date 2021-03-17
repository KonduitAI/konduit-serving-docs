---
description: Examples of CLI with inspect command
---

# Inspect Command

The `inspect` command can be used to inspect the details of a particular Konduit Server based on given the server's id. This command helps in getting the details of a server configuration which can be further filter and formatted through a query string. You can specify the query string with either the `--query` or `-q` option.

#### Examples

The following command will inspect the whole configuration of server with an id of 'inf\_server':

```bash
$ konduit inspect inf_server
```

The command will let you inspect the whole configuration setting based on your JSON/YAML file:

```bash
{
  "host" : "localhost",
  "port" : 42849,
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
        "height" : 28,
        "width" : 28,
        "dataType" : "FLOAT",
        "includeMinibatchDim" : true,
        "aspectRatioHandling" : "CENTER_CROP",
        "format" : "CHANNELS_FIRST",
        "channelLayout" : "GRAYSCALE",
        "normalization" : {
          "type" : "SCALE"
        },
        "listHandling" : "NONE"
      },
      "keys" : [ "image" ],
      "outputNames" : [ "layer0" ],
      "keepOtherValues" : true,
      "metadata" : false,
      "metadataKey" : "@ImageToNDArrayStepMetadata"
    }, {
      "@type" : "LOGGING",
      "logLevel" : "INFO",
      "log" : "KEYS_AND_VALUES"
    }, {
      "@type" : "DEEPLEARNING4J",
      "modelUri" : "dl4j-mnist.zip",
      "inputNames" : [ "layer0" ],
      "outputNames" : [ "layer5" ]
    }, {
      "@type" : "CLASSIFIER_OUTPUT",
      "inputName" : "layer5",
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

You can use `--query` command flag to get specific fields of the server configuration. For example, the following command will print the host and port of the server: 

```bash
$ konduit inspect inf_server --query {host}:{port}
```

You'll get the output based on what you have specified:

```bash
localhost:42849
```

You can also use same command flag to get pipeline details, for example: 

```bash
$ konduit inspect inf_server --query {host}:{port}-{pipeline}
```

You'll be able to see similar output including pipeline details like this:

```bash
localhost:42849-{"steps":[{"@type":"IMAGE_TO_NDARRAY","config":{"height":28,"width":28,"dataType":"FLOAT","includeMinibatchDim":true,"aspectRatioHandling":"CENTER_CROP","format":"CHANNELS_FIRST","channelLayout":"GRAYSCALE","normalization":{"type":"SCALE"},"listHandling":"NONE"},"keys":["image"],"outputNames":["layer0"],"keepOtherValues":true,"metadata":false,"metadataKey":"@ImageToNDArrayStepMetadata"},{"@type":"LOGGING","logLevel":"INFO","log":"KEYS_AND_VALUES"},{"@type":"DEEPLEARNING4J","modelUri":"dl4j-mnist.zip","inputNames":["layer0"],"outputNames":["layer5"]},{"@type":"CLASSIFIER_OUTPUT","inputName":"layer5","returnLabel":true,"returnIndex":true,"returnProb":true,"labelName":"label","indexName":"index","probName":"prob","labels":["0","1","2","3","4","5","6","7","8","9"],"allProbabilities":false}]}
```

