---
description: Example of DL4J framework with CUSTOM endpoints
---

# DL4J

### Including package to the classpath

Before starting to serve the model, let's add the main package to the classpath to load the whole necessary libraries to Jupyter Notebook kernel from Konduit-Serving.

```text
%classpath add jar ../../konduit.jar
```

{% hint style="info" %}
Classpaths can be considered similar to `site-packages` in the python ecosystem. It is loaded from each library that's to be imported to your code.
{% endhint %}

### Starting a server <a id="Model-Link"></a>

Let's start a server with an id of `dl4j-mnist` and use `dl4j.json` as the configuration file.

```text
%%bash
konduit serve -id dl4j-mnist -c dl4j.json -rwm -b
```

You'll notice with the following message indicating the server is starting.

```text
Starting konduit server...
Using classpath: /root/konduit/bin/../konduit.jar
INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command_logs/dl4j-mnist.log -Dlogback.configurationFile=/tmp/logback-run_command_a6000ad26ed94583.xml -jar /root/konduit/bin/../konduit.jar run --instances 1 -s inference -c dl4j.json -Dserving.id=dl4j-mnist
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs dl4j-mnist'

```

{% hint style="info" %}
The DL4J model is taken from the dl4j-example here: [https://github.com/eclipse/deeplearning4j-examples/blob/master/mvn-project-template/src/main/java/org/deeplearning4j/examples/sample/LeNetMNIST.java](https://github.com/eclipse/deeplearning4j-examples/blob/master/mvn-project-template/src/main/java/org/deeplearning4j/examples/sample/LeNetMNIST.java)
{% endhint %}

Use `konduit logs` to get the logs of served model.

```text
%%bash
konduit logs dl4j-mnist -l 100
```

The output of logging is similar to the below.

```text
.
.
.
15:00:54.683 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

15:00:54.683 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
15:00:54.703 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - MetricsProvider implementation detected, adding endpoint /metrics
15:00:54.718 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - No GPU binaries found. Selecting and scraping only CPU metrics.
15:00:54.861 [vert.x-eventloop-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Writing inspection data at '/root/.konduit-serving/servers/1517.data' with configuration: 
.
.
.
15:00:54.862 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:00:54.862 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 39487 with 4 pipeline steps
```

We'll be able to use `konduit list` command to view all active servers.

```text
%%bash
konduit list
```

These are examples of active servers if the previous one is still in use. 

```text
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS     
 1   | keras-mnist                    | inference  | localhost:33387      | 31757   | started    
 2   | dl4j-mnist                     | inference  | localhost:35921      | 31893   | started  
```

### Sending an input to served model

We're going to display the test image first before feeding it into the model.

```text
%%html
<img src="test-image.jpg"/>
```

The previous image is used as the testing image for this deployed model:

![](../../.gitbook/assets/test-image.jpg)

Now, let's predict the input by using the test image above.

```text
%%bash
konduit predict dl4j-mnist -it multipart "image=@test-image.jpg"
```

You'll see the following output with the label of classification.

```text
{
  "layer5" : [ [ 1.845163E-5, 1.8346094E-6, 0.31436875, 0.43937472, 2.6101702E-8, 0.24587035, 5.9430695E-6, 3.3270408E-4, 6.3698195E-8, 2.708706E-5 ] ],
  "prob" : 0.439374715089798,
  "index" : 3,
  "label" : "3"
}
```

### Stopping the server

We can stop the running server by using `konduit stop` command.

```text
%%bash
konduit stop dl4j-mnist
```

You'll see this output once the mentioned id's server is terminated. 

```text
Stopping konduit server 'dl4j-mnist'
Application 'dl4j-mnist' terminated with status 0
```

