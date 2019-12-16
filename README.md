---
description: >-
  Konduit Serving is a serving system and framework focused on deploying machine
  learning pipelines to production.
---

# Introduction

## Overview

Konduit Serving \(like [Seldon](http://seldon.io/) and [MLflow](http://mlflow.org/)\) provides building blocks for developers to write their own production machine learning pipelines from pre-processing to model serving, exposable as a simple REST API. 

The core abstraction is an idea called a **pipeline step**. A pipeline step performs a task as part of using a machine learning model in a deployment. These steps generally include:

1. Pre- or post-processing steps
2. One or more machine learning models
3. Transforming the output in a way that can be understood by humans, such as labels in a classification example.

For instance, if you want to run arbitrary Python code for pre-processing purposes, you can use a`PythonStep`. 

{% page-ref page="examples/onnx.md" %}

Konduit Serving also contains functionality for other pre-processing tasks, such as DataVec transform processes and image transforms. 

{% page-ref page="examples/datavec.md" %}

To perform inference on a \(mix of\) TensorFlow, Keras, DL4J or PMML models, use `ModelStep`. 

{% page-ref page="examples/tensorflow-model-serving/" %}

{% page-ref page="examples/dl4j.md" %}

{% page-ref page="examples/keras.md" %}

## Usage

A Konduit Serving instance [can be configured using a YAML file](yaml-configurations.md). The following YAML configuration file configures a Konduit Serving instance to run a short Python script as specified in the `python_code` argument:  

```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
steps:
  python_step:
    type: PYTHON
    python_code: |
      first += 2
      second = first
    python_inputs:
      first: NDARRAY
    python_outputs:
      second: NDARRAY
client:
    port: 1337
```

[Installing the Konduit Serving Python SDK](installation.md) exposes the `konduit` command line tool. Assuming the YAML file above is saved in the current directory as `simple.yaml`, a Konduit Serving instance can be started by running the following code in the command line:

```bash
konduit serve --config simple.yaml
```

## Why Konduit Serving?

Konduit Serving was built with the goal of providing proper low level interoperability with native math libraries such as TensorFlow and Deeplearning4j's core math library libnd4j. At the core of Konduit Serving are the [JavaCPP Presets](https://github.com/bytedeco/javacpp-presets), [vertx](http://vertx.io) and [Deeplearning4j](http://deeplearning4j.org) for running Keras models in Java.

#### Better performance, more secure

Combining JavaCPP's low-level access to C-like apis from Java, with Java's robust server side application development \(vertx on top of [netty](http://netty.io/)\) allows for better access to faster math code in production while minimizing the surface area where native code = more security flaws \(mainly in server side networked applications\). This allows us to do things like in zero-copy memory access of NumPy arrays or Arrow records for consumption straight from the server without copy or serialization overhead. Extending that to Python SDK, we know when to return a raw Arrow record and return it as a pandas DataFrame. 

When dealing with deep learning, we can handle proper inference on the GPU \(batching large workloads\).

#### Python-first

We strive to provide a Python-first SDK that makes it easy to integrate pipelines into a Python-first workflow. 

#### Java microservices

A vertx-based model server and pipeline development framework allows a thin abstraction that can be embedded in a Java microservice.

#### Modern visualization standards 

We want to expose [modern standards](http://prometheus.io/) for monitoring everything from your GPU to your inference time. Konduit Serving supports visualization applications such as [Grafana](http://grafana.com) that support the [Prometheus](https://prometheus.io/) standard for visualizing data.

#### Enterprise integration

We aim to provide integrations with more enterprise platforms typically seen outside the big data space.



