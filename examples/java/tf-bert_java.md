---
description: >-
  This program illustrates a simple client-server interaction to perform
  inference on a TensorFlow model using the Java SDK for Konduit Serving.
---

# BERT

```java
import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.config.ParallelInferenceConfig;
import ai.konduit.serving.config.ServingConfig;
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.model.ModelConfigType;
import ai.konduit.serving.model.TensorDataTypesConfig;
import ai.konduit.serving.model.TensorFlowConfig;
import ai.konduit.serving.pipeline.step.ModelStep;
```
## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

Start by downloading the model weights to the `data` folder.The downloaded zip file can be unzipped using Util class.

```java
String bertFileName = "bert_mrpc_frozen.pb";
bertmodelfilePath = bertmodelfilePath + "/" +bertFileName;
File bertFile = new File(bertmodelfilePath);
if (!bertFile.exists()) {
  File bertFileDir = Util.fileDownload("https://deeplearning4jblob.blob.core.windows.net/testresources/bert_mrpc_frozen_v1.zip");
  Util.unzipFile(bertFileDir.toString(),bertFileName);
}
```
A reference Java project is provided in this repository with a Maven pom.xml dependencies file. If using the IntelliJ IDEA IDE, open the java folder as a Maven project and run the main function of InferenceModelStep the class.

## Configure the step

Define the TensorFlow configuration as a `TensorFlowConfig` object.

* `tensorDataTypesConfig`: The TensorFlowConfig object requires a dictionary `input_data_types`. Its keys should represent column names, and the values should represent data types as strings, e.g. `"INT32"`. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `modelConfigType`: This argument requires a `ModelConfigType` object. Specify `modelType` as `TENSORFLOW`, and `modelLoadingPath` to point to the location of TensorFlow weights saved in the PB file format.

```java
HashMap<String, TensorDataType> input_data_types = new LinkedHashMap<>();
input_data_types.put("IteratorGetNext:0", TensorDataType.INT32);
input_data_types.put("IteratorGetNext:1", TensorDataType.INT32);
input_data_types.put("IteratorGetNext:4", TensorDataType.INT32);

//Model config and set model type as BERT
ModelConfig bertModelConfig = TensorFlowConfig.builder()
    .tensorDataTypesConfig(TensorDataTypesConfig.builder().
            inputDataTypes(input_data_types).build())
    .modelConfigType(ModelConfigType.builder().
            modelLoadingPath(bertmodelfilePath.toString()).
            modelType(ModelConfig.ModelType.TENSORFLOW).build())
    .build();
```

Now that we have a `TensorFlowConfig` defined, we can define a `ModelStep`. The following parameters are specified:

* `modelConfig`: pass the TensorFlowConfig object here 
* `parallelInferenceConfig`: specify the number of workers to run in parallel. Here, we specify `workers=1`.
* `inputNames`:  names for the input data  
* `outputNames`: names for the output data

```java
//Set the input and output names for model step
List<String> input_names = new ArrayList<String>(input_data_types.keySet());
ArrayList<String> output_names = new ArrayList<>();
output_names.add("loss/Softmax");

//Set the configuration of model to step
ModelStep bertModelStep = ModelStep.builder()
    .modelConfig(bertModelConfig)
    .inputNames(input_names)
    .outputNames(output_names)
    .parallelInferenceConfig(ParallelInferenceConfig.builder().workers(1).build())
    .build();
```

## Configure the server

Specify the following:

* `httpPort`: select a random port.
* `inputDataFormat`, `outputDataFormat`: Specify input and output data formats as strings. 

