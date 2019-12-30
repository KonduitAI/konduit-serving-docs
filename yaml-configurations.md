---
description: >-
  Konduit Serving supports specifying server configurations as YAML files. This
  allows you to serve simple server configurations using the Konduit Python CLI
  and the konduit.load module.
---

# YAML configurations

## YAML components

A Konduit Serving YAML configuration file has three top-level entities: 

1. `serving`
2. `steps`
3. `client`

The following is a sample YAML file for serving a Python script located at `simple.py` which takes a NumPy array `first` as input and a NumPy array `second` as output:

```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
  log_timings: True
  extra_start_args: -Xmx8g
steps:
  python_step:
    type: PYTHON
    python_path: .
    python_code_path: ./simple.py
    python_inputs:
      first: NDARRAY
    python_outputs:
      second: NDARRAY
client:
    port: 1337
```

### Serving

The server configuration takes the following arguments:

* `http_port`: specify the port number 
* `listen_host`: the host of the Konduit Serving instance. Defaults to `http://localhost`. 
* `input_data_format` and `output_data_format`: specify one of the following: JSON, NUMPY, ARROW, IMAGE

Additional arguments include: 

* `prediction_type`: Determines which "output adapter" is used to transform the output. Choose one of `'CLASSIFICATION'`, `YOLO'`, `'SSD'`, `'RCNN'` or `'RAW'` \(default, no transformation\). 
* `uploads_directory`: Directory to store file uploads. Defaults to `'file-uploads/'`.
* `jar_path`: Path to the Konduit Serving uberjar. Defaults to the `KONDUIT_JAR_PATH` environment variable, or if unavailable, `~/.konduit/konduit-serving/konduit.jar`. 
* `log_timings`: Whether to log timings for this config. Defaults to `False`.
* `extra_start_args`: Java Virtual Machine \(JVM\) arguments. In this case, `-Xmx8g` specifies that the maximum memory allocation for the JVM is 8GB. 

Refer to the [Server](server/inference.md) documentation for other arguments.

### Client

The client configuration takes the following arguments:

* `port`: specify the same HTTP port as the Server. 
* `host`: defaults to `http://localhost`. Ignore this argument for local instances.

Typically it is sufficient to specify the `port` and `host`as the remaining attributes are obtained from the Server. Refer to the [Client](client/python-client.md) documentation for details.

* `input_names`, `output_names`: names of the first and final nodes of the Konduit Serving pipeline configuration defined in the Server. 
* `input_data_format`, `output_data_format`: One of the following: JSON, NUMPY, ARROW, IMAGE, ND4J. `input_data_format` and `output_data_format` refer to the format of the server's input and output.

### Steps

#### Python steps 

Python steps run code specified in the `python_code` or `python_code_path` argument, whichever is specified. Python steps defined in the YAML configuration default to input name and output name `"default"`.  

Importantly, the `python_inputs` argument maps each input and output variable to the respective data type. Accepted data types are`"INT"`, `"STR"`, `"FLOAT"`, `"BOOL"`, `"NDARRAY"`

```yaml
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
```

If no Python path is specified, NumPy will still be available in the environment where the Python step is run. 

To further customize Python steps, refer to the YAML configuration section of the Python pipeline steps page. 

{% page-ref page="steps/python.md" %}

A more comprehensive example is available on the following page: 

{% page-ref page="examples/python/onnx.md" %}

#### Model steps 

Use model steps when you want to use pre-packaged modules such as TensorFlow, DL4J and PMML for inference. 

```yaml
steps:
  tensorflow_step:
    type: TENSORFLOW
    model_loading_path: ../data/mnist/mnist_2.0.0.pb
    input_names:
      - input_layer
    output_names:
      - output_layer/Softmax
    input_data_types:
      input_layer: FLOAT
```

The following parameters should be specified:

* `type`: one of `"TENSORFLOW"`, `"KERAS"`, `"COMPUTATION_GRAPH"`, `"MULTI_LAYER_NETWORK"`, `"PMML"`, `"SAMEDIFF"`;
* `model_loading_path`: location of your model file; 
* `input_names`: list of the names of input nodes of your model file;
* `output_names`: list of the names of output nodes of your model file;
* `input_data_types`: map each of the input nodes to one of the following data types using the input names as keys: `"INT"`, `"STR"`, `"FLOAT"`, `"BOOL"`, `"NDARRAY"`. 

Refer to the model-specific example for details on configuring model steps. 

{% page-ref page="examples/python/tensorflow-model-serving/" %}

{% page-ref page="examples/python/keras.md" %}

{% page-ref page="examples/python/dl4j.md" %}

## Usage

On the **server**, start a Konduit Serving instance by: 

1. creating a Server object using `server_from_file`, 
2. starting the server using the `.start()` method. 

```python
from konduit.load import server_from_file

konduit_yaml_path = "../yaml/simple.yaml"

server = server_from_file(konduit_yaml_path)
server.start()
```

```text
Starting server..

Server has started successfully.
```

After the server has started, on the **client**:

1. create a Client object using `client_from_file`; and
2. use the `.predict()` method to perform inference on a NumPy array \(note that the input name of this Server configuration is `default`, therefore we can pass a NumPy array directly to the `.predict()` method.\).

```python
import numpy as np 
import os
from konduit.load import client_from_file

konduit_yaml_path = "../yaml/simple.yaml"
input_arr = np.array(33)

client = client_from_file(konduit_yaml_path)
print(client.predict(input_arr))
```

```text
[35]
```

Finally, stop the server with the `.stop()` method: 

```python
server.stop()
```

This can also be run in the **command line**. In the root folder of [konduit-serving-examples](https://github.com/KonduitAI/konduit-serving-examples), initialize the Konduit Serving instance with 

```bash
konduit serve --config yaml/simple.yaml
```

Send the NPY file to the server for inference with 

```bash
konduit predict-numpy --config yaml/simple.yaml --numpy_data data/simple/input_arr.npy
```

and finally, stop the server with 

```bash
konduit stop-server --config yaml/simple.yaml
```

## Resources 

Some resources on the YAML format:

* [https://gettaurus.org/docs/YAMLTutorial/](https://gettaurus.org/docs/YAMLTutorial/)
* [https://docs.saltstack.com/en/latest/topics/yaml/](https://docs.saltstack.com/en/latest/topics/yaml/)
* [http://jessenoller.com/blog/2009/04/13/yaml-aint-markup-language-completely-different](http://jessenoller.com/blog/2009/04/13/yaml-aint-markup-language-completely-different)

