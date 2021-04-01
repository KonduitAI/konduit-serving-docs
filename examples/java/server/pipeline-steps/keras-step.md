---
description: Example of applying Keras Step
---

# Keras Step

The example starts with configuring the pipeline step in the inference configuration and then deploying the server using the Keras model with Konduit-Serving.

```java
import ai.konduit.serving.data.image.convert.ImageToNDArrayConfig;
import ai.konduit.serving.data.image.convert.config.NDChannelLayout;
import ai.konduit.serving.data.image.convert.config.NDFormat;
import ai.konduit.serving.data.image.step.ndarray.ImageToNDArrayStep;
import ai.konduit.serving.examples.utils.Train;
import ai.konduit.serving.models.deeplearning4j.step.keras.KerasStep;
import ai.konduit.serving.pipeline.impl.pipeline.SequencePipeline;
import ai.konduit.serving.vertx.api.DeployKonduitServing;
import ai.konduit.serving.vertx.config.InferenceConfiguration;
import ai.konduit.serving.vertx.config.InferenceDeploymentResult;
import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;
import io.vertx.core.DeploymentOptions;
import io.vertx.core.VertxOptions;
import org.nd4j.common.io.ClassPathResource;
```

### Configure the step

Let's start from the main function by getting the trained model.

```java
//get the file of trained model
Train.ModelTrainResult modelTrainResult = Train.kerasMnistModel();
```

Create a default inference configuration that the server will use.

```java
//a default Inference Configuration
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();
```

Add `ImageToNDArrayStep()` which are pre-processing step into the pipeline as a need to convert an input image to an array and must specified with a shape size. We'll also need to include `KerasStep` into the pipeline of the inference configuration. Specify the following:

* `modelUri` : the model file path
* `inputNames` : names for model's input layer
* `outputNames` : names for model's output layer

```java
//include pipeline step into the Inference Configuration
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
        .add(new KerasStep() //add KerasStep into pipeline
                .modelUri(modelTrainResult.modelPath())
                .inputNames(modelTrainResult.inputNames())
                .outputNames(modelTrainResult.outputNames())
        ).build()
);
```

### Deploy the server

Let's deploy the model in the server by calling `DeployKonduitServing` with the configuration made before. A callback function is implemented to get a response only after a successful or failed server deployment inside the handler block.

```java
//deploy the model in server
DeployKonduitServing.deploy(new VertxOptions(), new DeploymentOptions(),
        inferenceConfiguration,
        handler -> {
            if (handler.succeeded()) { // If the server is sucessfully running
                // Getting the result of the deployment
                InferenceDeploymentResult inferenceDeploymentResult = handler.result();
                int runnningPort = inferenceDeploymentResult.getActualPort();
                String deploymentId = inferenceDeploymentResult.getDeploymentId();

                System.out.format("The server is running on port %s with deployment id of %s%n",
                        runnningPort, deploymentId);

                try {
                    String result;
                    try {
                        result = Unirest.post(String.format("http://localhost:%s/predict", runnningPort))
                                .header("Accept", "application/json")
                                .field("image", new ClassPathResource("inputs/mnist-image-2.jpg").getFile(), "image/jpg")
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

Note that we consider only one test input image in this example for inference to show the model's deployment in Konduit-Serving. After the above execution, you can check for the below output to confirm the successful server deployment.

```aspnet
The server is running on port 46233 with deployment id of f9b7a616-2d54-4814-86ac-1f888052ef34
Result from server : {
  "output_layer" : [ [ 6.086365E-10, 6.585195E-11, 7.845706E-7, 1.8983503E-6, 2.3600207E-11, 1.6447022E-8, 4.0799048E-11, 2.2013203E-12, 0.99999714, 6.1216234E-8 ] ]
}

Process finished with exit code 0
```

The complete inference configuration in JSON format is as follows.

```java
System.out.format(inferenceConfiguration.toJson());
```

```aspnet
{
  "host" : "localhost",
  "port" : 0,
  "useSsl" : false,
  "protocol" : "HTTP",
  "staticContentRoot" : "static-content",
  "staticContentUrl" : "/static-content",
  "staticContentIndexPage" : "/index.html",
  "kafkaConfiguration" : {
    "startHttpServerForKafka" : true,
    "httpKafkaHost" : "localhost",
    "httpKafkaPort" : 0,
    "consumerTopicName" : "inference-in",
    "consumerKeyDeserializerClass" : "io.vertx.kafka.client.serialization.JsonObjectDeserializer",
    "consumerValueDeserializerClass" : "io.vertx.kafka.client.serialization.JsonObjectDeserializer",
    "consumerGroupId" : "konduit-serving-consumer-group",
    "consumerAutoOffsetReset" : "earliest",
    "consumerAutoCommit" : "true",
    "producerTopicName" : "inference-out",
    "producerKeySerializerClass" : "io.vertx.kafka.client.serialization.JsonObjectSerializer",
    "producerValueSerializerClass" : "io.vertx.kafka.client.serialization.JsonObjectSerializer",
    "producerAcks" : "1"
  },
  "mqttConfiguration" : { },
  "customEndpoints" : [ ],
  "pipeline" : {
    "steps" : [ {
      "@type" : "IMAGE_TO_NDARRAY",
      "config" : {
        "height" : 28,
        "width" : 28,
        "dataType" : "FLOAT",
        "includeMinibatchDim" : true,
        "aspectRatioHandling" : "CENTER_CROP",
        "format" : "CHANNELS_LAST",
        "channelLayout" : "GRAYSCALE",
        "normalization" : {
          "type" : "SCALE"
        },
        "listHandling" : "NONE"
      },
      "keys" : [ "image" ],
      "outputNames" : [ "input_layer" ],
      "keepOtherValues" : true,
      "metadata" : false,
      "metadataKey" : "@ImageToNDArrayStepMetadata"
    }, {
      "@type" : "KERAS",
      "modelUri" : "/home/zulfadzli/KS2/konduit-serving-examples/java/target/classes/models/keras/mnist/keras-mnist.h5",
      "inputNames" : [ "input_layer" ],
      "outputNames" : [ "output_layer" ]
    } ]
  }
}
```

