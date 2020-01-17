---
description: >-
  This page documents two ways to create Konduit Serving configurations with the Java SDK:  

---

# Deeplearning4j \(DL4J\)


* Using Java to create a configuration


```java
package ai.konduit.serving.examples.inference;

import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.config.Input;
import ai.konduit.serving.config.ServingConfig;
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.model.ModelConfig;
import ai.konduit.serving.model.ModelConfigType;
import ai.konduit.serving.model.TensorDataTypesConfig;
import ai.konduit.serving.pipeline.step.ModelStep;
import org.apache.commons.io.FileUtils;
import org.nd4j.linalg.io.ClassPathResource;
import org.nd4j.tensorflow.conversion.TensorDataType;

import javax.annotation.concurrent.NotThreadSafe;
import java.io.File;
import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

package ai.konduit.serving.examples.inference;

import com.mashape.unirest.http.Unirest;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.serde.binary.BinarySerde;

import java.io.File;
```



## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.


## Configure the step

Define the DL4J configuration as a `ModelConfig` object.

* `tensorDataTypesConfig`: The ModelConfig object requires a dictionary `inputDataTypes`. Its keys should represent column names, and the values should represent data types as strings, e.g. `"INT32"`. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `modelConfigType`: This argument requires a `ModelConfigType` object. In the Java program above, we recognised that SimpleCNN is configured as a MultiLayerNetwork, in contrast with the ComputationGraph class, which is used for more complex networks. Specify `modelType` as `MULTI_LAYER_NETWORK`, and `modelLoadingPath` to point to the location of DL4J weights saved in the ZIP file format.

```java
input_data_types = {"image_array": "FLOAT"}
input_names = list(input_data_types.keys())
output_names = ["output"]
int port = Util.randInt(1000, 65535);

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
* `InferenceConfiguration`: specify the number of workers to run in parallel. Here, we specify `workers=1`.
* `input_names`:  names for the input data  
* `output_names`: names for the output data

```java
ModelStep dl4jModelStep = ModelStep.builder()
                .modelConfig(dl4jModelConfig)
                .inputNames(input_names)
                .outputNames(output_names)
                .build();
```

{% hint style="info" %}
To find the names of input and output nodes in DL4J,

* for `input_names`, print the first element of `net.getLayerNames()`.
* for `output_names`, check the last layer when printing `net.summary()`. 

## Configure the server

Specify the following:

* `httpPort`: select a random port.
* `inputDataFormat`, `outputDataFormat`: Specify input and output data formats as strings. 

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Java list. In this case, there is a single step: `dl4jModelStep`.

```java
ServingConfig servingConfig = ServingConfig.builder().httpPort(3000).
                inputDataFormat(Input.DataFormat.ND4J).
                // outputDataFormat(Output.DataFormat.ND4J).
                        build();
```


{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.
{% endhint %}


## Start the server

Start server by calling KonduitServingMain.main with the configurations mentioned in the following code block. 

```java
KonduitServingMain.main("--configPath", configFile.getAbsolutePath());" File configFile = new File("config.json");
FileUtils.write(configFile, inferenceConfiguration.toJson(), Charset.defaultCharset());
  KonduitServingMain.main("--configPath", configFile.getAbsolutePath());
```

```text
18:50:42.336 ai.konduit.serving.configprovider.KonduitServingMain

INFO: Deployed verticle {}

18:50:42.336 [vert.x-worker-thread-1] DEBUG a.k.s.v.inference.InferenceVerticle - Server started on port 3000

```

## Inference

We generate a \(3, 224, 224\) array of random numbers between 0 and 255 as input to the model for prediction.

NDARRAY inputs to ModelSteps must be specified with a preceding `batchSize` dimension. For batches with a single observation, this can be done by using `np.expand_dims()` to add an additional dimension to your array. 

Before requesting for a prediction, we normalize the image to be between 0 and 1:

```java
INDArray rand_image = Util.randInt(new int[]{1, 3, 244, 244}, 255)
```

```java
File file = new File("src/main/resources/data/test-dl4j.zip");
        BinarySerde.writeArrayToDisk(rand_image, file);
        System.out.println(rand_image);

server.stop()
```

```text
[[4.1741084e-02 3.2335979e-01 2.5368158e-02 3.9881383e-05 6.0949111e-01]]
```

```text
ModelConfig(tensorDataTypesConfig=TensorDataTypesConfig(inputDataTypes={image_array=FLOAT}, outputDataTypes={}), modelConfigType=ModelConfigType(modelType=MULTI_LAYER_NETWORK, modelLoadingPath=C:\Projects\Konduit\Konduit-Nidrvie-Examples\konduit-serving-examples\java\target\classes\data\multilayernetwork\SimpleCNN.zip))
{
  "memMapConfig" : null,
  "servingConfig" : {
    "httpPort" : 3000,
    "inputDataFormat" : "ND4J",
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
    "inputNames" : [ "image_array" ],
    "inputSchemas" : { },
    "modelConfig" : {
      "@type" : "ModelConfig",
      "modelConfigType" : {
        "modelLoadingPath" : "konduit-serving-examples\\java\\target\\classes\\data\\multilayernetwork\\SimpleCNN.zip",
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

# Output
The output of this code file will look similar to this: 
```java
{
  "output" : {
    "batchId" : "552d68d0-c83d-446d-9366-cf9617ad390b",
    "ndArray" : {
      "dataType" : "FLOAT",
      "shape" : [ 1, 5 ],
      "data" : [ 0.027882336, 0.374272, 0.022953229, 3.813142E-5, 0.5748543   ]
    }
  }
}
```