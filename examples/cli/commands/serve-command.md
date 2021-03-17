---
description: Examples of CLI with serve command
---

# Serve Command

The`serve` command is used to deploy a Konduit-Serving application which must be followed by configuration file either in JSON or YAML. There are a few other options to use with `serve` command which can be seen through the `konduit serve --help` command.

#### Examples

The server identifies with an id that can be set using the `--serving-id` or `-id` option, for example:

```bash
$ konduit serve -id inf_server -c config.json
```

You'll be able to see the following output \(trimmed for brevity\):

```bash
.
.
.
16:52:08.812 [vert.x-worker-thread-0] INFO  o.d.nn.multilayer.MultiLayerNetwork - Starting MultiLayerNetwork with WorkspaceModes set to [training: ENABLED; inference: ENABLED], cacheMode set to [NONE]
16:52:08.838 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

16:52:08.838 [vert.x-worker-thread-0] INFO  a.k.s.v.verticle.InferenceVerticle - Pending server start, please wait...
.
.
.
.
16:52:09.052 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: 'localhost'
16:52:09.052 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 42823 with 4 pipeline steps
```

To start with a specific profile you can run the following command which starts a server in the foreground with an id of 'inf\_server' using 'config.json' as configuration file and GPU profile:

```bash
$ konduit serve -id inf_server -c config.json -p GPU
```

To learn more about profiles navigate to the following section:

{% page-ref page="profile-command.md" %}

You’ll see output like this, although the version number, etc. may be different based on your local machine:

```bash
Starting konduit server...
.
.
.
16:06:49.704 [vert.x-worker-thread-0] INFO  org.nd4j.nativeblas.NativeOpsHolder - Number of threads used for linear algebra: 32
16:06:49.722 [vert.x-worker-thread-0] INFO  o.n.l.a.o.e.DefaultOpExecutioner - Backend used: [CUDA]; OS: [Linux]
16:06:49.722 [vert.x-worker-thread-0] INFO  o.n.l.a.o.e.DefaultOpExecutioner - Cores: [12]; Memory: [5.2GB];
16:06:49.722 [vert.x-worker-thread-0] INFO  o.n.l.a.o.e.DefaultOpExecutioner - Blas vendor: [CUBLAS]
16:06:49.729 [vert.x-worker-thread-0] INFO  o.nd4j.linalg.jcublas.JCublasBackend - ND4J CUDA build version: 11.0.221
16:06:49.730 [vert.x-worker-thread-0] INFO  o.nd4j.linalg.jcublas.JCublasBackend - CUDA device 0: [GeForce RTX 2060]; cc: [7.5]; Total memory: [6222839808]
16:06:49.731 [vert.x-worker-thread-0] INFO  o.nd4j.linalg.jcublas.JCublasBackend - Backend build information:
 GCC: "9.3.0"
STD version: 201402L
CUDA: 11.0.221
DEFAULT_ENGINE: samediff::ENGINE_CUDA
HAVE_FLATBUFFERS
.
.
```

Starts a server in the background with an id of 'inf\_server' using 'config.yaml' as configuration file without creating the manifest jar file before launching the server:

```bash
$ konduit serve -id inf_server -c config.yaml -b -rwm
```

The output will be like this showing the server is running in background:

```bash
Starting konduit server...
Expected classpath: /opt/konduit/bin/../konduit.jar
INFO: Running command /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java -Dkonduit.logs.file.path=/home/zulfadzli/.konduit-serving/command_logs/inf_server.log -Dlogback.configurationFile=/opt/konduit/bin/../conf/logback-run_command.xml -cp /opt/konduit/bin/../konduit.jar ai.konduit.serving.cli.launcher.KonduitServingLauncher run --instances 1 -s inference -c config.json -Dserving.id=inf_server
For server status, execute: 'konduit list'
For logs, execute: 'konduit logs inf_server'
```

