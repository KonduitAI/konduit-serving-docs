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

## YAML configuration 

Refer to the [YAML configuration page](../yaml-configurations.md#serving). 

