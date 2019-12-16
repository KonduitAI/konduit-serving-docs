---
description: >-
  This page illustrates a simple client-server interaction to perform inference
  on a Deeplearning4j image classification model using the Python SDK for
  Konduit Serving.
---

# Deeplearning4j

```python
import numpy as np 
import os
```

This page documents two ways to create Konduit configurations with the Python SDK:

1. Using Python to create a configuration, and 
2. Writing the configuration as a YAML file, then serving it using the Python SDK. 

These approaches are documented in separate tabs throughout this page. For example, the following code block shows the imports for each approach in separate tabs:

{% tabs %}
{% tab title="Python" %}
```python
from konduit import ModelConfig, TensorDataTypesConfig, ModelConfigType, \
ModelStep, ParallelInferenceConfig, ServingConfig, InferenceConfiguration

from konduit.server import Server
from konduit.client import Client
```
{% endtab %}

{% tab title="Python from YAML" %}
```python
from konduit.load import server_from_file, client_from_file
```
{% endtab %}
{% endtabs %}

## Saving models in Deeplearning4j

The following is a short Java program that loads a simple CNN model from Deeplearning4j's model zoo, initializes weights, then saves the model to a new file, "SimpleCNN.zip".

```java
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.zoo.ZooModel;
import org.deeplearning4j.zoo.model.SimpleCNN;

import java.io.File;

public class SaveSimpleCNN {
    private static int nClasses = 5;
    private static boolean saveUpdater = false;

    public static void main(String[] args) throws Exception {
        ZooModel zooModel = SimpleCNN.builder()
            .numClasses(nClasses)
            .inputShape(new int[]{3, 224, 224})
            .build();
        MultiLayerConfiguration conf = ((SimpleCNN) zooModel).conf();
        MultiLayerNetwork net = new MultiLayerNetwork(conf);
        net.init();
        System.out.println(net.summary());
        File locationToSave = new File("SimpleCNN.zip");
        net.save(locationToSave, saveUpdater);
    }
}
```

A reference Java project using Deeplearning4j 1.0.0-beta5 is provided in this repository with a Maven `pom.xml` dependencies file. If using the IntelliJ IDEA IDE, open the `java` folder as a Maven project and run the `main` function of the `SaveSimpleCNN` class.

## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

