---
description: Simple example to demonstrate Konduit-Serving
---

# Basic

In the first example, we’ll use normal operations as a model, but it is deploying on Konduit-Serving. You can give input and get the return of output from the server. This will make your model more straightforward to understand by humans as it can provide direct results.

### Viewing directory structure

Let’s run cells with bash in a sub-process by using cell magic command and view files in the current directory that will be used in this demonstration.

```bash
%%bash
echo "Current directory $(pwd)" && tree
```

The following files are present in our simple python script demo.

```text
Current directory /root/konduit/demos/0-python-simple
.
├── init_script.py
├── python-simple.ipynb
├── python.yaml
└── run_script.py

0 directories, 4 files
```

### Viewing Python script content

The scripts contain a simple initialization script for an add function which loads the main function in the `init_script.py` and executes the incoming input through `run_script.py`.

```bash
%%bash
less init_script.py
```

You’ll be able to see the following.

```text
def add_function(x, y):
    return x + y
```

Once again, let’s browse through the python script for the calling function from `init_script.py`.

```bash
%%bash
less run_script.py
```

You’ll notice the script only has a line of code.

```aspnet
c = add_function(a, b)
```

### Viewing the main configuration file

The main configuration should define the inputs as `a` and `b` and the output as `c`, just as we've showed in the `run_script.py`.

```bash
%%bash
less python.yaml
```

The YAML script file is as follows.

```aspnet
---
host: "0.0.0.0"
pipeline:
  steps:
  - '@type': "PYTHON"
    python_config:
      append_type: "BEFORE"
      extra_inputs: {}
      import_code_path: "init_script.py"
      python_code_path: "run_script.py"
      io_inputs:
        a:
          python_type: "float"
          secondary_type: "NONE"
          type: "DOUBLE"
        b:
          python_type: "float"
          secondary_type: "NONE"
          type: "DOUBLE"
      io_outputs:
        c:
          python_type: "float"
          secondary_type: "NONE"
          type: "DOUBLE"
      job_suffix: "konduit_job"
      python_config_type: "CONDA"
      python_path: "1"
      environment_name: "base"
      python_path_resolution: "STATIC"
      python_inputs: {}
      python_outputs: {}
      return_all_inputs: false
      setup_and_run: false
port: 8082
protocol: "HTTP"
```

### Using the configuration to start a server

Now we can use the `konduit serve` command to start the server in background with the given files and configurations.

```bash
%%bash
konduit serve -rwm --config python.yaml -id server --background
```

You’ll get the message like this.

```aspnet
Starting konduit server...
Expected classpath: /root/konduit/bin/../konduit.jar
INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command_logs/server.log -Dlogback.configurationFile=/tmp/logback-run_command_13ccd5e27dfe43b1.xml -cp /root/konduit/bin/../konduit.jar ai.konduit.serving.cli.launcher.KonduitServingLauncher run --instances 1 -s inference -c python.yaml -Dserving.id=server
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs server'
```

### Listing the servers

We can list the created servers with `konduit list` command

```bash
%%bash
konduit list
```

The ID’s server lists like below, giving the status of the server.

```aspnet
Listing konduit servers...

 #   | ID                             | TYPE       | URL                  | PID     | STATUS     
 1   | server                         | inference  | 0.0.0.0:8082         | 421     | started    
```

### Viewing logs

Logs can be viewed for the server with an ID of `server` through running `konduit logs server ..` command.

```bash
%%bash
konduit logs server --lines 1000
```

Logs output of started server:

```aspnet
09:44:17.852 [main] INFO  a.k.s.c.l.command.KonduitRunCommand - Processing configuration: /root/konduit/demos/0-python-simple/python.yaml
.
.
.
09:44:19.436 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

09:44:19.436 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
.
.
.
09:44:19.589 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: '0.0.0.0'
09:44:19.589 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 8082 with 1 pipeline steps
```

### Sending inputs

Now we’ll be able to send the inputs for inferring the output.

```bash
%%bash
konduit predict server '{"a":1,"b":2}'
```

The output result of the function deployed from the server.

```aspnet
{
  "c" : 3.0
}
```

### Stopping the server

Stop the server by giving the ID’s we want to terminate. 

```bash
%%bash
konduit stop server
```

Status of the server will be printed out as below.

```aspnet
Stopping konduit server 'server'
Application 'server' terminated with status 0
```

As you can see from this example, we only use a simple function to deploy in Konduit-Serving. Next, we'll deploy the model in Konduit-Serving.

