---
description: >-
  A Konduit Serving instance is configured by a Server object with a fully
  configured list of pipeline steps.
---

# Server

After the Server object is configured, you can use the `.start()` and `.stop()` methods to initialize and stop the Serving instance.

```python
from konduit.server import Server 

server = Server(
    inference_config=None,
    serving_config=None,
    steps=None,
    extra_start_args="-Xmx8g",
    config_path="config.json",
    jar_path=None,
    pid_file_path="konduit-serving.pid",
    start_timeout=120,
 );
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

## Additional arguments

* `extra_start_args`: Java Virtual Machine \(JVM\) arguments. In this case, `-Xmx8g` specifies that the maximum memory allocation for the JVM is 8GB. 
* `config_path`: path to write the config object to \(as json\)
* `jar_path`: path to the konduit uberjar. If `None`, defaults to the `KONDUIT_JAR_PATH` environment variable, or `~/.konduit/konduit-serving` if `KONDUIT_JAR_PATH` is not available.
* `pid_file_path`: path to write the process ID to, as a text file. 
* `start_timeout`: time to wait for the server to timeout when starting the server instance. 

## `server.start()`

The `start` method initializes a Konduit Serving instance, and ends any previously started server that is still running. Set the `kill_existing_server` argument to `False` to change this behaviour. 

## `server.stop()`

The `stop` method ends the Konduit Serving process defined by the `Server` object.

## YAML configuration 

Refer to the [YAML configuration page](../yaml-configurations.md#serving). 

