---
description: Custom model in Konduit-Serving with HTML content
---

# Serving a BMI Model

### Introduction

Body Mass Index \(BMI\) is a well-used measure to describe weight status based on the ratio between an individual's height and weight. BMI is used to classify an individual's weight status as underweight, normal weight, overweight or obese. Even though this is a simple application to be used, it is necessary to monitor healthcare by providing weight status generally.

Findings from [Pursey et al.](https://pubmed.ncbi.nlm.nih.gov/24398335/) and [Stommel et al.](https://bmcpublichealth.biomedcentral.com/articles/10.1186/1471-2458-9-421) showed that adults tend to self-report inaccurate BMI, overestimate their weight, and underestimate the weight. Time-consuming in measuring weight and height also one of the factors that contribute to this issue. Thus, the BMI model is introduced as a new approach for estimating BMI using facial images that contain facial features.

The model is deployed in **Konduit-Serving** through the pipeline server from pre-processing step of input until producing output that a human can understand. The backend server used for taking image data and providing BMI values is served using Konduit-Serving, a high-performance model pipeline server.

The main workflow we'll look at in this document is how to serve a model that can see at a person's face and respond with a BMI value through REST API. We'll also be setting up a web server through Konduit-Serving "custom endpoints" that will make use of a webcam and label the canvas with the detected face along with the corresponding estimated BMI value.

Note that gathering, preparing dataset and model training are out of the scope of this document. Assuming you're on notebook environment and opened `bmi-onnx-pytorch.ipynb`.

{% hint style="info" %}
The notebook is ready to use from [https://github.com/ShamsUlAzeem/konduit-serving-demo/blob/master/demos/6-bmi-onnx-pytorch/bmi-onnx-pytorch.ipynb](https://github.com/ShamsUlAzeem/konduit-serving-demo/blob/master/demos/6-bmi-onnx-pytorch/bmi-onnx-pytorch.ipynb). Please follow [Quickstart](../../quickstart/using-docker.md) guide to run the notebook. 
{% endhint %}

### Start the server

Let's serve the model via Konduit-Serving with provided configuration file and python scripts. The `serve` command is as following, serving id name \(can be any other preferred name\) is `bmi-onnx-pytorch` and configuration file is `bmi-onnx-pytorch.yaml`. 

```text
%%bash
nohup konduit serve --serving-id bmi-onnx-pytorch --config bmi-onnx-pytorch.yaml &
```

Once the command committed, the server is started in Konduit-Serving. We are using Python Step in Pipeline from the configuration file,, where the step run by python scripts `init_script.py` and `run_script.py`. The step calls the model, `version-RFB-320.onnx` from the model directory, `konduit-serving-demo/demos/6-bmi-oonx-pytorch/models`and perform the BMI classification based on facial feature captured on the test image. You can view any scripts, either configuration or python, by adding the cell on the notebook and run,

```text
%%bash
less bmi-onnx-pytorch.yaml
less run_script.py
```

### View the logs

The command to view the logs from the server is `konduit logs` command, and we'll view the last 300 lines by using `--lines` flag with the command. By default, the konduit logs command will only preview ten lines of the logs. The command is like the following:

```text
%%bash
konduit logs bmi-onnx-pytorch --lines 300
```

You can view the logs output similar to below.

```text
06:53:46.317 [main] INFO  a.k.s.c.l.command.KonduitRunCommand - Processing configuration: /root/konduit/demos/6-bmi-onnx-pytorch/bmi-onnx-pytorch.yaml
.
.
. 

####################################################################
#                                                                  #
#    |  /   _ \   \ |  _ \  |  | _ _| __ __|    |  /     |  /      #
#    . <   (   | .  |  |  | |  |   |     |      . <      . <       #
#   _|\_\ \___/ _|\_| ___/ \__/  ___|   _|     _|\_\ _) _|\_\ _)   #
#                                                                  #
####################################################################

.
.
.
06:53:48.703 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server is listening on host: '0.0.0.0'
06:53:48.703 [vert.x-eventloop-thread-0] INFO  a.k.s.v.p.h.v.InferenceVerticleHttp - Inference HTTP server started on port 9009 with 2 pipeline steps
```

### View a list of server

The `konduit list` command is used to check the running server from Konduit-Serving. If you serve another model on the server, it will show more than one ID based on the specified given name. Run this command to view the list of the servers.

```text
%%bash
konduit list
```

### Make an inference

We can display the test image for the inference result of the model. You'll be able to see the picture we provided as the testing image by running the following.

```text
%%html
<img src="image_me.jpg"/>
```

Now, we want to make the inference of the testing image. The `konduit predict` command is used to classify the BMI of the person in the testing image and classify the output based on the label of highest probability.

```text
%%bash
konduit predict bmi-onnx-pytorch --input-type multipart "image=@image_me.jpg"
```

If we are viewing the configuration file, the post-processing \(`CLASSIFIER_OUTPUT`\) is taking place to classify the output based on labels rendered from the output layer of the served model. You'll get a result similar to the following.

```text
{
  "bmi_value" : 22.18,
  "boxes" : [ 447.0, 174.0, 636.0, 470.0 ],
  "predictions" : [ 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0 ],
  "prob" : 1.0,
  "index" : 1,
  "bmi_class" : "Normal_Range"
}
```

The test image is not limited to provide one, and you can use your testing image to make the inference. Let's do something interesting by following the next step. 

### Before stopping the server

You can demonstrate the served model with HTML content to view a dashboard and test the model with your camera. Let's move to the sub-section to find more.

{% page-ref page="with-html-content.md" %}

### Stop the server

Let's stop the server once finished to serve on Konduit-Serving.

```text
%%bash
konduit stop bmi-onnx-pytorch
```

As the server is stopped, you'll see a message similar to the following.

```text
Stopping konduit server 'bmi-onnx-pytorch'
Application 'bmi-onnx-pytorch' terminated with status 0
```

