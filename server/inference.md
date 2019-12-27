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
    serving_config=ServingConfig(http_port=port), 
    steps=[preprocessing_step, onnx_step]
)
```

### Create an InferenceConfig object 

```python
inference_config = InferenceConfig(
    serving_config=ServingConfig(http_port=port), 
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

## ServingConfig

For most configurations, the following arguments are sufficient:

* `http_port`: HTTP port of the Konduit Serving instance.
* `listen_host`: Host of the Konduit Serving instance. Defaults to `'localhost'`.
* `input_data_format`: Input data format: one of  `'NUMPY'`, `'JSON'`, `'ND4J'`, `'IMAGE'`or `'ARROW'`. Defaults to `NUMPY`. 
* `output_data_format`: Output data format: one of  `'NUMPY'`, `'JSON'`, `'ND4J'`, or `'ARROW'`. Defaults to `NUMPY`. 

The following arguments are optional: 

* `prediction_type`: Prediction type. This argument determines which "output adapter" is used to transform the output. Choose one of `'CLASSIFICATION'`, `'YOLO'`, `'SSD'`, `'RCNN'`, `'RAW'`, `'REGRESSION'`. The default prediction type is `'RAW'`: that is, no adapter is applied to the output. 
* `uploads_directory`: Directory to store file uploads. Defaults to `'file-uploads/'`.
* `log_timings`: Whether to log timings for this config. Defaults to False
* `metric_types`: The types of metrics logged for your `ServingConfig` can currently only be configured and extended from Java. Don't modify this property.

## `server.start()`

The `start` method initializes a Konduit Serving instance, and ends any previously started server that is still running. Set the `kill_existing_server` argument to `False` to change this behaviour. 

## `server.stop()`

The `stop` method ends the Konduit Serving process defined by the `Server` object.

## YAML configuration 

Refer to the [YAML configuration page](../yaml-configurations.md#serving). 

