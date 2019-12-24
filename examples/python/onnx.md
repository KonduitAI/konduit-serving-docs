---
description: >-
  This notebook provides an example of serving a model built in PyTorch with
  ONNX Runtime, a cross-platform, high performance scoring engine for machine
  learning models.
---

# Open Neural Network Exchange \(ONNX\)

The Open Neural Network Exchange \(ONNX\) format is supported by a number of deep learning frameworks, including PyTorch, CNTK and MXNet.

```python
import os 
from urllib.request import urlretrieve 
import sys 
import numpy as np 
from PIL import Image 

import onnx
from onnx import optimizer

from konduit.utils import default_python_path
```

This page documents two ways to create Konduit Serving configurations with the Python SDK:

1. Using Python to create a configuration, and 
2. Writing the configuration as a YAML file, then serving it using the Python SDK. 

These approaches are documented in separate tabs throughout this page. For example, the following code block shows the imports for each approach in separate tabs:

{% tabs %}
{% tab title="Python" %}
```python
from konduit import PythonConfig, ServingConfig, InferenceConfiguration, \
PythonStep
from konduit.server import Server
from konduit.client import Client 
```
{% endtab %}

{% tab title="YAML" %}
```python
from konduit.load import server_from_file, client_from_file
```
{% endtab %}
{% endtabs %}

## Download file

