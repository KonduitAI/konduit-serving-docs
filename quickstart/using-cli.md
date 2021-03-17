---
description: Guide to start using Konduit-Serving with CLI
---

# Using CLI

This document will demonstrate using Konduit-Serving using mainly CLI tools. You can deploy ML/DL models to production using minimal effort using Konduit-Serving. Let's look at the process of building and installing Konduit-Serving from source and how to deploy a model using a simple configuration. 

### Prerequisite

You will need following prerequisites to follow along

* Maven 3.x
* JDK 8
* Git

## Installation from Sources

The following two sections explains how to clone, build and install Konduit-Serving from sources.

To build from source, follow the guide below

{% page-ref page="../building-from-source.md" %}

To install the respective built binaries you can navigate to the section below

{% page-ref page="../installation.md" %}

After you've installed Konduit-Serving in your local machine you can switch to a terminal and verify the installation by running

```text
konduit --version
```

You'll see an output similar to the one below

```text
$ konduit --version
------------------------------------------------
Version: 0.1.0-SNAPSHOT
Commit hash: 3dd38832
Commit time: 01.03.2021 @ 03:37:08 MYT
Build time: 07.03.2021 @ 16:57:51 MYT
```

## Deploying Models

Let's look at how to deploy a dl4j/keras model using Konduit-Serving

### Cloning Examples Repo

Let's clone the `konduit-serving-examples` repo

```text
git clone https://github.com/KonduitAI/konduit-serving-examples.git
```

and navigate to the `quickstart` folder

```text
cd konduit-serving-demo/quickstart
```

The examples we want to run are under the folders `3-keras-mnist` and `5-dl4j-mnist`. Let's follow a basic workflow for both models using the Konduit-Serving CLI.

{% tabs %}
{% tab title="Keras" %}
Navigate to `3-keras-mnist` 

```text
cd 3-keras-mnist
```

Here, you'll find the following files:

```text
.
├── keras-mnist.ipynb   |    A supplementary jupyter/beakerx notebook
├── keras.h5            |    Model file we want to serve
├── keras.json          |    Konduit-Serving configuration
├── test-image.jpg      |    Test input for predictions
└── train.py            |    Script for creating 'keras.h5' model file
```

The `keras.json` contains the configuration file for running an MNIST dataset trained model in Keras. To serve the model, execute the following command

```bash
konduit serve --config keras.json -id keras-server 
```

You'll be able to see a similar output like the following

```bash
.
.
.
15:00:08.575 [vert.x-worker-thread-0] INFO  a.k.s.m.d.step.DL4JRunner - 
15:00:08.576 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

15:00:08.576 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
.
.
.
.
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 40987 with 4 pipeline steps
```

The last line will show you the details about which URL the server is serving the models at.

Press `Ctrl + C`, or execute `konduit stop keras-server` to kill the server. 

To run the server in the background, you can run the same command with the `--background` or `-b` flag.

```text
konduit serve --config keras.json -id keras-server --background
```

You'll see something similar to

```text
Starting konduit server...
Expected classpath: /Users/konduit/Projects/Konduit/konduit-serving/konduit-serving-tar/target/konduit-serving-tar-0.1.0-SNAPSHOT-dist/bin/../konduit.jar
INFO: Running command /Users/konduit/opt/miniconda3/jre/bin/java -Dkonduit.logs.file.path=/Users/konduit/.konduit-serving/command_logs/keras-server.log -Dlogback.configurationFile=/Users/konduit/Projects/Konduit/konduit-serving/konduit-serving-tar/target/konduit-serving-tar-0.1.0-SNAPSHOT-dist/bin/../conf/logback-run_command.xml -cp /Users/konduit/Projects/Konduit/konduit-serving/konduit-serving-tar/target/konduit-serving-tar-0.1.0-SNAPSHOT-dist/bin/../konduit.jar ai.konduit.serving.cli.launcher.KonduitServingLauncher run --instances 1 -s inference -c keras.json -Dserving.id=keras-server
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs keras-server'
```

To list the server, simply run

```text
konduit list
```

You'll see the running servers as a list

```text
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS     
 1   | keras-server                   | inference  | localhost:1000       | 1200    | Started
```

To view the logs, you can run the following command

```text
konduit logs keras-server --lines 2
```

