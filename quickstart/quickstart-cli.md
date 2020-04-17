---
description: A brief overview of konduit-serving command line interface.
---

# Command line interface \(CLI\)

[konduit-serving](https://github.com/KonduitAI/konduit-serving) comes with a handy CLI that you can use to manage your serving instances. Konduit CLI comes with the [konduit](https://pypi.org/project/konduit/) pip package. You can use the `konduit` command line tool by typing the following on the terminal:

```text
konduit --help
```

which should prompt all currently available commands:

```text
Usage: konduit [COMMAND] [OPTIONS] [arg...]

Commands:
    config    A helper command for creating JSON for inference configuration
    inspect   Inspect the details of a particular konduit server.
    list      Lists the running konduit servers.
    logs      View the logs of a particular konduit server
    predict   Run inference on konduit servers using given inputs
    serve     Start a konduit server application
    stop      Stop a running konduit server
    version   Displays konduit-serving version.

Run 'konduit COMMAND --help' for more information on a command.
```

For more help on individual commands you can run `konduit [COMMAND] --help`. For example:

```text
konduit serve --help
```

to view the usage of `serve` command:

```text
Usage: konduit serve [-b] [-cp <classpath>] -c <server-config>  [-i <instances>]
       [-jo <value>]  [-s <type>] [-id <value>]

Start a konduit server application

Start a konduit server application. The application is identified with an id
that can be set using the `--serving-id` or `-id` option. The application can be
stopped with the `stop` command. This command takes the `run` command
parameters. To see the run command parameters, execute `run --help`

Example usages:
--------------
- Starts a server in the foreground with an id of 'inf_server' using
'config.json' as configuration file:
$ konduit serve -id inf_server -c config.json

- Starts a server in the background with an id of 'inf_server' using
'config.json' as configuration file:
$ konduit serve -id inf_server -c config.json -b
--------------

Options and Arguments:
 -b,--background               Runs the process in the background, if set.
 -cp,--classpath <classpath>   Provides an extra classpath to be used for the
                               verticle deployment.
 -c,--config <server-config>   Specifies configuration that should be provided
                               to the verticle. <config> should reference either
                               a text file containing a valid JSON object which
                               represents the configuration OR be a JSON string.
 -i,--instances <instances>    Specifies how many instances of the server will
                               be deployed. Defaults to 1.
 -jo,--java-opts <value>       Java Virtual Machine options to pass to the
                               spawned process such as "-Xmx1G -Xms256m
                               -XX:MaxPermSize=256m". If not set the `JAVA_OPTS`
                               environment variable is used.
 -s,--service <type>           Service type that needs to be deployed. Defaults
                               to "inference"
 -id,--serving-id <value>      Id of the serving process. This will be visible
                               in the 'list' command. This id can be used to
                               call 'predict' and 'stop' commands on the running
                               servers. If not given then an 8 character UUID is
                               created automatically.
```

The `--help` argument for the individual commands gives you a quick summary and a detailed description of what the command is about along with a few examples of common usage patterns. 

## **Example Workflow**

The CLI provides a handy`predict` command that returns predictions from a model server. To use the `predict` command, 

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



