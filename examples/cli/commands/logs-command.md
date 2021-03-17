---
description: Examples of CLI with logs command
---

# Logs Command

The`logs` command can be used to view the logs of a particular Konduit Server, given a server id. There are a couple other options to use with the `logs` command.

#### Examples

The following command outputs the log file contents of server with an id of 'inf\_server': 

```bash
$ konduit logs inf_server
```

The output of log file will print the last 10 lines by default:

```bash
      "labelName" : "label",
      "indexName" : "index",
      "probName" : "prob",
      "labels" : [ "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" ],
      "allProbabilities" : false
    } ]
  }
}
17:18:15.938 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
17:18:15.938 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 42849 with 4 pipeline steps
```

You can also output and tail the log file contents of server with an id of 'inf\_server' by adding `-f` or `--follow` option:

```bash
$ konduit logs inf_server --follow
```

You'll notice the output is similar to `konduit logs inf_server` but the text cursor is still in the tail of the printed logs. You can press CTRL + C to exit. To view the last 50 lines of the server logs, run the following command:

```bash
$ konduit logs inf_server --lines 50
```

You'll be able to see the last 50 lines of output

```bash
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
17:18:15.938 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
17:18:15.938 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 42849 with 4 pipeline steps
```



