---
description: >-
  This page illustrates a simple client-server interaction to perform inference
  on a Keras LSTM model using the Java SDK for Konduit Serving.
---

# Keras

```java
import ai.konduit.serving.model.ModelConfig;
import ai.konduit.serving.model.ModelConfigType;
import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.pipeline.step.ModelStep;
import ai.konduit.serving.config.ServingConfig;
```

## Saving models in Keras HDF5 \(.h5\) format

Models can be saved with the `.save()` method. Refer to the [TensorFlow documentation for Keras](https://www.tensorflow.org/guide/keras/save_and_serialize) for details.

{% hint style="info" %}
Keras model loading functionality in Konduit Serving converts Keras models to Deeplearning4J models. As a result, Keras models containing operations not supported in Deeplearning4J cannot be served in Konduit Serving. See [issue 8348](https://github.com/eclipse/deeplearning4j/issues/8348).
{% endhint %}

## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

## Configure the step

Define the Keras configuration as a `ModelConfig` object.

* `modelConfigType`: This argument requires a `ModelConfigType` object. In the following Java program, we recognised that SimpleCNN is configured as a MultiLayerNetwork, in contrast with the ComputationGraph class, which is used for more complex networks. Specify `modelType` as `ModelConfig.ModelType.KERAS`, and `modelLoadingPath` to point to the location of Keras weights saved in the HDF5 file format. 


For the `ModelStep` object, the following parameters are specified:

* `modelConfig`: pass the ModelConfig object here 
* `InferenceConfiguration`: specify the number of workers to run in parallel. Here, we specify `workers = 1`.
* `inputName`, `outputName`: names for the input and output nodes, as lists 

```java
String kerasmodelfilePath = new ClassPathResource("data/keras/embedding_lstm_tensorflow_2.h5").getFile().getAbsolutePath();

ModelConfig kerasModelConfig = ModelConfig.builder()
                .modelConfigType(ModelConfigType.builder().modelLoadingPath(kerasmodelfilePath.toString()).
                        modelType(ModelConfig.ModelType.KERAS).build())
                .build();

ModelStep kerasmodelStep = ModelStep.builder()
                .modelConfig(kerasModelConfig)
                .inputName("input")
                .outputName("lstm_1")
                .build(); 
```

{% hint style="info" %}
Input and output names can be obtained by visualizing the graph in [Netron](https://github.com/lutzroeder/netron).
{% endhint %} 


## Configure the server

In the `ServingConfig`, specify any port number that is not reserved.


```java
ServingConfig servingConfig = ServingConfig.builder().httpPort(3000).
                inputDataFormat(Input.DataFormat.ND4J).
                outputDataFormat(Output.DataFormat.ND4J).
                 build();
```

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Python list. In this case, there is a single step: `kerasmodelStep`. 

```java
 //Inference Configuration
        InferenceConfiguration inferenceConfiguration = InferenceConfiguration.builder()
                .servingConfig(servingConfig)
                .step(kerasmodelStep)
                .build();
```

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

To configure the client, set the required URL to connect server and specify any port number that is not reserved (as used in server configuration).  

Note that you should create the Client object after the Server has started, so that Client can inherit the Server's attributes. 

NDARRAY inputs to set ModelStep must be specified with a shape size. 


```java
INDArray arr = Nd4j.create(new float[]{1.0f, 1.0f, 1.0f, 1.0f, 1.0f, 1.0f, 1.0f, 1.0f, 1.0f, 1.0f}, 1, 10);

        //Create new file to write binary input data.
        File file = new File("src/main/resources/data/test-input.zip");
        System.out.println(file.getAbsolutePath());

        BinarySerde.writeArrayToDisk(arr, file);

        String response = Unirest.post("http://localhost:3000/raw/nd4j")
                .field("input", file)
                .asString().getBody();

```

## Confirm the output

        System.out.print(response);

```text
{
"lstm_1" : {
"batchId" : "61c32b20-20c1-42d9-909e-40519304f2ac",
"ndArray" : {
"dataType" : "FLOAT",
"shape" : [ 1, 6, 10 ],
"data" : [ -0.0022880966, -0.0042849067, -0.005983479, -0.0073982426, -0.008555864, -0.009488584, -0.010229964, -0.010812048, -0.011263946,
-0.011611057, -0.0027362786, -0.005198746, -0.0072902446, -0.009000374, -0.010361125, -0.011421581, -0.012234278, -0.012848378, -0.013306688,
-0.013644846, 8.9187745E-4, 0.0012898755, 0.0014061421, 0.0013690761, 0.0012552886, 0.00110957, 9.573803E-4, 8.1235886E-4, 6.812691E-4, 5.6666887E-4,
-0.0029521661, -0.0049125706, -0.0062154937, -0.0070830043, -0.007662085, -0.008049844, -0.008310454, -0.008486291, -0.00860548, -0.008686648, -2.41272E-4,
-2.1998871E-4, -3.9213814E-5, 2.2318505E-4, 5.1378313E-4, 7.9911196E-4, 0.0010602918, 0.0012885022, 0.0014812968, 0.00164004, 0.0029523545, 0.0050065047,
0.006426977, 0.007400234, 0.008058914, 0.008497588, 0.008783782, 0.00896551, 0.009076695, 0.009141108 ]
}
}
```



