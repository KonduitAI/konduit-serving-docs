---
description: >-
  Konduit Serving is a framework focused on deploying machine learning pipelines
  to production.
---

# Introduction

## Overview

Konduit-Serving provides building blocks for developers to write their own production machine learning pipelines from pre-processing to model serving, exposable as a simple REST API. It also allows embedding Python/Java code \(pre/post processing, custom models\). It primarily focuses on server and edge deployments using REST and gRPC endpoints. Pipelines are deployed and defined using JSON/YAML or a command line interface.

The core abstraction is an idea called a **pipeline step**. A pipeline step performs a task such as:

1. Pre-processing steps;
2. Running one or more machine learning models; and
3. Post-processing steps: transforming the output in a way that can be understood by humans, such as labels in a classification example,

as part of using a machine learning model in a deployment scenario.

For instance, `TensorflowStep, KerasStep and Dl4jStep` performs inference on TensorFlow, Keras, Deeplearning4j \(DL4J\), respectively. Similarly, there are multiple steps defined in Konduit-Serving for setting up pre/post-processing in the machine learning pipeline.

A custom pipeline step can be built using a `PythonStep`. This allows you to embed pre/post-processing steps into your machine learning pipeline, or to serve models built in frameworks that do not have built-in model steps such as **scikit-learn** and **PyTorch**.

### **High-Level Architecture**

Konduit-Serving utilizes many popular and performant libraries. Out of them the major ones are [Vert.x](https://vertx.io/) and [Deeplearning4J](https://deeplearning4j.org/). 

#### **Vert.x Library**

Vert. x is an open source, reactive and polyglot software development toolkit and has great support for Java language. For Konduit-Serving, Vert.x is used for building and managing CLI, Web-servers and implementing data transmission backends through multiple protocols like [gRPC](https://en.wikipedia.org/wiki/GRPC), [MQTT](https://en.wikipedia.org/wiki/MQTT) and [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol). It also implements a [Kafka](https://kafka.apache.org/) client setup and can link with a Kafka backend server. 

Vert.x also allows us to scale our Konduit-Serving deployments very efficiently on multiple nodes. Although this feature hasn’t been tested thoroughly, Konduit-Serving has the ability to do it. The following stack shows how the different components in the stack work together.  


![](https://docs.google.com/drawings/u/2/d/sWyHSpGIxUFD7erZvfIL7nw/image?w=477&h=327&rev=196&ac=1&parent=1pghi_Njn8fb-rcy9nOwCivozE5-CUEkbvq0vyDYCibc)

### **DL4J Support**

In prior phases,  we have focused on benchmarks running tensorflow models on aurora.

We have also added support for running dl4j on aurora from java. This means running aurora math workloads from java via compiled libnd4j c++ that uses NCC. We’ve managed to get dl4j compiled on Aurora running via javacpp. An up to date link can be found [here](https://github.com/KonduitAI/deeplearning4j/wiki/Current-Build-Status-On-Aurora).

A build of konduit-serving that runs applications on aurora is pending, but possible based on the current work that’s been done.

Konduit-serving runs dl4j through an interface agnostic architecture. This means konduit-serving is not aware of which  backend it’s running. When compiling konduit-serving, you can use the command line to build a specific spin of konduit-serving optimized for a particular use case.

In our case, when running konduit-serving in production  a particular spin can be used for certain operating systems and hardware allowing for testing on 1 platform, but deployment on another.

The principal way a user would use aurora on konduit-serving and dl4j is the samediff framework. Aurora is accessible in a transparent way just by setting up a jar file containing the backend for aurora. GPUs are accessible in a similar way.

The SameDiff pipeline step is the component to use in order to make aurora accessible to konduit-serving. 

SameDiffStep configuration looks like this from java:

```java
Pipeline p = SequencePipeline.builder()
               .add(SameDiffStep.builder()
                       .modelUri(f.toURI().toString())
                       .outputNames(Collections.singletonList("out"))
                       .build())
               .build();
```

The configuration above represents a standalone model that runs a model specified as a file. A SameDiff model is saved as a [flatbuffers](https://google.github.io/flatbuffers/) file. This file is a descriptor containing the graph layout and associated weights for each ndarray embedded in the samediff graph.

In order to input variables in to a graph, graphs in different frameworks \(including tensorflow and pytorch\) rely on a concept called PlaceHolders. Placeholders are what you pass in to a graph where an input variable is passed in and replaces a stub element in the graph. A SameDiffStep allows you to specify the input and output placeholders to get different outputs out of the graph accessible by name. All pipeline steps that use a graph based framework require these input and output steps.

Sometimes, these inputs and outputs can be automatically inferred, but it’s generally a good idea. Specifying less outputs allows a user to specify only what outputs they want rather than everything.

### Supported Frameworks

The following main model pipeline step execution framework are supported by Konduit-Serving:

#### **Models Supported Via Onnx**

1. Pytorch
2. MXNet
3. Others which can be converted into ONNX\)
4. PMML \(Experimental\)
5. Predictive Model Markup Language also a platform interchange format for mostly all of the traditional ML models such as random forest.

#### Models supported via PMML

1. Scikit-learn
2. SparkML
3. XGBoost
4. Others with supports PMML conversion
5. Custom models
6. Through custom Python or Java code

#### **Custom models**

Through custom Python or Java code

### **Source Code**

Konduit-serving source code can be found [here](https://github.com/KonduitAI/konduit-serving). There’s also a few demos and use cases on the repo link [here](https://github.com/KonduitAI/nec-sra-workshop). The examples are based on a Docker image implementation for both CPU and GPU backends which is a great source to start learning more about Konduit-Serving.

To get started with Konduit Serving, check out the Quickstart page.

