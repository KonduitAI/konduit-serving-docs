---
description: >-
  Konduit Serving supports data transformations defined by the DataVec
  vectorization and ETL library.
---

# DataVec

```python
from konduit import TransformProcessStep, ServingConfig
from konduit.server import Server
from konduit.client import Client
from konduit.utils import is_port_in_use

from pydatavec import Schema, TransformProcess

from utils import load_java_tp

import numpy as np 
import random
import time
import json
import os
```

DataVec transformations can be defined in Python using the [PyDataVec](https://github.com/eclipse/deeplearning4j/tree/master/pydatavec) package, which can be installed from PyPi:

```text
pip install pydatavec
```

Using PyDataVec requires [Docker](https://docs.docker.com/v17.09/engine/installation/#supported-platforms). For Windows 10 Home edition users, note that Docker Toolbox is not supported.

Run the following cell to check that your Docker installation is successful:

```python
!docker run hello-world
```

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Data transformations with PyDataVec

### **Schema \(**[**source**](https://github.com/eclipse/deeplearning4j/blob/master/pydatavec/pydatavec/schema.py)**\)**

A `Schema` specifies the structure of your data. In DataVec, a `TransformProcess` requires the `Schema` of the data to be specified.

`Schema` objects have a number of methods that define different data types for columns: `add_string_column()`, `add_integer_column()`, `add_long_column()`, `add_float_column()`, `add_double_column()` and `add_categorical_column()`.

### **TransformProcess \(**[**source**](https://github.com/eclipse/deeplearning4j/blob/master/pydatavec/pydatavec/transform_process.py)**\)**

`TransformProcess` provides a number of methods to manipulate your data. The following methods are available in the Python API:

* Reduce the number of rows: `filter()`
* General data transformations: `replace()`, 
* Type casting: `string_to_time()`, `derive_column_from_time()`, `categorical_to_integer()`, 
* Combining/reducing the values in each column: `reduce()`
* String operations: `append_string()`, `lower()`, `upper()`, `concat()`, `remove_white_spaces()`, `replace_empty_string()`, `replace_string()`, `map_string()`
* Column selection/renaming: `remove()`, `remove_columns_except()`, `rename_column()`
* One-hot encoding: `one_hot()`

In this short example, we append the string `two` to the end of values in the string column `first`.

```python
schema = Schema()
schema.add_string_column("first")

tp = TransformProcess(schema)
tp.append_string("first", "two")
```

The `TransformProcess` configuration has to be converted into JSON format to be passed to Konduit Serving.

```python
java_tp = tp.to_java()
tp_json = java_tp.toJson()
load_java_tp(tp_json)
as_python_json = json.loads(tp_json)
```

## Configure the step

The `TransformProcess` can now be defined in the Konduit Serving configuration with a `TransformProcessStep`. Here, we

* **configure the inputs and outputs**: the schema, column names and data types should be defined here. 
* **declare the `TransformProcess`** using the `.transform_process()` method. 

Note that `Schema` data types are not defined in the same way as `PythonStep` data types. See the [source](https://github.com/KonduitAI/konduit-serving/blob/78851701004ebb3dbf079889d46b79a9db8fac60/konduit-serving-api/src/main/java/ai/konduit/serving/util/SchemaTypeUtils.java#L154-L195) for a complete list of supported Schema data types:

* `NDArray`
* `String`
* `Boolean`
* `Categorical`
* `Float`
* `Double`
* `Integer`
* `Long`
* `Bytes`

You should define the Schema data types in `TransformProcessStep()` as strings.

```python
transform_step = (TransformProcessStep()
                  .set_input(schema=None, 
                             column_names=["first"], 
                             types=["String"])
                  .set_output(schema=None, 
                              column_names=["first"], 
                              types=["String"])
                  .transform_process(as_python_json))
```

## Configure the server

Configure the Server using `ServingConfig` to define the port using the `http_port` argument and data formats using the `input_data_type` and `output_data_type` arguments.

```python
port = np.random.randint(1000, 65535)
serving_config = ServingConfig(
    http_port=port,
    input_data_format='JSON',
    output_data_format='JSON',
)

server = Server(
    serving_config=serving_config,
    steps=[transform_step]
)
```

The complete configuration is as follows:

```python
server.config.as_dict()
```

```text
{'@type': 'InferenceConfiguration',
 'steps': [{'@type': 'TransformProcessStep',
   'inputSchemas': {'default': ['String']},
   'outputSchemas': {'default': ['String']},
   'inputNames': ['default'],
   'outputNames': ['default'],
   'inputColumnNames': {'default': ['first']},
   'outputColumnNames': {'default': ['first']},
   'transformProcesses': {'default': {'actionList': [{'transform': {'@class': 'org.datavec.api.transform.transform.string.AppendStringColumnTransform',
        'columnName': 'first',
        'toAppend': 'two'}}],
     'initialSchema': {'@class': 'org.datavec.api.transform.schema.Schema',
      'columns': [{'@class': 'org.datavec.api.transform.metadata.StringMetaData',
        'name': 'first'}]}}}}],
 'servingConfig': {'@type': 'ServingConfig',
  'httpPort': 47964,
  'inputDataFormat': 'JSON',
  'outputDataFormat': 'JSON',
  'logTimings': True}}
```

## Start the server 

```python
server.start()
```

## Configure the client

The `Client` should be configured to match the Konduit Serving instance.

* `input_names` and `output_names` should match the columns in the DataVec schema 
* Data formats should be defined as one of the following: `JSON`, `RAW`, `ARROW`, `IMAGE`, `NUMPY` 
* As this example is run on a local computer, the server is located at host `'http://localhost'` and port `port`. Since `'http://localhost'` is the default argument, we can omit it: 

```python
client = Client(input_names=["first"],
                output_names=["first"],
                return_output_data_format='JSON',
                input_data_format='JSON',
                output_data_format='RAW',
                port=port)
```

```text
Starting server..

Server has started successfully.
```

## Inference

Finally, we run the Konduit Serving instance. Recall that the `TransformProcessStep()` appends a string `two` to strings in the column `first`:

```python
data_input = {'first': 'value'}
predicted = client.predict(data_input)
print(predicted)
server.stop()
```

```text
{'first': 'valuetwo'}
```

