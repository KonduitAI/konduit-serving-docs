
Description: This notebook illustrates a simple client-server interaction to perform
  inference on a TensorFlow model using the Java SDK for Konduit Serving.

* To run this java example, load the pom.xml for all dependencies to run the examples in konduit-serving-examples.


# MNIST

This tutorial is split into two parts:

1. Configuration 
2. Running the server

{% hint style="info" %}

This tutorial is tested on TensorFlow 1.14, 1.15 and 2.00.

{% endhint %}


```java
package ai.konduit.serving.examples.inference;

import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.config.Input;
import ai.konduit.serving.config.Output;
import ai.konduit.serving.config.ParallelInferenceConfig;
import ai.konduit.serving.config.ServingConfig;
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.model.ModelConfig;
import ai.konduit.serving.model.ModelConfigType;
import ai.konduit.serving.model.TensorDataTypesConfig;
import ai.konduit.serving.model.TensorFlowConfig;
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
```

## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.


## Configure the step   

Define the TensorFlow configuration as a `TensorFlowConfig.builder` object.

* `tensorDataTypesConfig`: The `TensorFlowConfig.builder()` object requires a dictionary `inputDataTypes`. Its keys should represent column names, and the values should represent data types as strings, e.g. `"INT32"`. See [here](https://github.com/KonduitAI/konduit-serving/blob/master/konduit-serving-api/src/main/java/ai/konduit/serving/model/TensorDataType.java) for a list of supported data types. 
* `modelConfigType`: This argument requires a `ModelConfigType.builder()` object. Specify `modelType` as `TENSORFLOW`, and `modelLoadingPath` to point to the location of TensorFlow weights saved in the PB file format.

```java
        ModelConfig mnistModelConfig = TensorFlowConfig.builder()
                .tensorDataTypesConfig(TensorDataTypesConfig.builder().
                        inputDataTypes(input_data_types).build())

                .modelConfigType(ModelConfigType.builder().
                        modelLoadingPath(mnistmodelfilePath.toString()).
                        modelType(ModelConfig.ModelType.TENSORFLOW).build())
                .build();
```

```text
{'@type': 'TensorFlowConfig',
 'tensorDataTypesConfig': {'@type': 'TensorDataTypesConfig',
  'inputDataTypes': {'input_layer': 'FLOAT'}},
 'modelConfigType': {'@type': 'ModelConfigType',
  'modelType': 'TENSORFLOW',
  'modelLoadingPath': '\konduit-serving-examples\\data\\mnist\\mnist_2.0.0.pb'}}
```

Now that we have a `TensorFlowConfig` defined, we can define a `ModelStep`. The following parameters are specified:

* `modelConfig`: pass the mnistModelConfig object here 
* `parallelInferenceConfig`: specify the number of workers to run in parallel. Here, we specify `ParallelInferenceConfig.builder().workers(1)`.
* `inputNames`:  names for the input data  
* `outputNames`: names for the output data

```java
        ModelStep bertModelStep = ModelStep.builder()
                .modelConfig(mnistModelConfig)
                .inputNames(input_names)
                .outputNames(output_names)
                .parallelInferenceConfig(ParallelInferenceConfig.builder().workers(1).build())
                .build();
```

{% hint style="info" %}

Konduit Serving requires input and output names to be specified. In TensorFlow, you can find the names of your input and output nodes by printing `model.inputs[0].op.name` and `model.outputs[0].op.name` respectively. For more details, please refer to this [StackOverflow answer](https://stackoverflow.com/a/49154874/12260518).

{% endhint %}


```java
# make note of hwo to obtain inputNames and outputNames
inputDataFormat = {'input_layer': 'FLOAT'}
inputNames = list(input_data_types.keys())
outputNames = ["output_layer/Softmax"]
```

## Configure the server

Specify the following:

* `httpPort`: Assign a random port to “httpPort.
* `inputDataFormat`, `outputDataFormat`: Specify input and output data formats as strings. 

```java
ServingConfig servingConfig = ServingConfig.builder().httpPort(port).
                inputDataFormat(Input.DataFormat.NUMPY).
                outputDataFormat(Output.DataFormat.NUMPY).
                build();
```

By default, `Server()` looks for the Konduit Serving JAR `konduit.jar` in the directory the script is run in. To change this default, use the `jar_path` argument.

{% hint style="info" %}

Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.

{% endhint %}

## Start the server
Start server by calling KonduitServingMain with the configurations mentioned in the KonduitServingMainArgs using Callback Function(as per the code mentioned in the **Inference** Section below)
```java
        KonduitServingMainArgs args1 = KonduitServingMainArgs.builder()
                .configStoreType("file").ha(false)
                .multiThreaded(false).configPort(port)
                .verticleClassName(InferenceVerticle.class.getName())
                .configPath(configFile.getAbsolutePath())
                .build();

        ImageTransformProcess imageTransformProcess = new ImageTransformProcess.Builder()
                .scaleImageTransform(20.0f)
                .resizeImageTransform(28, 28)
                .build();

        ImageLoadingStep imageLoadingStep = ImageLoadingStep.builder()
                .imageProcessingInitialLayout("NCHW")
                .imageProcessingRequiredLayout("NHWC")
                .inputName("default")
                .dimensionsConfig("default", new Long[]{240L, 320L, 3L}) // Height, width, channels
                .imageTransformProcess("default", imageTransformProcess)
                .build();

        ArrayList<INDArray> imageArr = new ArrayList<>();
        ArrayList<String> inputString = new ArrayList<>();
        inputString.add("data/facedetector/1.jpg");
```
## Configure the client

To configure the client, create a Client object with the following arguments:
* `httpPort`: Assign a random port to “httpPort.
* `inputDataFormat`: data format passed to the server for inference
* `outputDataFormat`: data format returned by the server endpoint 


```java
ServingConfig servingConfig = ServingConfig.builder().httpPort(port).
                inputDataFormat(Input.DataFormat.NUMPY).
                outputDataFormat(Output.DataFormat.NUMPY).
                build();
```

## Inference 

{% hint style="warning" %}

NDARRAY inputs to ModelSteps must be specified with a preceding `batchSize` dimension. For batches with a single observation, this can be done by using `np.expand_dims()` to add an additional dimension to your array. 

{% endhint %}

We obtain test images from the test set defined by `keras.datasets`.

```java
for (String imagePathStr : inputString) {
            String tmpInput = new ClassPathResource(imagePathStr).getFile().getAbsolutePath();
            Writable[][] tmpOutput = imageLoadingStep.createRunner().transform(tmpInput);
            INDArray tmpImage = ((NDArrayWritable) tmpOutput[0][0]).get();
            imageArr.add(tmpImage);
        }

        KonduitServingMain.builder()
                .onSuccess(() -> {
                    try {
                        for (INDArray indArray : imageArr) {

                            //Create new file to write binary input data.
                            File file = new File("src/main/resources/data/test-input.zip");
                            BinarySerde.writeArrayToDisk(indArray, file);

                            String result = Unirest.post(String.format("http://localhost:%s/raw/nd4j", port))
                                    .field("input_layer", file)
                                    .asString().getBody();

                            System.out.println(result);
                            System.exit(0);
                        }
                    } catch (UnirestException | IOException e) {
                        e.printStackTrace();
                        System.exit(0);
                    }
                })
                .build()
                .runMain(args1.toArgs());
```

![png](output_28_0)

```text
{0: 0.0, 1: 0.0, 2: 0.001, 3: 0.001, 4: 0.0, 5: 0.0, 6: 0.0, 7: 0.998, 8: 0.0, 9: 0.0}
```

![png](output_28_2)

```text
{0: 0.0, 1: 0.0, 2: 0.998, 3: 0.002, 4: 0.0, 5: 0.0, 6: 0.0, 7: 0.0, 8: 0.0, 9: 0.0}
```

![png](output_28_4)

```text
{0: 0.0, 1: 0.986, 2: 0.005, 3: 0.0, 4: 0.0, 5: 0.0, 6: 0.0, 7: 0.005, 8: 0.003, 9: 0.0}
```


## Confirm the output
The output from the log file looks like this

```text
{
  "memMapConfig" : null,
  "servingConfig" : {
    "httpPort" : 15531,
    "listenHost" : "localhost",
    "logTimings" : false,
    "metricTypes" : [ "CLASS_LOADER", "JVM_MEMORY", "JVM_GC", "PROCESSOR", "JVM_THREAD", "LOGGING_METRICS", "NATIVE" ],
    "outputDataFormat" : "JSON",
    "uploadsDirectory" : "file-uploads/"
  },
  "steps" : [ {
    "@type" : "ModelStep",
    "inputColumnNames" : { },
    "inputNames" : [ "input_layer" ],
    "inputSchemas" : { },
    "modelConfig" : {
      "@type" : "TensorFlowConfig",
      "configProtoPath" : null,
      "modelConfigType" : {
        "modelLoadingPath" : "C:\\Projects\\Konduit\\Konduit-Nidrvie-Examples\\konduit-serving-examples\\java\\target\\classes\\data\\mnist\\mnist_2.0.0.pb",
        "modelType" : "TENSORFLOW"
      },
      "savedModelConfig" : null,
      "tensorDataTypesConfig" : {
        "inputDataTypes" : {
          "input_layer" : "FLOAT"
        },
        "outputDataTypes" : { }
      }
    },
    "normalizationConfig" : null,
    "outputColumnNames" : { },
    "outputNames" : [ "output_layer/Softmax" ],
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



