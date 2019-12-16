---
description: >-
  This notebook illustrates a simple client-server interaction to perform
  inference on a TensorFlow model using the Python SDK for Konduit Serving.
---

# BERT



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
from konduit import ParallelInferenceConfig, ServingConfig, TensorFlowConfig, \
ModelConfigType, TensorDataTypesConfig, ModelStep, InferenceConfiguration
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

## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

Before running this notebook, run the `build_jar.py` script and copy the JAR \(`konduit.jar`\) to this folder. Refer to the [Python SDK README](https://github.com/KonduitAI/konduit-serving/blob/master/python/README.md) for details.

Start by downloading the model weights to the `data` folder.

```python
from urllib.request import urlretrieve 
from zipfile import ZipFile
dl_path = "../data/bert/bert.zip"
if not os.path.isfile(dl_path):
    urlretrieve("https://deeplearning4jblob.blob.core.windows.net/testresources/bert_mrpc_frozen_v1.zip", 
                dl_path)
with ZipFile(dl_path, 'r') as zipObj:
    zipObj.extractall()
```

## Configure the step

{% tabs %}
{% tab title="Python" %}
Define the TensorFlow configuration as a `TensorFlowConfig` object.

* `tensor_data_types_config`: The TensorFlowConfig object requires a dictionary `input_data_types`. Its keys should represent column names, and the values should represent data types as strings, e.g. `"INT32"`. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `model_config_type`: This argument requires a `ModelConfigType` object. Specify `model_type` as `TENSORFLOW`, and `model_loading_path` to point to the location of TensorFlow weights saved in the PB file format.

```python
input_data_types = {'IteratorGetNext:0': 'INT32',
                    'IteratorGetNext:1': 'INT32',
                    'IteratorGetNext:4': 'INT32'}

tensorflow_config = TensorFlowConfig(
    tensor_data_types_config = TensorDataTypesConfig(
        input_data_types=input_data_types
        ),
    model_config_type = ModelConfigType(
        model_type='TENSORFLOW',
        model_loading_path=os.path.abspath('bert_mrpc_frozen.pb')
    )
)
```

Now that we have a `TensorFlowConfig` defined, we can define a `ModelStep`. The following parameters are specified:

* `model_config`: pass the TensorFlowConfig object here 
* `parallel_inference_config`: specify the number of workers to run in parallel. Here, we specify `workers=1`.
* `input_names`:  names for the input data  
* `output_names`: names for the output data

```python
input_names = list(input_data_types.keys())
output_names = ["loss/Softmax"]

tf_step = ModelStep(
    model_config=tensorflow_config,
    parallel_inference_config=ParallelInferenceConfig(workers=1),
    input_names=input_names,
    output_names=output_names
)
```
{% endtab %}

{% tab title="YAML" %}
In the YAML file, we define `steps` with a single `tensorflow_step`. 

```yaml
steps:
  tensorflow_step:
    type: TENSORFLOW
    model_loading_path: bert_mrpc_frozen.pb
    input_names:
      - IteratorGetNext:0
      - IteratorGetNext:1
      - IteratorGetNext:4
    output_names:
      - loss/Softmax
    input_data_types:
      IteratorGetNext:0: INT32
      IteratorGetNext:1: INT32
      IteratorGetNext:4: INT32
    parallel_inference_config:
      workers: 1
```

* `model_loading_path`: location of the model file
* `input_names`, `output_names`:  names of the input and output nodes respectively.  **Important**: specify `input_names` and `output_names`as lists. 
* `input_data_types`: maps each of the `input_names` to the corresponding data type. The values should represent data types as strings, e.g. `INT32`. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `parallel_inference_config`: specify the number of workers to run in parallel. Here, we specify `workers=1`.
{% endtab %}
{% endtabs %}

{% hint style="info" %}
### Names of input and output nodes

In TensorFlow, you can find the names of your input and output nodes by iterating through`model.inputs`and `model.outputs`respectively and printing the `.os.name`attribute of each. For more details, please refer to this [StackOverflow answer](https://stackoverflow.com/a/49154874/12260518).
{% endhint %}

## Configure the server

Specify the following:

* `http_port`: select a random port.
* `input_data_format`, `output_data_format`: Specify input and output data formats as strings. 

{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.
{% endhint %}

{% tabs %}
{% tab title="Python" %}
```python
port = np.random.randint(1000, 65535)
serving_config = ServingConfig(
    http_port=port,
    input_data_format='NUMPY',
    output_data_format='NUMPY'
)
```

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Python list. In this case, there is a single step: `tf_step`.

```python
server = Server(
    serving_config=serving_config,
    steps=[tf_step]
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
```
{% endtab %}
{% endtabs %}

By default, `Server()` looks for the Konduit Serving JAR `konduit.jar` in the directory the script is run in. To change this default, use the `jar_path` argument.

## Start the server

{% tabs %}
{% tab title="Python" %}
Start the server:

```python
server.start()
```

```text
Starting server.................

Server has started successfully.





<subprocess.Popen at 0x21acae42f60>
```
{% endtab %}

{% tab title="Python from YAML" %}
```python
konduit_yaml_path = "../yaml/tensorflow-bert.yaml"
server = server_from_file(konduit_yaml_path)
server.start()
```
{% endtab %}
{% endtabs %}

## Configure the client

To configure the client, create a Client object specifying the port number:

{% tabs %}
{% tab title="Python" %}
```python
client = Client(port=port)
```
{% endtab %}

{% tab title="YAML" %}
Add the following to your YAML configuration file: 

```yaml
client:
    port: 1337
```

Create a Client object using the `client_from_file` function:

```python
konduit_yaml_path = "../yaml/tensorflow-bert.yaml"
client = client_from_file(konduit_yaml_path)
```
{% endtab %}
{% endtabs %}

## Inference

{% hint style="warning" %}
NDARRAY inputs to ModelSteps must be specified with a preceding `batchSize` dimension. For batches with a single observation, this can be done by using `np.expand_dims()` to add an additional dimension to your array. 
{% endhint %}

Load some sample data from NumPy files. Note that these are NumPy arrays, each with shape \(4, 128\):

```python
data_input = {
    'IteratorGetNext:0': np.expand_dims(np.load('../data/bert/input-0.npy'), axis=0),
    'IteratorGetNext:1': np.expand_dims(np.load('../data/bert/input-1.npy'), axis=0),
    'IteratorGetNext:4': np.expand_dims(np.load('../data/bert/input-4.npy'), axis=0)
}
```

```python
predicted = client.predict(data_input)
print(predicted)

server.stop()
```

```text
[[9.9860090e-01 1.3990625e-03]
 [7.0319971e-04 9.9929678e-01]
 [9.9866593e-01 1.3340610e-03]
 [9.7927457e-01 2.0725440e-02]]
```

The configuration is stored as a dictionary. Note that the configuration can be converted to a dictionary using the `as_dict()` method:

```python
server.config.as_dict()
```

```text
{'@type': 'InferenceConfiguration',
 'steps': [{'@type': 'ModelStep',
   'inputNames': ['IteratorGetNext:0',
    'IteratorGetNext:1',
    'IteratorGetNext:4'],
   'outputNames': ['loss/Softmax'],
   'modelConfig': {'@type': 'TensorFlowConfig',
    'tensorDataTypesConfig': {'@type': 'TensorDataTypesConfig',
     'inputDataTypes': {'IteratorGetNext:0': 'INT32',
      'IteratorGetNext:1': 'INT32',
      'IteratorGetNext:4': 'INT32'}},
    'modelConfigType': {'@type': 'ModelConfigType',
     'modelType': 'TENSORFLOW',
     'modelLoadingPath': 'C:\\Users\\Skymind AI Berhad\\Documents\\konduit-serving-examples\\notebooks\\bert_mrpc_frozen.pb'}},
   'parallelInferenceConfig': {'@type': 'ParallelInferenceConfig',
    'workers': 1}}],
 'servingConfig': {'@type': 'ServingConfig',
  'httpPort': 36846,
  'inputDataFormat': 'NUMPY',
  'outputDataFormat': 'NUMPY',
  'logTimings': True}}
```