Before running this notebook, run the `build_jar.py` script and copy the JAR \(`konduit.jar`\) to this folder. Refer to the [Python SDK README](https://github.com/KonduitAI/konduit-serving/blob/master/python/README.md) for details.

## Configure the step

{% tabs %}
{% tab title="Python" %}
Define the Deeplearning4j configuration as a `ModelConfig` object.

* `tensor_data_types_config`: The ModelConfig object requires a dictionary `input_data_types`. Its keys should represent column names, and the values should represent data types as strings, e.g. `"INT32"`. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `model_config_type`: This argument requires a `ModelConfigType` object. In the Java program above, we recognised that SimpleCNN is configured as a MultiLayerNetwork, in contrast with the ComputationGraph class, which is used for more complex networks. Specify `model_type` as `MULTI_LAYER_NETWORK`, and `model_loading_path` to point to the location of Deeplearning4j weights saved in the ZIP file format.

```python
input_data_types = {"image_array": "FLOAT"}
input_names = list(input_data_types.keys())
output_names = ["output"]
port = np.random.randint(1000, 65535)

dl4j_config = ModelConfig(
    tensor_data_types_config=TensorDataTypesConfig(
        input_data_types=input_data_types
    ), 
    model_config_type=ModelConfigType(
        model_type="MULTI_LAYER_NETWORK", 
        model_loading_path=os.path.abspath("../data/multilayernetwork/SimpleCNN.zip")
    )
)
```

For the `ModelStep` object, the following parameters are specified:

* `model_config`: pass the ModelConfig object here 
* `parallel_inference_config`: specify the number of workers to run in parallel. Here, we specify `workers=1`.
* `input_names`:  names for the input data  
* `output_names`: names for the output data

```python
dl4j_step = ModelStep(
    model_config=dl4j_config,
    parallel_inference_config=ParallelInferenceConfig(workers=1),
    input_names=input_names,
    output_names=output_names
)
```
{% endtab %}

{% tab title="YAML" %}
In the Java program above, we recognised that SimpleCNN is configured as a MultiLayerNetwork, in contrast with the ComputationGraph class, which is used for more complex networks. Hence, we create a `dl4j_mln_step` of type `MULTI_LAYER_NETWORK`.

* `model_loading_path` denotes the location of the model file 
* `input_names` and `output_names` denote the names of the input and output nodes, as lists
* `input_data_types` maps the data types of the input nodes to the data type.  See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `parallel_inference_config`: specify the number of workers to run in parallel. Here, we specify `workers=1`.

```yaml
steps:
  dl4j_mln_step:
    type: MULTI_LAYER_NETWORK
    model_loading_path: ../data/multilayernetwork/SimpleCNN.zip
    input_names: 
    - image_array
    output_names: 
    - output
    input_data_types:
      image_array: FLOAT
    parallel_inference_config: 
      workers: 1
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
To find the names of input and output nodes in Deeplearning4j,

* for `input_names`, print the first element of `net.getLayerNames()`.
* for `output_names`, check the last layer when printing `net.summary()`. 
{% endhint %}

## Configure the server

Specify the following:

* `http_port`: select a random port.
* `input_data_format`, `output_data_format`: Specify input and output data formats as strings. 

{% tabs %}
{% tab title="Python" %}
The `ServingConfig` has to be passed to `Server` in addition to the steps as a Python list. In this case, there is a single step: `dl4j_step`.

```python
serving_config = ServingConfig(
    http_port=port,
    input_data_format='NUMPY',
    output_data_format='NUMPY'
)

server = Server(
    serving_config=serving_config,
    steps=[dl4j_step]
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
  log_timings: True
  extra_start_args: -Xmx8
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.
{% endhint %}

## Start the server

{% tabs %}
{% tab title="Python" %}
```python
server.start()
```

```text
Starting server...

Server has started successfully.





<subprocess.Popen at 0x2723b619ac8>
```
{% endtab %}

{% tab title="Python from YAML" %}
```python
konduit_yaml_path = "../yaml/deeplearning4j.yaml"
server = server_from_file(konduit_yaml_path)
server.start()
```
{% endtab %}
{% endtabs %}

## Configure the client

To configure the client, create a Client object with the following arguments:

* `input_data_format`: data format passed to the server for inference
* `output_data_format`: data format returned by the server endpoint 
* `return_output_data_format`: data format to be returned to the client. Note that this argument can be used to convert the output returned from the server to the client into a different format, e.g. NUMPY to JSON.

{% tabs %}
{% tab title="Python" %}
```python
client = Client(
    input_data_format='NUMPY',
    output_data_format='NUMPY',
    return_output_data_format="NUMPY",
    host='http://localhost', 
    port=port
)
```
{% endtab %}

{% tab title="YAML" %}
Add the following to your YAML configuration file: 

```yaml
client:
    input_data_format: NUMPY
    output_data_format: NUMPY
    return_output_data_format: NUMPY
    host: http://localhost
    port: 1337
```

Use `client_from_file` to create a `Client` object: 

```python
konduit_yaml_path = "../yaml/deeplearning4j.yaml"
client = client_from_file(konduit_yaml_path)
```
{% endtab %}
{% endtabs %}

## Inference

We generate a \(3, 224, 224\) array of random numbers between 0 and 255 as input to the model for prediction.

{% hint style="warning" %}
NDARRAY inputs to ModelSteps must be specified with a preceding `batchSize` dimension. For batches with a single observation, this can be done by using `np.expand_dims()` to add an additional dimension to your array. 
{% endhint %}

Before requesting for a prediction, we normalize the image to be between 0 and 1:

```python
rand_image = np.random.randint(255, size=(1, 3, 224, 224)) / 255
```

```python
prediction = client.predict({"image_array": rand_image})
print(prediction)

server.stop()
```

```text
[[4.1741084e-02 3.2335979e-01 2.5368158e-02 3.9881383e-05 6.0949111e-01]]
```

Again, we can use the `as_dict()` method of the `config` attribute of `server` to view the overall configuration:

```python
server.config.as_dict()
```

```text
{'@type': 'InferenceConfiguration',
 'steps': [{'@type': 'ModelStep',
   'inputNames': ['image_array'],
   'outputNames': ['output'],
   'modelConfig': {'@type': 'ModelConfig',
    'tensorDataTypesConfig': {'@type': 'TensorDataTypesConfig',
     'inputDataTypes': {'image_array': 'FLOAT'}},
    'modelConfigType': {'@type': 'ModelConfigType',
     'modelType': 'MULTI_LAYER_NETWORK',
     'modelLoadingPath': 'C:\\Users\\Skymind AI Berhad\\Documents\\konduit-serving-examples\\data\\multilayernetwork\\SimpleCNN.zip'}},
   'parallelInferenceConfig': {'@type': 'ParallelInferenceConfig',
    'workers': 1}}],
 'servingConfig': {'@type': 'ServingConfig',
  'httpPort': 57441,
  'inputDataFormat': 'NUMPY',
  'outputDataFormat': 'NUMPY',
  'logTimings': True}}
```

