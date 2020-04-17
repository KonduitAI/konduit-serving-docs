# Command line interface \(CLI\)

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
konduit serve --config hello-world.yaml
```

Once the server has started, run `predict-numpy` to obtain the predicted output given the location of the NumPy array saved as a [NumPy `.npy` file](https://docs.scipy.org/doc/numpy/reference/generated/numpy.lib.format.html):

```bash
konduit predict-numpy --config hello-world.yaml --numpy_data data/bert/input-0.npy
```

Finally, to stop the server, run the `stop-server` command:

```bash
konduit stop-server --config hello-world.yaml
```

## Next steps

Check out the YAML configurations page to write configurations for use with the CLI. 

{% page-ref page="../yaml-configurations.md" %}



