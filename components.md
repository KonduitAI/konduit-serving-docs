---
description: Describes the components that make up Konduit-Serving
---

# Components

## **Components**

Konduit-Serving has many internal components that work together to achieve the desired result. Following are the main concepts and components that make up Konduit-Serving:

1. CLI 
2. Jar Package
3. Pipelines
4. Pipeline Step
5. Inference Configuration

This page will go through each one of them and explain what they are used for.

### **CLI Interface**

Konduit-Serving comes with a CLI interface \(with a konduit alias\) that's responsible for taking care of most aspects of the application. The help command will describe most of what we are able to do with konduit-serving. Executing konduit --help command will show us the following output:  


```bash
$ konduit --help
---------------------------------------------------------------------------
Usage: konduit [COMMAND] [OPTIONS] [arg...]

Commands:
    build         Command line interface for performing Konduit Serving builds.
    config        A helper command for creating boiler plate json/yaml for
                  inference configuration
    inspect       Inspect the details of a particular konduit server.
    list          Lists the running konduit servers.
    logs          View the logs of a particular konduit server
    predict       Run inference on konduit servers using given inputs
    profile       Command to List, view, edit, create and delete konduit
                  serving run profiles.
    pythonpaths   A utility command to manage system installed and manually
                  registered python binaries.
    serve         Start a konduit server application
    stop          Stop a running konduit server
    version       Displays konduit-serving version.

Run 'konduit COMMAND --help' for more information on a command.
---------------------------------------------------------------------------
```

**Each command describes its shorthand description right in front of it. If you want to look at an individual command in detail, you can use the corresponding --help command with them. For example, the help menu for the logs command can be seen by executing, konduit logs --help:**  


| **$ konduit logs --help ---------------------------------------------------------------------------Usage: konduit logs  \[-f\] \[-l &lt;value&gt;\]  server-id  View the logs of a particular konduit server  View the logs of a particular konduit server given an id.  Example usages: -------------- - Outputs the log file contents of server with an id of 'inf\_server': $ konduit logs inf\_server  - Outputs and tail the log file contents of server with an id of 'inf\_server': $ konduit logs inf\_server -f  - Outputs and tail the log file contents of server with an id of 'inf\_server'   from the last 10 lines: $ konduit logs inf\_server -l 10 -f --------------  Options and Arguments:  -f,--follow          Follow the logs output.  -l,--lines &lt;value&gt;   Sets the number of lines to be printed. Default is '10'.                       Use -1 for outputting everything.   &lt;server-id&gt;          Konduit server id ---------------------------------------------------------------------------** |
| :--- |


**As can be seen, the --help command for an individual help command describes its functionality in detail along with some explicit examples and use cases. It also describes each individual optional/non-optional argument that can be used with it. This can come in very handy while learning about konduit-serving for the first time and is a useful starting place to play around with a specific command. You can do the same for the rest of the commands. Which are:**   


* **build**
* **config**
* **inspect**
* **list**
* **logs**
* **predict**
* **profile**
* **pythonpaths**
* **serve**
* **stop**
* **version**

### **Jar File Package**

**Each konduit-serving distribution whether it is for Windows, Linux or MacOS comes contained in a JAR file. So, you'll need a Java Virtual Machine present in the system where you're using Konduit-Serving as a Model Pipeline Server. The CLI itself is linked with the jar file and utilizes a java runtime internally to interact with the Konduit-Serving package. If you look at the konduit serving distribution, you'll see the following folder architecture in the root folder:**  


