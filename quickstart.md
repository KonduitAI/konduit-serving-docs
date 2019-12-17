# Quickstart

## Command line interface \(CLI\)

You can test the `konduit` command line tool by typing the following into your command line:

```text
konduit --help
```

which should prompt all currently available commands:

```text
Usage: konduit [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  build          Build the underlying konduit.jar (again).
  init           Initialize the konduit-python CLI.
  predict-numpy  Get predictions for your pipeline from numpy input.
  serve          Serve a pipeline from a konduit.yaml
  stop-server    Stop the Konduit server associated with a given config...
```

For more help on individual commands you can run

```text
konduit serve --help
```

to get help for the `serve` command \(and all others in a similar way\)

```text
Usage: konduit serve [OPTIONS]

  Serve a pipeline from a konduit.yaml

Options:
  --config TEXT          Relative or absolute path to your konduit serving YAML
                       file.
  --start_server TEXT  Whether to start the server instance after 
                       initialization.
  --help               Show this message and exit.
```

The CLI provides a handy command `predict-numpy` that returns predictions from a model server. To use the `predict-numpy` command, 

1. the input name must be `default`, and 
2. a [**NumPy array**](https://docs.scipy.org/doc/numpy/reference/arrays.html) stored in the [NPY format](https://numpy.org/devdocs/reference/generated/numpy.lib.format.html) is supplied as input. 

To initialize the server, run the following command in the root folder of [konduit-serving-examples](https://github.com/KonduitAI/konduit-serving-examples/):

```bash
konduit serve --config yaml/simple.yaml
```

Once the server has started, run `predict-numpy` to obtain the predicted output given the location of the NumPy array saved as a [NumPy `.npy` file](https://docs.scipy.org/doc/numpy/reference/generated/numpy.lib.format.html):

```bash
konduit predict-numpy --config yaml/simple.yaml --numpy_data data/bert/input-0.npy
```

Finally, to stop the server, run the `stop-server` command:

```bash
konduit stop-server --config yaml/simple.yaml
```

## Python SDK

A Konduit Serving instance can be created by: 

1. creating a Python object of the `Server` class using 
   1. the `Server()` function; or 
   2. the `server_from_file` function from the `konduit.load` module; and 
2. starting the server using the `.start()` method of the `Server` object created in step 1. 

Save the configuration below as a text file named `hello-world.yml`

```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
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
client:
    port: 1337
```

In Python, specify the path to your configuration in `konduit_yaml_path`: 

```python
konduit_yaml_path = "hello-world.yml"
```

Initialize a Konduit Serving instance with the following code: 

```python
from konduit.load import server_from_file 
server = server_from_file(konduit_yaml_path)
server.start()
```

Note that the file also contains Client configuration. To create a `Client` object, use the `client_from_file` function from the `konduit.load` module:

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

The `.predict()` method takes a single argument`data_input` which is typically a dictionary. A NumPy array can be directly passed to the `.predict()` method if the input name is `default`.

## 

