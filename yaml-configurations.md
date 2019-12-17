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
* `input_data_format` and `output_data_format`: specify one of the following: JSON, NUMPY, ARROW, IMAGE
* `log_timings`: specify True to log timings 
* `extra_start_args`: Java Virtual Machine \(JVM\) arguments. In this case, `-Xmx8g` specifies that the maximum memory allocation for the JVM is 8GB. 

Refer to the [Server](server/inference.md) documentation for details.

### Client

Refer to the [Client](client/python-client.md) documentation for details.

* `input_names`, `output_names`: names of the first and final nodes of the Konduit Serving pipeline configuration defined in the Server. These arguments are typically inherited from the Server when initialized. 
* `input_data_format`, `output_data_format`, `return_output_data_format`: One of the following: JSON, NUMPY, ARROW, IMAGE. `input_data_format` and `output_data_format` refer to the format of the server's input and output, whereas `return_output_data_format` specifies the data format returned to the client. 
* `port`: specify the same HTTP port as the Server. 

### Steps

Detailed instructions on configuring steps are available in the [Examples](https://serving.oss.konduit.ai/examples). 

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

