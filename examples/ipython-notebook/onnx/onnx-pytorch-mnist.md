---
description: Running and MNIST dataset classifier through CUSTOM image endpoints
---

# Pytorch \(MNIST\)

### Overview

This example shows a complete step in the pipeline through custom endpoints:

1. Pre-processing step
2. Serve a model step
3. Post-processing step

### Adding package to the classpath

We need to add the main package to the classpath so that the notebook can load all the necessary libraries from Konduit-Serving into the Jupyter Notebook kernel.

```text
%classpath add jar ../../konduit.jar
```

{% hint style="info" %}
Classpaths can be considered similar to `site-packages` in the python ecosystem where each library that's to be imported to your code is loaded from.
{% endhint %}

### Viewing the configuration file

To view the configuration file contents, use the following command and select the JSON file you want to view.

```text
%%bash
less config.json
```

You'll be able to view the following.

```text
{
  "host" : "localhost",
  "port" : 0,
  "protocol" : "HTTP",
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
      "outputNames" : [ "Input3" ],
      "keepOtherValues" : true,
      "metadata" : false,
      "metadataKey" : "@ImageToNDArrayStepMetadata"
    }, {
      "@type" : "LOGGING",
      "logLevel" : "INFO",
      "log" : "KEYS_AND_VALUES"
    }, {
      "@type" : "ONNX",
      "modelUri" : "mnist.onnx",
      "inputNames" : [ "Input3" ],
      "outputNames" : [ "Plus214_Output_0" ]
    }, {
      "@type" : "CLASSIFIER_OUTPUT",
      "inputName" : "Plus214_Output_0",
      "labels" : [ "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" ],
      "allProbabilities" : false
    } ]
  }
}

```

### Starting a server

Starts a server in the background with an id of `onnx-mnist` using `config.json` as configuration file without creating the manifest jar file before launching the server.

```text
%%bash
konduit serve -id onnx-mnist -c config.json -rwm -b
```

You'll get the following message once the server starts in the background.

```text
Starting konduit server...
Using classpath: /root/konduit/bin/../konduit.jar
INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command_logs/onnx-mnist.log -Dlogback.configurationFile=/tmp/logback-run_command_2ead2d4d1b15431d.xml -jar /root/konduit/bin/../konduit.jar run --instances 1 -s inference -c config.json -Dserving.id=onnx-mnist
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs onnx-mnist'
```

We use `konduit list` command to view the list of activated server

```text
%%bash
konduit list
```

The list of the activated server is below.

```text
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS     
 1   | onnx-mnist                     | inference  | localhost:33895      | 888     | started    
```

To view the logs of the running server, use `konduit logs` commands.

```text
%%bash
konduit logs onnx-mnist --lines 100
```

The output of server logging.

```text
14:59:45.188 [main] INFO  a.k.s.c.l.command.KonduitRunCommand - Processing configuration: /root/konduit/demos/2-pytorch-onnx-mnist/config.json
.
.
. 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

14:59:45.601 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
.
.
.
14:59:45.733 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
14:59:45.733 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 33895 with 4 pipeline steps

```

### Making a prediction

Let's display the test image before feeding it as an input into the model for the classification.

```text
%%html
<img src="test-image.jpg" alt="title">
```

Here is the test image used in the prediction for this example:

![](../../../.gitbook/assets/test-image.jpg)

`konduit predict` command is used to classify the image based on the model served in Konduit-Serving with the id given before. 

```text
%%bash
konduit predict onnx-mnist --input-type multipart 'image=@test-image.jpg'
```

You'll be able to get the output similar to the following.

```text
{
  "Plus214_Output_0" : [ [ -1.7924803, -9.652266, 11.478509, 5.148998, -7.9367347, 9.756878, 0.544513, -7.6820283, 9.234719, -6.431969 ] ],
  "prob" : 11.478508949279785,
  "index" : 2,
  "label" : "2"
}
```

### Stopping the server

After we're finished with the server, we can terminate it through the `konduit stop` command.

```text
%%bash
konduit stop onnx-mnist
```

You'll receive this message once the server is terminated.

```text
Stopping konduit server 'onnx-mnist'
Application 'onnx-mnist' terminated with status 0
```

