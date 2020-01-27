---
description: >-
  This page provides a Java example of inferencing  a model, built in Python
  with ONNX Runtime, a cross-platform, high performance scoring engine for
  machine learning models.
---

# Open Neural Network Exchange (ONNX)

The Open Neural Network Exchange (ONNX) format is supported by a number of deep learning frameworks, including PyTorch, CNTK, MXNet, etc,.

```java
import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.config.ServingConfig;
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.model.PythonConfig;
import ai.konduit.serving.pipeline.step.ImageLoadingStep;
import ai.konduit.serving.pipeline.step.PythonStep;
import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;
import org.apache.commons.io.FileUtils;
import org.datavec.python.PythonVariables;
import org.nd4j.linalg.io.ClassPathResource;
```

{% hint style="info" %} A reference Java project is provided in the Example repository ( <https://github.com/KonduitAI/konduit-serving-examples> ) with a Maven pom.xml dependencies file. If using the IntelliJ IDEA IDE, open the java folder as a Maven project and run the main function of InferenceModelStepONNX class. {% endhint %}

For the purposes of this example, we use ONNX model files from [Ultra-Light-Fast-Generic-Face-Detector-1MB](https://github.com/Linzaer/Ultra-Light-Fast-Generic-Face-Detector-1MB) by Linzaer, a lightweight facedetection model designed for edge computing devices.

## Python script with PyTorch and ONNX Runtime

Now that we have an optimized ONNX file, we can serve our model.

The following is the python script `onnxFacedetect.py` :

- transforms a [PIL](https://python-pillow.org/) image into a 240 x 320 image,
- casts it into a PyTorch Tensor,
- adds an extra dimension with [`unsqueeze`](https://pytorch.org/docs/stable/torch.html#torch.unsqueeze),
- casts the Tensor into a NumPy array, then
- returns the model's output with ONNX Runtime.

```python
import os
import numpy as np
from PIL import Image
import torchvision.transforms as transforms
import onnxruntime
from matplotlib.image import imread
dl_path = os.path.abspath("./src/main/resources/data/facedetector/facedetector.onnx")
sys.path.append(dl_path)
a,b,c,d=inputimage.shape
inputimage=inputimage.reshape(b,c,d)
im=np.array(inputimage)
image = Image.fromarray(im.astype('uint8'), 'RGB')
resize = transforms.Resize([240, 320])
img_y = resize(image)
to_tensor = transforms.ToTensor()
img_y = to_tensor(img_y)
img_y.unsqueeze_(0)

def to_numpy(tensor):
    return tensor.detach().cpu().numpy() if tensor.requires_grad else tensor.cpu().numpy()

ort_session = onnxruntime.InferenceSession(dl_path)
ort_inputs = {ort_session.get_inputs()[0].name: to_numpy(img_y)}
ort_outs = ort_session.run(None, ort_inputs)
_, boxes = ort_outs
```

## Configure the step

### Defining a `PythonConfig`

- Here we use the `pythonCodePath` argument instead of `pythonCode`, in order to specify the location of the Python script.
- Define the inputs and outputs name and type of the value as defined by the name() method of a PythonVariables.Type, here we use NDARRAY. See <https://serving.oss.konduit.ai/python> for supported data types.
- To run this example please install (PIL 6.21,numpy,matplotlib 3.1.2,onnxruntime 1.1.0, torchvision 0.4.2)and set the python path as `pythonPath(pythonPath)` in the `python_config` to refer the required Python libraries.

```java
 String pythonCodePath = new ClassPathResource("scripts/onnxFacedetect.py").getFile().getAbsolutePath();

String pythonPath = Arrays.stream(cachePackages())
        .filter(Objects::nonNull)
        .map(File::getAbsolutePath)
        .collect(Collectors.joining(File.pathSeparator));

PythonConfig python_config = PythonConfig.builder()
        .pythonCodePath(pythonCodePath)
        .pythonInput("inputimage", PythonVariables.Type.NDARRAY.name())
        .pythonOutput("boxes", PythonVariables.Type.NDARRAY.name())
        .pythonPath(pythonPath)
        .build();
```

### Define a pipeline step with the `PythonStep` class

In the `.step()` method, define the input configuration (`python_config`).

```java
PythonStep onnx_step = new PythonStep().step(python_config);
```

### Define a pipeline step with `ImageLoadingStep` class

A Pipeline Step for loading and transforming an image

```java
ImageLoadingStep imageLoadingStep = ImageLoadingStep.builder()
                .inputName("inputimage")
                .dimensionsConfig("default", new Long[]{478L, 720L, 3L}) // Height, width, channels
                .build();
```

## Configure the server

In the `ServingConfig`, specify any port number that is not reserved.

```java
int port = Util.randInt(1000, 65535);

ServingConfig servingConfig = ServingConfig.builder().httpPort(port).
    build();
```

The `ServingConfig` has to be passed to `Server` in addition to the `imageLoadingStep` as a list. In this case, there is a single step: `onnx_step`.

```java
InferenceConfiguration inferenceConfiguration = InferenceConfiguration.builder()
    .steps(Arrays.asList(imageLoadingStep, onnx_step)).servingConfig(servingConfig).build();
```

The `inferenceConfiguration` is stored as a JSON File.

```java
File configFile = new File("config.json");
FileUtils.write(configFile, inferenceConfiguration.toJson(), Charset.defaultCharset());
```

## Inference

Load a sample image and send the image as NDARRAY to the server for prediction.

{% hint style="info" %} Accepted input and output data formats are as follows:

- Input: JSON, ARROW, IMAGE, ND4J and NUMPY.
- Output: NUMPY, JSON, ND4J and ARROW. {% endhint %}

The `Client` should be configured to match the Konduit Serving instance. As this example is run on a local computer, the server is located at host `'http://localhost'` and port `port`. And Finally, we run the Konduit Serving instance with the saved **config.json** file path as `configPath` and other necessary server configuration arguments.

A Callback Function onSuccess is implemented in order to post the Client request and get the HttpResponse, only after the successful run of the KonduitServingMain Server.

```java
KonduitServingMain.builder()
    .onSuccess(() -> {
        try {
            HttpResponse<JsonNode> response = Unirest.post(String.format("http://localhost:%s/raw/json", port))
                    .header("Content-Type", "application/json")
                    .body("{\"first\" :\"value\"}").asJson();

            System.out.println(response.getBody().toString());
            System.exit(0);
        } catch (UnirestException e) {
            e.printStackTrace();
            System.exit(0);
        }
    })
    .build()
    .runMain("--configPath", configFile.getAbsolutePath());
```

## Confirm the output

After executing the above, in order to confirm the successful start of the Server, check for the below output text:

```text
Jan 08, 2020 6:33:47 PM ai.konduit.serving.configprovider.KonduitServingMain
INFO: Deployed verticle ai.konduit.serving.verticles.inference.InferenceVerticle
```

The Output of the program is as follows:

```java
System.out.println(response.getBody().toString());
```

```text
{
  "default" : {
    "batchId" : "78590ecd-3dd0-4fe4-9b9f-c6046e691207",
    "ndArray" : {
      "dataType" : "FLOAT",
      "shape" : [ 1, 4420, 4 ],
      "data" : [ 0.0053688805, 0.0025114473, 0.02034211, 0.03720106,
-0.0017913571, -0.008830132, 0.030541036, 0.062001586,
 -0.009721115, -0.02121478, 0.038849648,
......
......
 0.95745945, 1.198462, 0.379943, 0.39496967, 1.0312535,
 1.2312568, 0.7340108, 0.62931126, 1.0600785,
 1.100086, 0.65668, 0.49809134, 1.1420089, 1.2198907,
0.57574236, 0.38997138, 1.2035433, 1.2634758 ]
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
    "httpPort" : 26652,
    "listenHost" : "localhost",
    "logTimings" : false,
    "metricTypes" : [ "CLASS_LOADER", "JVM_MEMORY", "JVM_GC", "PROCESSOR", "JVM_THREAD", "LOGGING_METRICS", "NATIVE" ],
    "outputDataFormat" : "JSON",
    "uploadsDirectory" : "file-uploads/"
  },
  "steps" : [ {
    "@type" : "ImageLoadingStep",
    "dimensionsConfigs" : {
      "default" : [ 478, 720, 3 ]
    },
    "imageProcessingInitialLayout" : null,
    "imageProcessingRequiredLayout" : null,
    "imageTransformProcesses" : { },
    "inputColumnNames" : { },
    "inputNames" : [ "inputimage" ],
    "inputSchemas" : { },
    "objectDetectionConfig" : null,
    "originalImageHeight" : 0,
    "originalImageWidth" : 0,
    "outputColumnNames" : { },
    "outputNames" : [ ],
    "outputSchemas" : { },
    "updateOrderingBeforeTransform" : false
  }, {
    "@type" : "PythonStep",
    "inputColumnNames" : {
      "default" : [ "inputimage" ]
    },
    "inputNames" : [ "default" ],
    "inputSchemas" : {
      "default" : [ "NDArray" ]
    },
    "outputColumnNames" : {
      "default" : [ "boxes" ]
    },
    "outputNames" : [ "default" ],
    "outputSchemas" : {
      "default" : [ "NDArray" ]
    },
    "pythonConfigs" : {
      "default" : {
        "extraInputs" : { },
        "pythonCode" : null,
        "pythonCodePath" : "C:\\Projects\\konduit-serving-examples\\java\\target\\classes\\scripts\\onnxFacedetect.py",
        "pythonInputs" : {
          "inputimage" : "NDARRAY"
        },
        "pythonOutputs" : {
          "boxes" : "NDARRAY"
        },
        "pythonPath" : "C:\\Users\\AppData\\Local\\Programs\\Python\\Python37\\python37.zip;C:\\Users\\venkat-nidrive`AppData\\Local\\Programs\\Python\\Python37\\DLLs;C:\\Users\\AppData\\Local\\Programs\\Python\\Python37\\lib;C:\\Users\\AppData\\Local\\Programs\\Python\\Python37;C:\\Users\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages;C:\\Users\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\pyyaml-5.2-py3.7-win-amd64.egg;c:\\projects\\konduit-serving\\python",
        "returnAllInputs" : false,
        "setupAndRun" : false
      }
    }
  } ]
}
```
