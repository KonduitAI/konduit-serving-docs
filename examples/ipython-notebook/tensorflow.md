---
description: Example of Tensorflow framework with CUSTOM endpoints
---

# Tensorflow

### Overview

In this example, we demonstrate Konduit-Serving with a complete pipeline step consist of :

1. Pre-processing step
2. Running a deep learning model
3. Post-processing step, expressing the output in a way human can understand

### Adding package to the classpaths

Let's add the main package of Konduit-Serving so that the notebook can load all the required libraries that need to be used by Jupyter Notebook kernel. 

```text
%classpath add jar ../../konduit.jar
```

{% hint style="info" %}
Classpaths can be considered similar to `site-packages` in the python ecosystem where each library that's to be imported to your code is loaded from.
{% endhint %}

### Starting a server

Before starting a server, let's check if there is a running server with id `tensorflow-mnist` and stop it. This command may use once the server finished.

```text
%%bash
konduit stop tensorflow-mnist
```

You'll get the following message if there is no server running with mentioned id.

```text
No konduit server exists with an id: 'tensorflow-mnist'.
```

Or, if you have a running server you'll received as following.

```text
Stopping konduit server 'tensorflow-mnist'
Application 'tensorflow-mnist' terminated with status 0
```

Now, let's start the server with an id of `tensorflow-mnist` using `tensorflow.json` as a configuration file in the background without creating the manifest jar file before launching the server.

```text
%%bash
konduit serve -id tensorflow-mnist -c tensorflow.json -rwm --background
```

You'll be able to view a similar message like below.

```text
Starting konduit server...
Using classpath: /root/konduit/bin/../konduit.jar
INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command_logs/tensorflow-mnist.log -Dlogback.configurationFile=/tmp/logback-run_command_5fd1b7c309d448ea.xml -jar /root/konduit/bin/../konduit.jar run --instances 1 -s inference -c tensorflow.json -Dserving.id=tensorflow-mnist
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs tensorflow-mnist'
```

View the logs for the last 1000 lines `-l` for a given id by using the `konduit logs` command.

```text
%%bash
konduit logs tensorflow-mnist -l 1000
```

The output of log is similar as following.

```text
08:23:16.423 [main] INFO  a.k.s.c.l.command.KonduitRunCommand - Processing configuration: /root/konduit/demos/4-tensorflow-mnist/tensorflow.json
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

08:23:17.982 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
.
.
.
08:23:18.145 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: '0.0.0.0'
08:23:18.145 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 9008 with 4 pipeline steps
```

### Sending an input to served model

We can display all the available image for the inference result of the model. You'll be able to see the picture from zero to nine.

```text
%%html
  <div style="display: flex; justify-content: center; align-items: center; border: 1px solid black;">
    <div style="display: inline-block; margin: 2px">
        <img src="test_files/test_input_number_0.png"/>
    </div>

    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_1.png"/>
    </div>

    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_2.png"/>
    </div>
      
    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_3.png"/>
    </div>
      
    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_4.png"/>
    </div>
      
    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_5.png"/>
    </div>

    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_6.png"/>
    </div>

    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_7.png"/>
    </div>
      
    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_8.png"/>
    </div>
      
    <div style="display: inline-block; margin: 10px">
        <img src="test_files/test_input_number_9.png"/>
    </div>
      
</div>
```

Let's take one of the testing images and send it to the served model in Konduit-Serving. With the help of the pipeline considered in the configuration, we could translate the image into an array and feed it into the model.

```text
%%bash
konduit predict tensorflow-mnist --input-type multipart "image=@test_files/test_input_number_9.png"
```

Thus, giving a result straight forward with the label of number classification based on prediction probabilities.

```text
{
  "output_layer/Softmax" : [ [ 3.0811898E-7, 6.085964E-6, 1.1470697E-4, 1.5436264E-9, 0.0023717284, 1.7763212E-12, 6.587209E-11, 0.99487466, 4.904844E-11, 0.0026325122 ] ],
  "prob" : 0.9948746562004089,
  "index" : 7,
  "label" : "7"
}
```

