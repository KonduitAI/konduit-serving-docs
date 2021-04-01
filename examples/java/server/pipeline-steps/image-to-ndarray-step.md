---
description: As pre-processing step
---

# Image To NDArray Step

Image To NDArray Step is used as a pre-processing step in Pipeline Step to manipulate or convert the image into an _n-_D array based on the model requirement input layer or the next pipeline step. We can include `ImageToNDArrayStep()` in pipeline into inference configuration.

```java
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();

inferenceConfiguration.pipeline(SequencePipeline.builder()
                .add(new ImageToNDArrayStep() //add ImageToNDArrayStep() into pipeline to set image to NDArray for input
                        .config(new ImageToNDArrayConfig() //image configuration
                                .width(28)
                                .height(28)
                                .includeMinibatchDim(true)
                                .channelLayout(NDChannelLayout.GRAYSCALE)
                                .format(NDFormat.CHANNELS_LAST)
                        )
                        .keys("image")
                        .outputNames("input_layer"))
                ).build()
        );
```

From `ImageToNDArrayConfig()` in the above, the input image will have 28 by 28 shape size and convert to a 3-D array. Set mini batch dimension to true, and the channel has a depth of 1 for grayscale, put at last of a list such \[1, 28, 28, 1\].

{% hint style="info" %}
The shape array such \[minibatch\_dim, width, height, channels\] if format is _CHANNELS\_LAST_ .
{% endhint %}

```java
inferenceConfiguration.pipeline(SequencePipeline.builder()
        .add(new ImageToNDArrayStep() //add ImageToNDArrayStep() into pipeline to set image to NDArray for input
                .config(new ImageToNDArrayConfig() //image configuration
                        .width(28)
                        .height(28)
                        .includeMinibatchDim(true)
                        .channelLayout(NDChannelLayout.GRAYSCALE)
                        .format(NDFormat.CHANNELS_FIRST)
                        .normalization(ImageNormalization.builder().type(ImageNormalization.Type.SCALE).build())
                )
                .keys("image")
                .outputNames("input_layer")
        ).build()
);
```

We'll be able to add data normalization and change channel element position to first which will give the input of \[1, 1, 28, 28\] to the model. Failure to give an image with characteristics mentioned in configuration will affect the deployment of model in server and return an error. You can see the similar configuration in [Keras Step](keras-step.md) and [Tensorflow Step](tensorflow-step.md).  

