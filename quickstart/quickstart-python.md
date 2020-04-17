# Python SDK

A Konduit Serving instance can be created by: 

1. creating a Python object of the `Server` class using 
   1. the `Server()` function; or 
   2. the `server_from_file()` function from the `konduit.load` module; and 
2. starting the server using the `.start()` method of the `Server` object created in step 1. 

We will use the `server_from_file()` function to configure Konduit Serving in this example.

In Python, specify the path to your configuration in `konduit_yaml_path`: 

```python
konduit_yaml_path = "hello-world.yaml"
```

Initialize a Konduit Serving instance with the following code: 

```python
from konduit.load import server_from_file 
server = server_from_file(konduit_yaml_path)
server.start()
```

Note that the file also contains Client configuration. To create a `Client` object, use the `client_from_file()` function from the `konduit.load` module:

```python
from konduit.load import client_from_file 
client = client_from_file(konduit_yaml_path)
```

The `Client` class provides a `.predict()` method that sends data to the Serving instance. First, create some sample data as a NumPy array:

```python
import numpy as np 
data_input = np.ones(5)
```

Assuming your data is declared in the `data_input` object, data can be passed to `client` for prediction using:

```python
client.predict(data_input)
```

The `.predict()` method takes a single argument `data_input` which is typically a dictionary. A NumPy array can be directly passed to the `.predict()` method if the input name is `default`.

## Next steps 

To build configurations using the YAML format, check out the YAML configurations page: 

{% page-ref page="../yaml-configurations.md" %}

YAML configurations are sufficient for most use cases. In particular, if your use case: 

* does not involve DataVec transformations, 
* for Python steps: has one transformation script at each pipeline step,

then you should use a YAML configuration. 

For more complex configurations, you should use the Python SDK. To build configurations with Python steps, start with the Python pipeline steps page:

{% page-ref page="../steps/python.md" %}

To build configurations in Python with TensorFlow, DL4J and Keras models using DL4J and JavaCPP Presets, refer to the example for the respective framework:

{% page-ref page="../examples/python/tensorflow-model-serving/" %}

{% page-ref page="../examples/python/keras.md" %}

To build ETL processes into your serving pipeline, refer to the DataVec example: 

{% page-ref page="../examples/python/datavec.md" %}



