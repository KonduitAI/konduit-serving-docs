---
description: >-
  This notebook illustrates a simple client-server interaction to perform
  inference on a TensorFlow model using the Java SDK for Konduit Serving.
---

# MNIST

This tutorial is split into two parts:

1. Configuration 
2. Running the server

{% hint style="info" %}
This tutorial is tested on TensorFlow 1.14, 1.15 and 2.00.

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

```text
Using TensorFlow backend.
```


## Overview

Konduit Serving works by defining a series of **steps**. These include operations such as 

1. Pre- or post-processing steps 
2. One or more machine learning models 
3. Transforming the output in a way that can be understood by humans

If deploying your model does not require pre- nor post-processing, only one step - a machine learning model - is required. This configuration is defined using a single `ModelStep`.

Before running this notebook, run the `build_jar.py` script and copy the JAR \(`konduit.jar`\) to this folder. Refer to the [Python SDK README](https://github.com/KonduitAI/konduit-serving/blob/master/python/README.md) for details.

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
* `input_names`:  names for the input data  
* `output_names`: names for the output data

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

```java
# make note of hwo to obtain inputNames and outputNames
input_data_types = {'input_layer': 'FLOAT'}
input_names = list(input_data_types.keys())
output_names = ["output_layer/Softmax"]
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

The `ServingConfig` has to be passed to `Server` in addition to the steps as a Python list. In this case, there is a single step: `tf_step`.

```python
server = Server(
    serving_config=serving_config,
    steps=[tf_step]
)
```

By default, `Server()` looks for the Konduit Serving JAR `konduit.jar` in the directory the script is run in. To change this default, use the `jar_path` argument.

{% hint style="info" %}
Accepted input and output data formats are as follows:

* Input: JSON, ARROW, IMAGE, ND4J \(not yet implemented\) and NUMPY.
* Output: NUMPY, JSON, ND4J \(not yet implemented\) and ARROW.

{% end hint style %}

## Start the server

Start the server:

```java
server.start()
```

```text
Starting server.....

Server has started successfully.

<subprocess.Popen at 0x1d6794ffd68>
```


## Configure the client

To configure the client, create a Client object with the following arguments:
* `httpPort`: Assign a random port to “httpPort.
* `inputDataFormat`: data format passed to the server for inference
* `outputDataFormat`: data format returned by the server endpoint 
* `return_output_data_format`: data format to be returned to the client. Note that this argument can be used to convert the output returned from the server to the client into a different format, e.g. NUMPY to JSON.

```java
ServingConfig servingConfig = ServingConfig.builder().httpPort(port).
                inputDataFormat(Input.DataFormat.NUMPY).
                outputDataFormat(Output.DataFormat.NUMPY).
                build();
```

## Inference 

{% hint style="warning" %}
NDARRAY inputs to ModelSteps must be specified with a preceding `batchSize` dimension. For batches with a single observation, this can be done by using `np.expand_dims()` to add an additional dimension to your array. 
{% end hint style %}

We obtain test images from the test set defined by `keras.datasets`.

```java
public class InferenceModelStepMNISTTest {
    public static void main(String[] args) throws Exception {
        String str = new ClassPathResource("images/COCO_train2014_000000000009.jpg").getFile().getAbsolutePath();

        File imageFile= new File (str);
        String output = Unirest.post("http://localhost:3000/RAW/IMAGE")
                .field("input_layer", imageFile)
                .asString().getBody();

        File outputImagePath = new File("src/main/resources/data/test-image-output.zip");
        FileUtils.writeStringToFile(outputImagePath, output, Charset.defaultCharset());

        System.out.println(BinarySerde.readFromDisk(outputImagePath));
```

![png](../../../.gitbook/assets/output_28_0.png)

```text
{0: 0.0, 1: 0.0, 2: 0.001, 3: 0.001, 4: 0.0, 5: 0.0, 6: 0.0, 7: 0.998, 8: 0.0, 9: 0.0}
```

![png](../../../.gitbook/assets/output_28_2.png)

```text
{0: 0.0, 1: 0.0, 2: 0.998, 3: 0.002, 4: 0.0, 5: 0.0, 6: 0.0, 7: 0.0, 8: 0.0, 9: 0.0}
```

![png](../../../.gitbook/assets/output_28_4.png)

```text
{0: 0.0, 1: 0.986, 2: 0.005, 3: 0.0, 4: 0.0, 5: 0.0, 6: 0.0, 7: 0.005, 8: 0.003, 9: 0.0}
```

### Batch prediction

To predict in batches, the `data_input` dictionary has to be specified differently for client images in NDARRAY format. To input a batch of observations, ensure that your inputs are in the NCHW format: number of observations, channels \(optional if single channel\), height and width. An example is as follows:

```python
predicted = client.predict(
    data_input={'input_layer': x_test[0:3].reshape(3, 28, 28)}
)

server.stop()
```

We compare the predicted probabilities and the corresponding labels:

```python
pd.DataFrame(predicted).round(3)
```

|  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 0 | 0.0 | 0.000 | 0.001 | 0.001 | 0.0 | 0.0 | 0.0 | 0.998 | 0.000 | 0.0 |
| 1 | 0.0 | 0.000 | 0.998 | 0.002 | 0.0 | 0.0 | 0.0 | 0.000 | 0.000 | 0.0 |
| 2 | 0.0 | 0.986 | 0.005 | 0.000 | 0.0 | 0.0 | 0.0 | 0.005 | 0.003 | 0.0 |

```python
y_test[0:3]
```

```text
array([7, 2, 1], dtype=uint8)
```



The configuration is stored as a dictionary. Note that the configuration can be converted to a dictionary using the `as_dict()` method:

```java
server.config.as_dict()
```

```text
{'@type': 'InferenceConfiguration',
 'steps': [{'@type': 'ModelStep',
   'inputNames': ['input_layer'],
   'outputNames': ['output_layer/Softmax'],
   'modelConfig': {'@type': 'TensorFlowConfig',
    'tensorDataTypesConfig': {'@type': 'TensorDataTypesConfig',
     'inputDataTypes': {'input_layer': 'FLOAT'}},
    'modelConfigType': {'@type': 'ModelConfigType',
     'modelType': 'TENSORFLOW',
     'modelLoadingPath': 'C:\\Users\\Skymind AI Berhad\\Documents\\konduit-serving-examples\\data\\mnist\\mnist_2.0.0.pb'}},
   'parallelInferenceConfig': {'@type': 'ParallelInferenceConfig',
    'workers': 1}}],
 'servingConfig': {'@type': 'ServingConfig',
  'httpPort': 4776,
  'inputDataFormat': 'NUMPY',
  'outputDataFormat': 'NUMPY',
  'logTimings': True}}
```



