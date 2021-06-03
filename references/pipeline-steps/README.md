---
description: Various type of steps is available in the Pipeline of Konduit-Serving.
---

# Pipeline Steps

Konduit-Serving provides various types of steps included in Pipeline from pre-processing, serving Machine Learning or Deep Learning model and post-processing. This section offers all the configurations with descriptions that may help set your Pipeline Steps. The example of `PipelineSteps`: 

```text
inferenceConfiguration.pipeline(SequencePipeline.builder()
                .add(new ImageToNDArrayStep() //add ImageToNDArrayStep() into pipeline to set image to NDArray for input
                        .config(new ImageToNDArrayConfig() //image configuration
                                .width(28)
                                .height(28)
                                .dataType(NDArrayType.FLOAT)
                                .aspectRatioHandling(AspectRatioHandling.CENTER_CROP)
                                .includeMinibatchDim(true)
                                .channelLayout(NDChannelLayout.GRAYSCALE)
                                .format(NDFormat.CHANNELS_FIRST)
                                .normalization(ImageNormalization.builder().type(ImageNormalization.Type.SCALE).build())
                        )
                        .keys("image")
                        .outputNames("input_layer")
                        .keepOtherValues(true)
                        .metadata(false)
                        .metadataKey(ImageToNDArrayStep.DEFAULT_METADATA_KEY))
                .add(new Nd4jTensorFlowStep() //add Nd4jTensorFlowStep into pipeline
                        .modelUri(modelTrainResult.modelPath())
                        .inputNames(modelTrainResult.inputNames())
                        .outputNames(modelTrainResult.outputNames())
                ).add(new ClassifierOutputStep()
                        .inputName(modelTrainResult.outputNames().get(0))
                        .labels(Arrays.asList(labels.clone()))
                        .allProbabilities(false)
                ).build()
        );
```

Here are the references in this section:

