---
description: >-
  The Client class allows a client to obtain outputs from a Konduit Serving
  instance given a named set of inputs via the predict() method.
---

# Python client configuration

```python
from konduit.client import Client 

Client(
    timeout=60,
    input_data_format='NUMPY',
    output_data_format='NUMPY',
    input_names=['default'],
    output_names=['default'],
    host='http://localhost',
    port=None, 
    prediction_type=None
)
```

### Data formats

Data formats define how data is transported between the Client and the Konduit Serving instance. In addition to JSON, Konduit Serving also supports NUMPY, ARROW, RAW and IMAGE data formats. Specify data formats as strings. 

* `input_data_format` defines the data format of inputs sent to the server via the `predict()` method.
* `output_data_format`defines the data format returned by the API endpoint.

If you want to convert the result of your endpoint to another data format on return, change the `convert_to_format` attribute of the `Client` object: 

```python
client = Client(port=3226)
client.convert_to_format = "ARROW"
```

### Input and output names

Both the `input_names` and `output_names` arguments accept a list of strings. Input and output names are defined by the inputs and outputs to the first and last pipeline steps in the Konduit Serving pipeline configuration respectively. 

For `ModelStep`, input and output names should be configured when defining the model for training, or may need to be obtained by inspecting the model file. See the examples for details.  

For `PythonStep`, input names are defined in the `step()` method to a `PythonStep` object.

### Other arguments 

* `timeout`: Integer. Defaults to 60 \(seconds\). 
* `host`: String. If the model is hosted locally, the host should be specified as `http://localhost` \(the default argument\) 
* `port`: Integer. 
* `prediction_type`: Defaults to `"RAW"`. One of `"CLASSIFICATION"`, `"YOLO"`, `"SSD"`, `"RCNN",` `"RAW"`, `"REGRESSION"`.

### `.predict()`method

The `.predict()`method takes a dictionary with`input_names` as keys and the data inputs as values. 

## Example

Assume a Server object has been fully configured as `server`. Start the server:

```python
server.start()
```

Define a Client:

```python
client = Client(
    input_data_format='NUMPY',
    output_data_format="NUMPY",
    return_data_output_format='NUMPY',
    input_names=["input1"],
    host='http://localhost', 
    port=port
)
```

Request for a prediction: 

```python
client.predict({"input1": np.ones(5)})
server.stop()
```

## YAML configuration 

Refer to the [YAML configuration page](../yaml-configurations.md#client). 

