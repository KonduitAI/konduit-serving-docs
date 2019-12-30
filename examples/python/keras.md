---
description: >-
  This page illustrates a simple client-server interaction to perform inference
  on a Keras LSTM model using the Python SDK for Konduit Serving.
---

# Keras \(TensorFlow 2.0\)

```python
from konduit import ModelConfig, ParallelInferenceConfig, ModelConfigType, \
ModelStep, ServingConfig

from konduit.server import Server
from konduit.client import Client
import os 
import numpy as np
```

## Saving models in Keras HDF5 \(.h5\) format

HDF5 model files can be saved with the `.save()` method. Refer to the [TensorFlow documentation for Keras](https://www.tensorflow.org/guide/keras/save_and_serialize) for details.

{% hint style="info" %}
Keras model loading functionality in Konduit Serving converts Keras models to Deeplearning4J models. As a result, Keras models containing operations not supported in Deeplearning4J cannot be served in Konduit Serving. See [issue 8348](https://github.com/eclipse/deeplearning4j/issues/8348).
{% endhint %}

## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

Before running this notebook, run the `build_jar.py` script or the `konduit init` command. Refer to the [Building from source](../../building-from-source.md#manual-build) page for details.

## Configure the step

{% tabs %}
{% tab title="Python" %}
Define the Keras configuration as a `ModelConfig` object.

* `model_config_type`: This argument requires a `ModelConfigType` object. Specify `model_type` as `KERAS`, and `model_loading_path` to point to the location of Keras weights saved in the HDF5 file format.

For the `ModelStep` object, the following parameters are specified:

* `model_config`: pass the `ModelConfig` object here.
* `parallel_inference_config`: specify the number of workers to run in parallel. Here, we specify `workers=1`.
* `input_names`:  names for the input nodes.
* `output_names`: names for the output nodes.

```python
keras_config = ModelConfig(    
    model_config_type = ModelConfigType(
        model_type='KERAS',
        model_loading_path=os.path.abspath(
            f'../data/keras/embedding_lstm_tensorflow_2.h5'
        )
    )
)

keras_step = ModelStep(
    model_config=keras_config, 
    parallel_inference_config=ParallelInferenceConfig(workers=1), 
    input_names=["input"], 
    output_names=["lstm_1"]
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
steps:
  keras_step:
    type: KERAS
    model_loading_path: ../data/keras/embedding_lstm_tensorflow_2.h5
    input_names:
    - input 
    output_names:
    - lstm_1
```

* `type`: specify this as `KERAS`.
* `model_loading_path`: location of the model weights.
* `input_names`, `output_names`: names for the input and output nodes, as lists.  
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Input and output names can be obtained by visualizing the graph in [Netron](https://github.com/lutzroeder/netron).
{% endhint %}

## Configure the server

{% tabs %}
{% tab title="Python" %}
In the `ServingConfig`, specify a port number.

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Python list. In this case, there is a single step: `keras_step`.

```python
serving_config = ServingConfig(http_port=1337)

server = Server(
    serving_config=serving_config, 
    steps=[keras_step]
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
serving:
  http_port: 1337
```
{% endtab %}
{% endtabs %}

## Start the server

{% tabs %}
{% tab title="Python" %}
Use the `.start()` method:

```python
server.start()
```

```text
Starting server..

Server has started successfully.





<subprocess.Popen at 0x11b2e11bb48>
```
{% endtab %}

{% tab title="YAML" %}
```python
input_array = np.random.uniform(size = [10])
konduit_yaml_path = "../yaml/keras.yaml"
server = server_from_file(konduit_yaml_path)
server.start()
```
{% endtab %}
{% endtabs %}

## Configure the client

To configure the client, create a Client object with the `port` argument.

Note that you should create the Client object after the Server has started, so that Client can inherit the Server's attributes.

{% tabs %}
{% tab title="Python" %}
```python
client = Client(port=1337)
```
{% endtab %}

{% tab title="YAML" %}
Add the following to your YAML file:

```yaml
client:
    port: 1337
```

Use `client_from_file` to create a `Client` object in Python:

```python
konduit_yaml_path = "../yaml/keras.yaml"
client = client_from_file(konduit_yaml_path)
```
{% endtab %}
{% endtabs %}

## Inference 

{% hint style="warning" %}
NDARRAY inputs to ModelSteps must be specified with a preceding `batchSize` dimension. For batches with a single observation, this can be done by using `numpy.expand_dims()` to add an additional dimension to your array. 
{% endhint %}

```python
input_array = np.random.uniform(size = [10])

prediction = client.predict({"input": np.expand_dims(input_array, axis=0)})

server.stop()
```

```python
print(prediction) 
prediction.shape
```

```text
[[[0.49911702 0.4983615  0.49773094 0.49721536 0.496801   0.49647287
   0.49621654 0.49601877 0.495868   0.4957543 ]
  [0.49977526 0.49945772 0.49912393 0.4988142  0.49854672 0.49832645
   0.49815112 0.49801567 0.49791327 0.49783763]
  [0.5001145  0.5001818  0.500209   0.50020653 0.50018466 0.50015163
   0.50011355 0.5000747  0.5000376  0.5000038 ]
  [0.499907   0.49980363 0.4997118  0.499638   0.49958205 0.49954122
   0.4995122  0.49949202 0.49947822 0.4994689 ]
  [0.49957263 0.4993314  0.49921444 0.49917603 0.4991838  0.499216
   0.4992586  0.49930325 0.4993452  0.4993822 ]
  [0.50122684 0.5020251  0.5025376  0.50286067 0.50305897 0.50317615
   0.5032414  0.50327414 0.50328714 0.50328875]]]





(1, 6, 10)
```