{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.
{% endhint %}

```java
int port = Util.randInt(1000, 65535);
ServingConfig servingConfig = ServingConfig.builder().httpPort(port).
    inputDataFormat(Input.DataFormat.NUMPY).
    build();
```

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Python list. In this case, there is a single step: `bertModelStep`.

```java
InferenceConfiguration inferenceConfiguration = InferenceConfiguration.builder()
    .servingConfig(servingConfig)
    .step(bertModelStep)
    .build();
```

Set the KonduitServingMainArgs with the Server Configuration arguments.

```java
KonduitServingMainArgs args1 = KonduitServingMainArgs.builder()
    .configStoreType("file").ha(false)
    .multiThreaded(false).configPort(port)
    .verticleClassName(InferenceVerticle.class.getName())
    .configPath(configFile.getAbsolutePath())
    .build();
```
Start server by calling KonduitServingMain with the configurations mentioned in the KonduitServingMainArgs using Callback Function(as per the code mentioned in the **Inference** Section below)

## Inference

Load some sample data from NumPy files. Note that these are NumPy arrays, each with shape \(4, 128\):

```java
File input0 = new ClassPathResource("data/bert/input-0.npy").getFile();
File input1 = new ClassPathResource("data/bert/input-1.npy").getFile();
File input4 = new ClassPathResource("data/bert/input-4.npy").getFile();
```
To configure the client, set the required URL to connect server and specify any port number that is not reserved (as used in server configuration).  

Note that you should create the Client object after the Server has started, so that Client can inherit the Server's attributes. A Callback Function onSuccess is implemented in order to start the Client only after the successful run of the KonduitServingMain Server.

```java
 KonduitServingMain.builder()
      .onSuccess(()->{
          try {
              //Create new file to write binary input data.
              //client config.
              String response = Unirest.post("http://localhost:"+port+"/raw/numpy")
                      .field("IteratorGetNext:0", input0)
                      .field("IteratorGetNext:1", input1)
                      .field("IteratorGetNext:4", input4)
                      .asString().getBody();
              System.out.print(response);
          } catch (UnirestException e) {
              e.printStackTrace();
          }
      })
      .build()
      .runMain(args1.toArgs());
```

## Confirm the output

After executing the above, in order to confirm the successful start of the Server, check for the below output text:

```text
Jan 15, 2020 6:02:49 PM ai.konduit.serving.configprovider.KonduitServingMain
INFO: Deployed verticle ai.konduit.serving.verticles.inference.InferenceVerticle
```

The Output of the program is as follows:

```java
System.out.print(response);
```

```text
"loss/Softmax" : {
  "batchId" : "41600218-5fb7-401f-af7d-e7fe13313f5d",
  "ndArray" : {
    "dataType" : "FLOAT",
    "shape" : [ 4, 2 ],
    "data" : [ 0.9894917, 0.010508226, 0.8021635, 0.19783656, 0.9874369, 0.012563077, 0.99294597, 0.0070540793 ]
  }
```

The configuration is stored as a JSON. Note that the configuration can be converted to a dictionary using the `toJson()` method:

```java
System.out.println(inferenceConfiguration.toJson());
```

```text
{
  "memMapConfig" : null,
  "servingConfig" : {
    "httpPort" : 11805,
    "inputDataFormat" : "NUMPY",
    "listenHost" : "localhost",
    "logTimings" : false,
    "metricTypes" : [ "CLASS_LOADER", "JVM_MEMORY", "JVM_GC", "PROCESSOR", "JVM_THREAD", "LOGGING_METRICS", "NATIVE" ],
    "outputDataFormat" : "JSON",
    "predictionType" : "RAW",
    "uploadsDirectory" : "file-uploads/"
  },
  "steps" : [ {
    "@type" : "ModelStep",
    "inputColumnNames" : { },
    "inputNames" : [ "IteratorGetNext:0", "IteratorGetNext:1", "IteratorGetNext:4" ],
    "inputSchemas" : { },
    "modelConfig" : {
      "@type" : "TensorFlowConfig",
      "configProtoPath" : null,
      "modelConfigType" : {
        "modelLoadingPath" : "C:\\konduit-serving-examples\\java\\target\\classes\\data\\bert/bert_mrpc_frozen.pb",
        "modelType" : "TENSORFLOW"
      },
      "savedModelConfig" : null,
      "tensorDataTypesConfig" : {
        "inputDataTypes" : {
          "IteratorGetNext:0" : "INT32",
          "IteratorGetNext:1" : "INT32",
          "IteratorGetNext:4" : "INT32"
        },
        "outputDataTypes" : { }
      }
    },
    "normalizationConfig" : null,
    "outputColumnNames" : { },
    "outputNames" : [ "loss/Softmax" ],
    "outputSchemas" : { },
    "parallelInferenceConfig" : {
      "batchLimit" : 32,
      "inferenceMode" : "BATCHED",
      "maxTrainEpochs" : 1,
      "queueLimit" : 64,
      "vertxConfigJson" : null,
      "workers" : 1
    }
  } ]
}
```