![](https://docs.google.com/drawings/u/2/d/sU9Kk3p6mTwO5DcFFatUAnA/image?w=397&h=320&rev=189&ac=1&parent=1pghi_Njn8fb-rcy9nOwCivozE5-CUEkbvq0vyDYCibc)

**The main CLI logic is places under bin/konduit file, which contains the following content:**  


| **\#!/usr/bin/env bash  SCRIPT\_DIR="$\(dirname "$0"\)"  . ${SCRIPT\_DIR}/../conf/konduit-serving-env.sh  java -jar -Dvertx.cli.usage.prefix=konduit ${SCRIPT\_DIR}/../konduit.jar "$@"** |
| :--- |


**As you can see, it uses the java command which is available through a Java runtime environment. The java command itself uses the konduit.jar file which is the main application package inside a Konduit-Serving distribution.**  


**This JAR file will be used as a Java application dependency while creating custom endpoints logic for a Konduit-Serving pipeline. We'll get to how we can do that later in this notebook.**

### **Introduction to Pipelines**

**Throughout this document the term "Model Serving Pipeline" has been used. This refers to how Machine Learning or Deep Learning models get served on an application server. Machine/Deep Learning models work on n-dimensional arrays \(also known as ND-Arrays\). They don't know how to convert a JPEG or PNG image into numbers directly. Instead, they expect pre-processed data in the form of a multidimensional array. Also, any other form of data, be it text, audio or video, gets converted into numbers ND-Arrays before getting fed into a machine learning model.**  


**The process during which the data is converted from one form to another is called pre-processing and is done just before it's fed as a model input. So, in a sense you can see this as being Lego blocks fitting into each other. One part takes input in a specific form and outputs it into another form, which in turn gets fed into the next part. This chaining of processes creates a series of steps which have specific jobs to perform before the next step and the end result is a machine learning Pipeline. The typical flow of the pipeline looks like the following:**  


![](https://docs.google.com/drawings/u/2/d/sSSQ69On15sDrkay14FWOCQ/image?w=624&h=112&rev=70&ac=1&parent=1pghi_Njn8fb-rcy9nOwCivozE5-CUEkbvq0vyDYCibc)

**A pipeline can also be in the form of a directed acyclic graph or DAG where data can flow into the graph and can give multiple outputs. In Konduit-Serving a Pipeline graph can also contain optional graph branches and can also concatenate outputs from multiple graph nodes. For the sake of the current goal \(BMI Model Serving\) we'll stick to a Sequential Pipeline which only has one input and one output.**

### **Pipeline Steps**

**Inside Konduit-Serving, a pipeline can be broken down into steps, where each step is responsible for performing a specific function. A pipeline step is the smallest component of a whole pipeline and can be used for a whole list of operations. To see the list of available pipeline steps you can use the config command in Konduit-Serving CLI.**  


<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p><b>$ konduit config --help<br />---------------------------------------------------------------------------<br />Usage: konduit config  [-m] [-o &lt;output-file&gt;] -p &lt;config&gt; [-pr &lt;value&gt;]  [-y]<br /><br />A helper command for creating boilerplate json/yaml for inference configuration<br /><br />This command is a utility to create boilerplate json/yaml configurations that can be conveniently modified to start konduit servers.<br /><br />Example usages:<br />--------------<br />                     -- FOR SEQUENCE PIPELINES--<br />- Prints &apos;logging -&gt; tensorflow -&gt; logging&apos; config in pretty format:<br />$ konduit config -p logging,tensorflow,logging<br /><br />- Prints &apos;logging -&gt; tensorflow -&gt; logging&apos; config with gRPC protocol<br />  in pretty format:<br />$ konduit config -p logging,tensorflow,logging -pr grpc<br /><br />- Prints &apos;dl4j -&gt; logging&apos; config in minified format:<br />$ konduit config -p dl4j,logging -m<br /><br />- Saves &apos;dl4j -&gt; logging&apos; config in a &apos;config.json&apos; file:<br />$ konduit config -p dl4j,logging -o config.json<br /><br />- Saves &apos;dl4j -&gt; logging&apos; config in a &apos;config.yaml&apos; file:<br />$ konduit config -p dl4j,logging -y -o config.json<br /><br /><br />                  -- FOR GRAPH PIPELINES --<br />- Generates a config that logs the input(1) then flow them through two<br />  tensorflow models(2,3) and merges the output(4):<br />$ konduit config -p<br />1=logging(input),2=tensorflow(1),3=tensorflow(1),4=merge(2,3)<br /><br />- Generates a config that logs the input(1) then channels(2) them through one<br />  of the two tensorflow models(3,4) and then selects the output(5) based<br />  on the value of the selection integer field &apos;select&apos;<br />$ konduit config -p<br />1=logging(input),[2_1,2_2]=switch(int,select,1),3=tensorflow(2_1),4=tensorflow(2_2),5=any(3,4)<br /><br />- Generates a config that logs the input(1) then channels(2) them through one<br />  of the two tensorflow models(3,4) and then selects the output(5) based<br />  on the value of the selection string field &apos;select&apos; in the selection map<br />  (x:0,y:1).<br />$ konduit config -p 1=logging(input),[2_1,2_2]=switch(string,select,x:0,y:1,1),3=tensorflow(2_1),4=tensorflow(2_2),5=any(3,4)<br />--------------<br /><br />Options and Arguments:<br /> -m,--minified               If set, the output json will be printed in a<br />                             single line, without indentations. (Ignored<br />                             for yaml)<br /><br /> -o,--output &lt;output-file&gt;   Optional: If set, the generated json/yaml will<br />                             be saved here. Otherwise, it&apos;s printed on the<br />                             console.</b>
        </p>
        <p><b><br /> -p,--pipeline &lt;config&gt;      A comma-separated list of sequence/graph<br />                             pipeline<br />                             steps to create boilerplate configuration<br />                             from. For<br />                             sequences, allowed values are: [crop_grid,<br />                             crop_fixed_grid, dl4j, keras,<br />                             Draw_bounding_box, draw_fixed_grid, draw_grid,<br />                             draw_segmentation,extract_bounding_box,<br />                             Camera_frame_capture, video_frame_capture,<br />                             Image_to_ndarray, logging,<br />                             ssd_to_bounding_box, samediff, show_image,<br />                             tensorflow, nd4jtensorflow, python, onnx].<br />                             For graphs, the list item should be in the<br />                             format<br />                             &apos;&lt;output&gt;=&lt;type&gt;(&lt;inputs&gt;)&apos; or<br />                             &apos;[outputs]=switch(&lt;inputs&gt;)&apos; for switches. The<br />                             pre-defined root input is named, &apos;input&apos;.<br />                             Examples<br />                             are ==&gt; Pipeline step:<br />                             &apos;a=tensorflow(input),b=dl4j(input)&apos; Merge<br />                             Step:<br />                             &apos;c=merge(a,b)&apos; Switch Step (int):<br />                             &apos;[d1,d2,d3]=switch(int,select,input)&apos; Switch<br />                             Step<br />                             (string):<br />                             &apos;[d1,d2,d3]=switch(string,select,x:1,y:2,z:3,input)&apos;<br />                             &apos;Any Step: &apos;e=any(d1,d2,d3)&apos; See the examples<br />                             above for more usage information.<br /> -pr,--protocol &lt;value&gt;      Protocol to use with the server. Allowed<br />                             values are<br />                             [http, grpc, mqtt]<br /> -y,--yaml                   Set if you want the output to be a yaml<br />                             configuration.<br />---------------------------------------------------------------------------</b>
        </p>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

**config command can take a pipeline pattern string and creates a pipeline configuration as well as server configuration. The server configuration combined with pipeline step configuration is collectively called Inference Configuration in Konduit-Serving. This is the final output from the config command.**  


**The following pipeline step types can be used to configure a pipeline configuration.**  


| **\#** | **Pipeline Step Type Name** | **Description** |
| :--- | :--- | :--- |
| **1** | **CROP\_GRID** | **A pipeline step that crops sub images out of a larger image, based on a grid.** |
| **2** | **CROP\_FIXED\_GRID** | **This step is similar to CROP\_GRID with the difference that the x/y location values are hardcoded into the configuration, instead of coming dynamically from the input Data instance.** |
| **3** | **DL4J** | **A model pipelinestep that serves a DL4J model.** |
| **4** | **KERAS** | **A model pipeline step that serves a Keras model.** |
| **5** | **DRAW\_BOUNDING\_BOX** | **A pipeline step that configures how to draw a bounding box onto an image. The bounding box data, that's to be drawn, is taken from the previous step's data instance.** |
| **6** | **DRAW\_GRID** | **Draw a grid on the specified image, based on the x/y coordinates of the corners, and the number of segments within the grid in both directions.** |
| **7** | **DRAW\_FIXED\_GRID** | **A pipeline step that draws a grid on an image. This is similar to DRAW\_GRID but the corner x/y location values are hardcoded into the configuration \(via points\), instead of coming dynamically from the input Data instance.** |
| **8** | **DRAW\_SEGMENTATION** | **A pipeline step that configures how to draw a segmentation mask, optionally on an image.** |
| **9** | **EXTRACT\_BOUNDING\_BOX** | **A pipeline step that extracts sub-images from an input image, based on the locations of input bounding boxes.**  |
| **10** | **CAMERA\_FRAME\_CAPTURE** | **A pipeline step that specifies an input that's taken from a camera feed.** |
| **11** | **VIDEO\_FRAME\_CAPTURE** | **A pipeline step that configures how to extract a single frame from a video each time inference is called. The video path is hardcoded, mainly used for testing/demo purposes.** |
| **12** | **IMAGE\_TO\_NDARRAY, LOGGING** | **A PipelineStep for converting images to n-dimensional arrays.**  |
| **13** | **SSD\_TO\_BOUNDING\_BOX** | **A pipeline step that configures extraction of bounding boxes from an SSD model output.** |
| **14** | **SAMEDIFF** | **A model pipeline step that serves a SameDiff model.** |
| **15** | **SHOW\_IMAGE** | **A pipeline step that configures how to show/render an image from a previous step in an application frame. Usually only used for testing and debugging locally, not when serving from HTTP/GRPC etc endpoints.** |
| **16** | **TENSORFLOW** | **A model pipeline step that serves a TensorFlow model using Tensorflow Java API. This is packaged into Konduit-Serving through JavaCPP.** |
| **17** | **ND4JTENSORFLOW** | **A pipeline step that configures a TensorFlow model that is to be executed based on an ND4J graph runner. This has performance benefits over native TensorFlow Java distribution.** |
| **18** | **PYTHON** | **This pipeline step can take an arbitrary python script and serve that through Konduit-Serving.** |
| **19** | **ONNX** | **A model pipeline step that serves a ONNX model.** |

### **Inference Configuration**

**Inference configuration contains all the details about how the pipeline server should be setup, which protocol it’s supposed to use for sending in/out data, whether we have a sequential or graph pipeline and finally how the pipeline is configured. The config command outputs inference configuration for us that can be directly used with a serve command after necessary changes are made. An Example of inference configuration through config command is as follows:**  


<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p><b>$ konduit config --pipeline logging --yaml</b>
        </p>
        <p><b>---------------------------------------------------------------------------<br />---<br />host: &quot;localhost&quot;<br />port: 0<br />use_ssl: false<br />protocol: &quot;HTTP&quot;<br />kafka_configuration:<br />  start_http_server_for_kafka: true<br />  http_kafka_host: &quot;localhost&quot;<br />  http_kafka_port: 0<br />  consumer_topic_name: &quot;inference-in&quot;<br />  consumer_key_deserializer_class: &quot;io.vertx.kafka.client.serialization.JsonObjectDeserializer&quot;<br />  consumer_value_deserializer_class: &quot;io.vertx.kafka.client.serialization.JsonObjectDeserializer&quot;<br />  consumer_group_id: &quot;konduit-serving-consumer-group&quot;<br />  consumer_auto_offset_reset: &quot;earliest&quot;<br />  consumer_auto_commit: &quot;true&quot;<br />  producer_topic_name: &quot;inference-out&quot;<br />  producer_key_serializer_class: &quot;io.vertx.kafka.client.serialization.JsonObjectSerializer&quot;<br />  producer_value_serializer_class: &quot;io.vertx.kafka.client.serialization.JsonObjectSerializer&quot;<br />  producer_acks: &quot;1&quot;<br />mqtt_configuration: {}<br />custom_endpoints: []<br />pipeline:<br />  steps:<br />  - &apos;@type&apos;: &quot;LOGGING&quot;<br />    logLevel: &quot;INFO&quot;<br />    log: &quot;KEYS_AND_VALUES&quot;</b>
        </p>
        <p><b>---------------------------------------------------------------------------</b>
        </p>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

**The above command creates a yaml configuration for a logging pipeline step. Logging pipeline step just takes in an input from the previous pipeline step and outputs \(and logs on console\) the same information. This is useful for debugging issues in the running servers.**  


**This configuration can be saved inside a file by executing the following:**  


<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p><b>$ konduit config --pipeline logging --yaml --output server-conf.yaml</b>
        </p>
        <p><b>---------------------------------------------------------------------------<br />Config file created successfully at /root/server-conf.yaml</b>
        </p>
        <p><b>---------------------------------------------------------------------------</b>
        </p>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

**Afterwards, this configuration can be used with the serve command to start a Konduit-Serving server. Example is following:**   


| **$ konduit serve --config server-conf.yaml -id server --background --runWithoutManifest --------------------------------------------------------------------------- Starting konduit server... Using classpath: /root/konduit/bin/../konduit.jar INFO: Running command /root/miniconda/jre/bin/java -Dkonduit.logs.file.path=/root/.konduit-serving/command\_logs/server.log -Dlogback.configurationFile=/tmp/logback-run\_command\_4b1acb956d0d436c.xml -jar /root/konduit/bin/../konduit.jar run --instances 1 -s inference -c server-conf.yaml -Dserving.id=server For server status, execute: 'konduit list' For logs, execute: 'konduit logs server' ---------------------------------------------------------------------------** |
| :--- |


