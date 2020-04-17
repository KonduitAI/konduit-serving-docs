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

Following is an example workflow of how to use the `konduit` CLI for serving an [ImageLoadingStep](../steps/image-loading-pipeline-steps.md).

#### 1. Create a configuration

Before running any server, you'll have to configure a json configuration for the serving pipeline. The `config` command is a very handy tool to create a baseline configuration that you can edit later based on your requirements. In this workflow, you'll see how to create a basic configuration for reading image file and return the loaded image in `JSON` format with the `predict` command. To create an image configuration you can run the `config` command as follows:

```text
konduit config -t image
```

You'll see the following output from it \(might differ based on your local environment\):

```text
{
  "servingConfig" : {
    "createLoggingEndpoints" : false,
    "httpPort" : 0,
    "listenHost" : "localhost",
    "logTimings" : false,
    "metricTypes" : [ "CLASS_LOADER", "JVM_MEMORY", "JVM_GC", "PROCESSOR", "JVM_THREAD", "LOGGING_METRICS", "NATIVE" ],
    "metricsConfigurations" : [ ],
    "outputDataFormat" : "JSON",
    "uploadsDirectory" : "C:\\Users\\konduit\\AppData\\Local\\Temp"
  },
  "steps" : [ {
    "@type" : "ImageLoadingStep",
    "dimensionsConfigs" : { },
    "imageProcessingInitialLayout" : "NCHW",
    "imageProcessingRequiredLayout" : "NCHW",
    "imageTransformProcesses" : { },
    "inputColumnNames" : { },
    "inputNames" : [ "default" ],
    "inputSchemas" : { },
    "originalImageHeight" : 0,
    "originalImageWidth" : 0,
    "outputColumnNames" : { },
    "outputNames" : [ "default" ],
    "outputSchemas" : { },
    "updateOrderingBeforeTransform" : false
  } ]
}
```

{% hint style="warning" %}
Note

A port equal to `0` means that a random port will be selected for the server when it's run.
{% endhint %}

To save the configuration in a file, you can run: 

```bash
konduit config -t image -o image-config.json
```

You'll see the following output from it: 

```text
Config file created successfully at C:\Users\konduit\image-config.json
```

#### 2. Start a server

For starting the server, you can use the `serve` command:

```bash
konduit serve -b -id image-server -c image-config.json
```

This will start a konduit server with the given configuration in the background.

#### 3. List the running servers

To view the running servers, you can use the `list` command:

```bash
konduit list
```

You can an output like the following:

```text
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS
 1   | image-server                   | inference  | localhost:58663      | 23756   | started

```

{% hint style="warning" %}
Note

You might see a different port based on your running environment.
{% endhint %}

#### 4. View the logs

You can view the logs of the running server with the `logs` command:

```bash
konduit logs image-server
```

which will show you the following logs for the running server \(truncated for brevity\):

```text
16:36:09.491 [main] INFO  ai.konduit.serving.util.LogUtils - Logging file at: C:\Users\shams\.konduit-serving\command_logs\image-server.log
16:36:09.631 [main] INFO  a.k.s.l.KonduitServingLauncher - Setup micro meter options.
16:36:10.154 [main] INFO  a.k.s.l.command.KonduitRunCommand - Starting konduit server with an id of 'image-server'
16:36:10.397 [vert.x-eventloop-thread-0] INFO  a.k.s.routers.PipelineRouteDefiner - Using metrics registry io.micrometer.prometheus.PrometheusMeterRegistry for inference
16:36:10.760 [vert.x-eventloop-thread-0] DEBUG o.h.common.AbstractCentralProcessor - Oracle MXBean detected.
16:36:10.811 [vert.x-eventloop-thread-0] DEBUG o.d.windows.PerfCounterWildcardQuery - Localized Processor to Processor
16:36:10.852 [vert.x-eventloop-thread-0] DEBUG o.h.p.w.WindowsCentralProcessor - Initialized Processor
16:36:11.896 [vert.x-eventloop-thread-0] DEBUG o.s.o.windows.WindowsOSVersionInfoEx - Initialized OSVersionInfoEx  .
 .
 .
 .
16:36:15.164 [vert.x-eventloop-thread-0] INFO  a.k.s.v.inference.InferenceVerticle - Inference server is listening on host: 'localhost'
16:36:15.164 [vert.x-eventloop-thread-0] INFO  a.k.s.v.inference.InferenceVerticle - Inference server started on port 58663 with 1 pipeline steps
16:36:15.164 [vert.x-eventloop-thread-1] INFO  i.v.c.i.l.c.VertxIsolatedDeployer - Succeeded in deploying verticle
```

#### 5. Running predictions

After a server is successfully started you can use the `predict` command to run inferences on the server:

```bash
konduit predict -it IMAGE image-server C:\Users\konduit\5_10x10.png
```

#### 6. Stop a server

Finally for stopping a server you can use the `stop` command:

```bash
konduit stop image-server
```