The `--lines` or `-l` flag shows the specified number of last lines. By executing the above command you'll see the following

```text
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 1000 with 4 pipeline steps
```

Now finally, let's look at running predictions with Konduit-Serving by sending an image file to the server. 

```text
konduit predict keras-server -it multipart 'image=@test-image.jpg'
```

It will convert the image into an n-dimensional array and then send the input to the keras model and you'll see the following output

```text
{
  "output_layer" : [ [ 9.0376153E-7, 1.0595608E-8, 1.3115231E-5, 0.44657645, 6.748624E-12, 0.5524258, 1.848306E-7, 2.7652052E-9, 9.76023E-4, 7.5933513E-6 ] ],
  "prob" : 0.5524258017539978,
  "index" : 5,
  "label" : "5"
}
```
{% endtab %}

{% tab title="DL4J" %}
Navigate to `5-dl4j-mnist`

```bash
cd 5-dl4j-mnist
```

Here, you'll find the following files:

```text
.
├── dl4j-mnist.ipynb    |    A supplementary jupyter/beakerx notebook
├── dl4j-mnist.zip      |    Model file we want to serve
├── dl4j.json           |    Konduit-Serving configuration 
└── test-image.jpg      |    Test input for predictions
```

The `dl4j.json` contains the configuration file for running an MNIST dataset trained model in DL4J. To serve the model, execute the following command

```bash
konduit serve --config dl4j.json -id dl4j-server 
```

You'll be able to see a similar output like the following

```bash
.
.
.
15:00:08.575 [vert.x-worker-thread-0] INFO  a.k.s.m.d.step.DL4JRunner - 
15:00:08.576 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

15:00:08.576 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
.
.
.
.
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 40987 with 4 pipeline steps
```

The last line will show you the details about which URL the server is serving the models at.

Press `Ctrl + C`, or execute `konduit stop keras-server` to kill the server. 

To run the server in the background, you can run the same command with the `--background` or `-b` flag.

```text
konduit serve --config dl4j.json -id dl4j-server --background
```

You'll see something similar to

```text
Starting konduit server...
Expected classpath: /Users/konduit/Projects/Konduit/konduit-serving/konduit-serving-tar/target/konduit-serving-tar-0.1.0-SNAPSHOT-dist/bin/../konduit.jar
INFO: Running command /Users/konduit/opt/miniconda3/jre/bin/java -Dkonduit.logs.file.path=/Users/konduit/.konduit-serving/command_logs/dl4j-server.log -Dlogback.configurationFile=/Users/konduit/Projects/Konduit/konduit-serving/konduit-serving-tar/target/konduit-serving-tar-0.1.0-SNAPSHOT-dist/bin/../conf/logback-run_command.xml -cp /Users/konduit/Projects/Konduit/konduit-serving/konduit-serving-tar/target/konduit-serving-tar-0.1.0-SNAPSHOT-dist/bin/../konduit.jar ai.konduit.serving.cli.launcher.KonduitServingLauncher run --instances 1 -s inference -c dl4j.json -Dserving.id=dl4j-server
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs dl4j-server'
```

To list the server, simply run

```text
konduit list
```

You'll see the running servers as a list

```text
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS     
 1   | dl4j-server                    | inference  | localhost:1000       | 1200    | Started
```

To view the logs, you can run the following command

```text
konduit logs dl4j-server --lines 2
```

The `--lines` or `-l` flag shows the specified number of last lines. By executing the above command you'll see the following

```text
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
15:00:08.752 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 1000 with 4 pipeline steps
```

Now finally, let's look at running predictions with Konduit-Serving by sending an image file to the server. 

```text
konduit predict dl4j-server -it multipart 'image=@test-image.jpg'
```

It will convert the image into an n-dimensional array and then send the input to the DL4J model and you'll see the following output

```text
{
  "layer5" : [ [ 1.845163E-5, 1.8346094E-6, 0.31436875, 0.43937472, 2.6101702E-8, 0.24587035, 5.9430695E-6, 3.3270408E-4, 6.3698195E-8, 2.708706E-5 ] ],
  "prob" : 0.439374715089798,
  "index" : 3,
  "label" : "3"
}
```
{% endtab %}
{% endtabs %}

Congratulations! You've learned the basic workflow for Konduit-Serving using the Command Line Interface.

