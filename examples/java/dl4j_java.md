---
Description: >-
  This page documents two ways to create Konduit Serving configurations with the Java SDK:  

---

# Deeplearning4j \(DL4J\)


* Using Java to create a configuration


```java
import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.config.ServingConfig;
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.configprovider.KonduitServingMainArgs;
import ai.konduit.serving.model.ModelConfig;
import ai.konduit.serving.model.ModelConfigType;
import ai.konduit.serving.model.TensorDataTypesConfig;
import ai.konduit.serving.pipeline.step.ModelStep;
import ai.konduit.serving.verticles.inference.InferenceVerticle;
import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;
import org.apache.commons.io.FileUtils;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.io.ClassPathResource;
import org.nd4j.serde.binary.BinarySerde;
import org.nd4j.tensorflow.conversion.TensorDataType;
```


## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

Set the dl4j model path to dl4jmodelfilePath.

```java
String dl4jmodelfilePath = new ClassPathResource("data/multilayernetwork/SimpleCNN.zip").
    getFile().getAbsolutePath();
```

{% hint style="info" %}
A reference Java project is provided in the Example repository \( https://github.com/KonduitAI/konduit-serving-examples \) with a Maven pom.xml dependencies file. If using the IntelliJ IDEA IDE, open the java folder as a Maven project and run the main function of the InferenceModelStepDL4J class.
{% endhint %}

## Configure the step

Define the DL4J configuration as a `ModelConfig` object.

* `tensorDataTypesConfig`: The ModelConfig object requires a HashMap `input_data_types`. Its keys should represent column names, and the values should represent data types as strings, e.g. `"INT32"`, `"FLOAT"`,etc,. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `modelConfigType`: This argument requires a `ModelConfigType` object. In the Java program above, we recognised that SimpleCNN is configured as a MultiLayerNetwork, in contrast with the ComputationGraph class, which is used for more complex networks. Specify `modelType` as `MULTI_LAYER_NETWORK`, and `modelLoadingPath` to point to the location of DL4J weights saved in the ZIP file format.

```java
Map<String, TensorDataType> input_data_types = new HashMap<>();
input_data_types.put("image_array", TensorDataType.FLOAT);

List<String> input_names = new ArrayList<String>(input_data_types.keySet());
List<String> output_names = new ArrayList<>();
output_names.add("output");

ModelConfig dl4jModelConfig = ModelConfig.builder()
    .tensorDataTypesConfig(TensorDataTypesConfig.builder().
            inputDataTypes(input_data_types).build())
    .modelConfigType(ModelConfigType.builder().
            modelLoadingPath(dl4jmodelfilePath.toString()).
            modelType(ModelConfig.ModelType.MULTI_LAYER_NETWORK).build())
    .build();
```

For the `ModelStep` object, the following parameters are specified:

* `modelConfig`: pass the ModelConfig object here 
* `input_names`:  names for the input data  
* `output_names`: names for the output data

```java
ModelStep dl4jModelStep = ModelStep.builder()
    .modelConfig(dl4jModelConfig)
    .inputNames(input_names)
    .outputNames(output_names)
    .build();
```

## Configure the server

Specify the following:

* `httpPort`: specify any port number that is not reserved.

```java
int port = Util.randInt(1000, 65535);

ServingConfig servingConfig = ServingConfig.builder()
    .httpPort(port)
    .build();
```

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Java list. In this case, there is a single step: `dl4jModelStep`.

```java
InferenceConfiguration inferenceConfiguration = InferenceConfiguration.builder()
    .servingConfig(servingConfig)
    .step(dl4jModelStep)
    .build();
```

{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.
{% endhint %}

The `inferenceConfiguration` is stored as a JSON File. Set the KonduitServingMainArgs with the saved **config.json** file path as `configPath` and other necessary server configuration arguments.

```java
File configFile = new File("config.json");
FileUtils.write(configFile, inferenceConfiguration.toJson(), Charset.defaultCharset());

KonduitServingMainArgs args1 = KonduitServingMainArgs.builder()
    .configStoreType("file").ha(false)
    .multiThreaded(false).configPort(port)
    .verticleClassName(InferenceVerticle.class.getName())
    .configPath(configFile.getAbsolutePath())
    .build();
```
Start server by calling KonduitServingMain with the configurations mentioned in the KonduitServingMainArgs using Callback Function(as per the code mentioned in the **Inference** Section below)


## Inference

We generate a \(3, 224, 224\) array of random numbers between 0 and 255 as input to the model for prediction.

Before requesting for a prediction, we normalize the image to be between 0 and 1:

```java
INDArray rand_image = Util.randInt(new int[]{1, 3, 244, 244}, 255);
```

```java
File file = new File("src/main/resources/data/test-dl4j.zip");

if(!file.exists()) file.createNewFile();

BinarySerde.writeArrayToDisk(rand_image, file);

```

To configure the client, set the required URL to connect server and specify any port number that is not reserved (as used in server configuration).  

A Callback Function onSuccess is implemented in order to post the Client request and get the HttpResponse, only after the successful run of the KonduitServingMain Server.

{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.
{% endhint %}

```java
KonduitServingMain.builder()
    .onSuccess(() -> {
        try {
            String response = Unirest.post(String.format("http://localhost:%s/raw/nd4j", port))
                    .field("image_array", file).asString().getBody();
            System.out.print(response);
            System.exit(0);
        } catch (UnirestException e) {
            e.printStackTrace();
            System.exit(0);
        }
    })
    .build()
    .runMain(args1.toArgs());
```

# Confirm the Output

After executing the above, in order to confirm the successful start of the Server, check for the below output text:

```text
Jan 08, 2020 3:03:50 PM ai.konduit.serving.configprovider.KonduitServingMain
INFO: Deployed verticle ai.konduit.serving.verticles.inference.InferenceVerticle
```

The Output of the program is as follows:

```java
System.out.print(response);
```
```text
{
  "output" : {
    "batchId" : "d5090c30-526d-4e1f-93e2-a918435ac1da",
    "ndArray" : {
      "dataType" : "FLOAT",
      "shape" : [ 1, 5 ],
      "data" : [ 0.028113496, 0.3778126, 0.023068674, 3.759411E-5, 0.5709677 ]
    }
  }
}
```

The complete inference configuration in JSON format is as follows:

```java
System.out.println(inferenceConfiguration.toJson());
```
```text
{
  "memMapConfig" : null,
  "servingConfig" : {
    "httpPort" : 24229,
    "listenHost" : "localhost",
    "logTimings" : false,
    "metricTypes" : [ "CLASS_LOADER", "JVM_MEMORY", "JVM_GC", "PROCESSOR", "JVM_THREAD", "LOGGING_METRICS", "NATIVE" ],
    "outputDataFormat" : "JSON",
    "uploadsDirectory" : "file-uploads/"
  },
  "steps" : [ {
    "@type" : "ModelStep",
    "inputColumnNames" : { },
    "inputNames" : [ "image_array" ],
    "inputSchemas" : { },
    "modelConfig" : {
      "@type" : "ModelConfig",
      "modelConfigType" : {
        "modelLoadingPath" : "C:\\konduit-serving-examples\\java\\target\\classes\\data\\multilayernetwork\\SimpleCNN.zip",
        "modelType" : "MULTI_LAYER_NETWORK"
      },
      "tensorDataTypesConfig" : {
        "inputDataTypes" : {
          "image_array" : "FLOAT"
        },
        "outputDataTypes" : { }
      }
    },
    "normalizationConfig" : null,
    "outputColumnNames" : { },
    "outputNames" : [ "output" ],
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
