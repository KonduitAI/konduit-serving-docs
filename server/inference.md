---
description: >-
  A Konduit Serving instance is configured by a Server object with a fully
  configured list of pipeline steps.
---

# Server

After the Server object is configured, you can use the `.start()` and `.stop()` methods to initialize and stop the Serving instance.

```python
server = Server(
    inference_config=None,
    serving_config=None,
    steps=None,
    extra_start_args='-Xmx8g',
    config_path='config.json',
    jar_path='konduit.jar',
)
```

## Configuring a Server object

There are two options for configuring a Server object: 

### Directly define a ServingConfig and a list of steps to Server

```python
server = Server(
    serving_config=ServingConfiguration(http_port=port), 
    steps=[preprocessing_step, onnx_step]
)
```

### Create an InferenceConfiguration object 

```python
inference_config = InferenceConfiguration(
    serving_config=ServingConfiguration(http_port=port), 
    steps=[preprocessing_step, onnx_step]
)

server = Server(
    inference_config=inference_config
)
```

Configurations are stored as dictionaries. You can access a server's configuration as a Dictionary object using the `server.config.as_dict()` method. 

## `server.start()`

The `start` method initializes a Konduit Serving instance, and ends any previously started server that is still running. Set the `kill_existing_server` argument to `False` to change this behaviour. 

## `server.stop()`

The `stop` method ends the Konduit Serving process defined by the `Server` object.

## YAML configuration 

Refer to the [YAML configuration page](../yaml-configurations.md#serving). 