For the purposes of this example, we use ONNX model files from [Ultra-Light-Fast-Generic-Face-Detector-1MB](https://github.com/Linzaer/Ultra-Light-Fast-Generic-Face-Detector-1MB) by Linzaer, a lightweight facedetection model designed for edge computing devices.

```python
dl_path = os.path.abspath("../data/facedetector/facedetector.onnx")
DOWNLOAD_URL = "https://raw.githubusercontent.com/Linzaer/Ultra-Light-Fast-Generic-Face-Detector-1MB/master/models/onnx/version-RFB-320.onnx"
if not os.path.isfile(dl_path):
    urlretrieve(DOWNLOAD_URL, filename=dl_path)
```

The following content is based on the PyTorch tutorial [Exporting a Model from PyTorch to ONNX and Running it using ONNX Runtime](https://pytorch.org/tutorials/advanced/super_resolution_with_onnxruntime.html), with modifications.

We start by loading the model and running `onnx.checker.check_model` to check whether the model has a valid schema.

```python
# Load the ONNX model
model = onnx.load(dl_path)
# model is a onnx.ModelProto object 

onnx.checker.check_model(model)
```

## Optimize

When loading some models, ONNX may return warnings that the model can be further optimized by removing some unused nodes.

Use ONNX's optimizer to optimize your ONNX file. The code below is adapted from this [GitHub comment](https://github.com/microsoft/onnxruntime/issues/1899#issuecomment-534806537).

Note that the API for optimizing models in ONNX Runtime is experimental, and [may change](https://github.com/onnx/onnx/blob/c08a7b76cf7c1555ae37186f12be4d62b2c39b3b/onnx/optimizer/optimize.h#L1-L2).

```python
onnx_model = onnx.load(dl_path)
passes = ["extract_constant_to_initializer", "eliminate_unused_initializer"]
optimized_model = optimizer.optimize(onnx_model, passes)
onnx.save(optimized_model, dl_path)
```

## Python script with PyTorch and ONNX Runtime

Now that we have an optimized ONNX file, we can serve our model.

The following code:

* transforms a [PIL](https://python-pillow.org/) image into a 240 x 320 image, 
* casts it into a PyTorch Tensor, 
* adds an extra dimension with [`unsqueeze`](https://pytorch.org/docs/stable/torch.html#torch.unsqueeze), 
* casts the Tensor into a NumPy array, then 
* returns the model's output with ONNX Runtime. 

```python
python_code = """

from PIL import Image 
import torchvision.transforms as transforms
import onnxruntime
import os 

dl_path = os.path.abspath("../data/facedetector/facedetector.onnx")

image = Image.fromarray(image.astype('uint8'), 'RGB')
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

"""
```

## Configure the step

{% tabs %}
{% tab title="Python" %}
### Defining a `PythonConfig`

* Here we use the `python_code` argument instead of `python_code_path`, since the code is defined as a string. 
* Define the inputs and outputs as dictionaries, where the keys represent objects in the server's Python environment, and the values represent data types \(Python data structures\), defined as strings. See [https://serving.oss.konduit.ai/python](https://serving.oss.konduit.ai/python) for supported data types. 

```python
work_dir = os.path.abspath('.')

python_config = PythonConfig(
    python_code=python_code,
    python_inputs={"image": "NDARRAY"}, 
    python_outputs={"boxes": "NDARRAY"}, 
    python_path=default_python_path(work_dir)
)
```

### Define a pipeline step with the `PythonStep` class.

In the `.step()` method, define a name for this step \(`input1`\) and the respective configuration \(`python_config`\).

```python
onnx_step = (PythonStep()
             .step(input_name="input1", 
                   python_config=python_config))
```
{% endtab %}

{% tab title="YAML" %}
```yaml
steps:
  python_step:
    type: PYTHON
    python_path: C:\\Users\\Skymind AI Berhad\\Documents\\konduit-serving-examples\\notebooks;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\python37.zip;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\DLLs;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch;;C:\\Users\\Skymind AI Berhad\\AppData\\Roaming\\Python\\Python37\\site-packages;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages\\konduit-0.1.4-py3.7.egg;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages\\pyyaml-5.1.2-py3.7-win-amd64.egg;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages\\win32;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages\\win32\\lib;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages\\Pythonwin;C:\\Users\\Skymind AI Berhad\\AppData\\Local\\Continuum\\miniconda3\\envs\\pytorch\\lib\\site-packages\\IPython\\extensions;C:\\Users\\Skymind AI Berhad\\.ipython;C:\\Users\\Skymind AI Berhad\\Documents\\konduit-serving-examples\\notebooks
    python_code: |
      from PIL import Image 
      import torchvision.transforms as transforms
      import onnxruntime
      import os 

      dl_path = os.path.abspath("../data/facedetector/facedetector.onnx")

      image = Image.fromarray(image.astype('uint8'), 'RGB')
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
      
    python_inputs:
      image: NDARRAY
    python_outputs:
      boxes: NDARRAY
```

We define a single `python_step` of type PYTHON. 

* `python_path` specifies the location of Python modules. 
* `python_code` specifies the Python code to be run. Here, we use a YAML literal block scalar.
* `python_inputs` and `python_outputs`specifies the data type of the objects in the Python script to be used as input\(s\) and output\(s\) respectively.

{% hint style="info" %}
Models loaded from a YAML configuration do not currently support input and output names for Python steps. To construct configurations with custom input and output names, use the Python SDK. 
{% endhint %}

{% hint style="info" %}
The default Python path includes NumPy and a basic set of modules. However, for this example, we require a Python path that contains PyTorch and ONNX Runtime. See the [Python pipeline steps page](../../steps/python.md#python-modules-and-the-pythonpath-argument) for additional documentation on Python paths, and refer to the [PyTorch quickstart](https://pytorch.org/) for recommended installation steps.

To locate your Python path, run the following:

```python
from konduit.utils import default_python_path
work_dir = os.path.abspath('.')
print(default_python_path(work_dir))
```
{% endhint %}
{% endtab %}
{% endtabs %}

## Configure the server

{% tabs %}
{% tab title="Python" %}
```python
port = np.random.randint(1000, 65535)

server = Server(
    steps=onnx_step, 
    serving_config=ServingConfig(http_port=port)
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
  log_timings: True
  extra_start_args: -Xmx8g

```
{% endtab %}
{% endtabs %}

## Start the server

{% tabs %}
{% tab title="Python" %}
```python
server.start()
```

```text
Starting server.........

Server has started successfully.
```
{% endtab %}

{% tab title="YAML" %}
```python
konduit_yaml_path = "../yaml/pytorch.yaml"
server = server_from_file(konduit_yaml_path)
server.start()
```
{% endtab %}
{% endtabs %}

## Configure the client

Make sure to configure the client after starting the server, so that the Client object can inherit the Server's attributes. 

{% tabs %}
{% tab title="Python" %}
```python
client = Client(port=port)
```
{% endtab %}

{% tab title="YAML" %}
Add the following to your YAML configuration file: 

```yaml
client:
  port: 1337
```

Use `client_from_file` to create a `Client` object: 

```python
konduit_yaml_path = "../yaml/pytorch.yaml"
client = client_from_file(konduit_yaml_path)
```
{% endtab %}
{% endtabs %}

## Inference

Load a sample image using PIL/Pillow and send the image to the server for prediction using the `predict()` method of the `Client` class.

```python
im = Image.open("../data/facedetector/1.jpg")
im = np.array(im).astype("int")
```

```python
output = client.predict(
    {"input1": im}
)
print(output)
```

```text
[[[ 0.00601701  0.00688479  0.02177745  0.03408115]
  [-0.0018133  -0.00657785  0.03698186  0.05206966]
  [-0.01035942 -0.01786287  0.04902049  0.06799769]
  ...
  [ 0.7294515   0.6165271   1.0584102   1.1059598 ]
  [ 0.65046376  0.48442802  1.141786    1.2248938 ]
  [ 0.5633501   0.37209463  1.2047783   1.2747201 ]]]
```



Finally, we stop the server:

```python
server.stop()
```

