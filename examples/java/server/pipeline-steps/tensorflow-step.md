---
description: Example of applying Tensorflow Step
---

# Tensorflow Step

In this example, we include a series of steps in the core abstraction, a pipeline step. The pipeline step performs a task such as:

* Pre-processing step
* Running a model
* Post-processing by transforming the output in a way that humans can understand

Once the pipeline is set up, the server can deploy the model.

```java
import ai.konduit.serving.data.image.convert.ImageToNDArrayConfig;
import ai.konduit.serving.data.image.convert.config.AspectRatioHandling;
import ai.konduit.serving.data.image.convert.config.ImageNormalization;
import ai.konduit.serving.data.image.convert.config.NDChannelLayout;
import ai.konduit.serving.data.image.convert.config.NDFormat;
import ai.konduit.serving.data.image.step.ndarray.ImageToNDArrayStep;
import ai.konduit.serving.examples.utils.Train;
import ai.konduit.serving.models.nd4j.tensorflow.step.Nd4jTensorFlowStep;
import ai.konduit.serving.pipeline.api.data.NDArrayType;
import ai.konduit.serving.pipeline.impl.pipeline.SequencePipeline;
import ai.konduit.serving.pipeline.impl.step.ml.classifier.ClassifierOutputStep;
import ai.konduit.serving.vertx.api.DeployKonduitServing;
import ai.konduit.serving.vertx.config.InferenceConfiguration;
import ai.konduit.serving.vertx.config.InferenceDeploymentResult;
import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;
import io.vertx.core.DeploymentOptions;
import io.vertx.core.VertxOptions;
import org.nd4j.common.io.ClassPathResource;
import java.io.IOException;
import java.util.Arrays;
```

### Configure the pipeline step

Let's start from the main function by declare the variable of labels that will be used by `ClassifierOutputStep()` and getting the trained model.

```java
String[] labels = {"0","1","2","3","4","5","6","7","8","9"};

//get the file of trained model
Train.ModelTrainResult modelTrainResult = Train.tensorflowMnistModel();
```

Create an inference configuration by default.

```java
//a default Inference Configuration
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();
```

We'll need to include pre-processing step using `ImageToNDArrayStep()` to convert the input image into an array and specify all characteristics of the input image. To run a model, add `Nd4jTensorFlowStep()` into the pipeline and specify `modelUri`, `inputNames` and `outputNames`. `ClassifierOutputStep()` can be included for transforming the output in a way that humans can understand and set the following:

* `inputName` : names for model's output layer
* `labels` : list of output labels classifier
* `allProbabilities` : false

```java
//include pipeline step into the Inference Configuration
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

### Deploy the server

Let's deploy the model in the server by calling  `DeployKonduitServing` with the configuration made before. The handler, a callback function, is applied, only after a successful or failed server deployment inside the handler block, as shown.

```java
//deploy the model in server
DeployKonduitServing.deploy(new VertxOptions(), new DeploymentOptions(),
        inferenceConfiguration,
        handler -> {
            if (handler.succeeded()) { // If the server is successfully running
                // Getting the result of the deployment
                InferenceDeploymentResult inferenceDeploymentResult = handler.result();
                int runnningPort = inferenceDeploymentResult.getActualPort(); //get server's port
                String deploymentId = inferenceDeploymentResult.getDeploymentId(); //get server's deployment id

                System.out.format("The server is running on port %s with deployment id of %s%n",
                        runnningPort, deploymentId);

                try {
                    String result;
                    try {
                        result = Unirest.post(String.format("http://localhost:%s/predict", runnningPort))
                                .header("Accept", "application/json")
                                .field("image", new ClassPathResource("inputs/test_files/test_input_number_2.png").getFile(), "image/png")
                                .asString().getBody();

                        System.out.format("Result from server : %s%n", result);

                        System.exit(0);
                    } catch (IOException e) {
                        e.printStackTrace();
                        System.exit(1);
                    }
                } catch (UnirestException e) {
                    e.printStackTrace();
                    System.exit(1);
                }
            } else { // If the server failed to run
                System.out.println(handler.cause().getMessage());
                System.exit(1);
            }
        });
```

Note that we consider only one test input image in this example for inference to show the model's deployment in Konduit-Serving. After implementation, the successful server deployment gives below output text.

```aspnet
The server is running on port 40521 with deployment id of 4b7d8bc5-a711-499f-ad9f-9ccd82c3e142
Result from server : {
  "output_layer/Softmax" : [ [ 1.8688768E-8, 0.0962552, 0.7753802, 1.7737559E-8, 0.122773424, 4.7498935E-13, 3.0434896E-6, 0.005588151, 7.329317E-12, 3.4533626E-10 ] ],
  "prob" : 0.7753801941871643,
  "index" : 2,
  "label" : "2"
}

Process finished with exit code 0
```

The complete inference configuration in YAML format is as follows.

```java
System.out.format(inferenceConfiguration.toYaml());
```

```aspnet
---
host: "localhost"
port: 0
use_ssl: false
protocol: "HTTP"
static_content_root: "static-content"
static_content_url: "/static-content"
static_content_index_page: "/index.html"
kafka_configuration:
  start_http_server_for_kafka: true
  http_kafka_host: "localhost"
  http_kafka_port: 0
  consumer_topic_name: "inference-in"
  consumer_key_deserializer_class: "io.vertx.kafka.client.serialization.JsonObjectDeserializer"
  consumer_value_deserializer_class: "io.vertx.kafka.client.serialization.JsonObjectDeserializer"
  consumer_group_id: "konduit-serving-consumer-group"
  consumer_auto_offset_reset: "earliest"
  consumer_auto_commit: "true"
  producer_topic_name: "inference-out"
  producer_key_serializer_class: "io.vertx.kafka.client.serialization.JsonObjectSerializer"
  producer_value_serializer_class: "io.vertx.kafka.client.serialization.JsonObjectSerializer"
  producer_acks: "1"
mqtt_configuration: {}
custom_endpoints: []
pipeline:
  steps:
  - '@type': "IMAGE_TO_NDARRAY"
    config:
      height: 28
      width: 28
      dataType: "FLOAT"
      includeMinibatchDim: true
      aspectRatioHandling: "CENTER_CROP"
      format: "CHANNELS_FIRST"
      channelLayout: "GRAYSCALE"
      normalization:
        type: "SCALE"
      listHandling: "NONE"
    keys:
    - "image"
    outputNames:
    - "input_layer"
    keepOtherValues: true
    metadata: false
    metadataKey: "@ImageToNDArrayStepMetadata"
  - '@type': "ND4JTENSORFLOW"
    input_names:
    - "input_layer"
    output_names:
    - "output_layer/Softmax"
    constants: {}
    model_uri: "/home/zulfadzli/KS2/konduit-serving-examples/java/target/classes/models/tensorflow/mnist/tensorflow.pb"
  - '@type': "CLASSIFIER_OUTPUT"
    input_name: "output_layer/Softmax"
    return_label: true
    return_index: true
    return_prob: true
    label_name: "label"
    index_name: "index"
    prob_name: "prob"
    labels:
    - "0"
    - "1"
    - "2"
    - "3"
    - "4"
    - "5"
    - "6"
    - "7"
    - "8"
    - "9"
    all_probabilities: false
```

