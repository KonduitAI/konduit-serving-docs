---
description: Example of applying ONNX Step
---

# ONNX Step

This page provides a Java example of deploying a built-in model Python with Open Neural Network Exchange \(ONNX\) platform. The ONNX format is supported by other deep learning frameworks such Tensorflow, Pytorch, etc. In this example, the ONNX model is used to deploy the Iris model in the server.

```java
import ai.konduit.serving.examples.utils.Train;
import ai.konduit.serving.models.onnx.step.ONNXStep;
import ai.konduit.serving.pipeline.impl.pipeline.SequencePipeline;
import ai.konduit.serving.vertx.api.DeployKonduitServing;
import ai.konduit.serving.vertx.config.InferenceConfiguration;
import ai.konduit.serving.vertx.config.InferenceDeploymentResult;
import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;
import io.vertx.core.DeploymentOptions;
import io.vertx.core.VertxOptions;
import org.json.JSONArray;
import org.json.JSONObject;
import java.io.IOException;
import java.util.Arrays;
```

### Configure the step

Let's start from the main function by getting the trained model.

```java
//get the file of trained model
Train.ModelTrainResult modelTrainResult = Train.onnxIrisModel();
```

Create an inference configuration by default.

```java
//a default Inference Configuration
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();
```

We'll need to include `ONNXStep` into the pipeline and specify the following:

* `modelUri` : the model file path
* `inputNames` : names for model's input layer
* `outputNames` : names for model's output layer

```java
//include pipeline step into the Inference Configuration
inferenceConfiguration.pipeline(SequencePipeline.builder()
        .add(new ONNXStep() //add ONNXStep into pipeline
                .modelUri(modelTrainResult.modelPath())
                .inputNames(modelTrainResult.inputNames())
                .outputNames(modelTrainResult.outputNames())
        ).build()
);
```

### Deploy the server

Let's deploy the model in the server by calling  `DeployKonduitServing` with the configuration made before. A callback function is used to respond only after a successful or failed server deployment inside the handler block.

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
                    String result = Unirest.post(String.format("http://localhost:%s/predict", runnningPort))
                            .header("Content-Type", "application/json")
                            .header("Accept", "application/json")
                            .body(new JSONObject().put("input",
                                    new JSONArray().put(Arrays.asList(1.0, 1.0, 1.0, 1.0)))
                            )
                            .asString().getBody();

                    System.out.format("Result from server : %s%n", result);

                    System.exit(0);
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

Note that we consider only one test input array in this example for inference to show the model's deployment in Konduit-Serving. After execution, the successful server deployment gives below output text.

```aspnet
The server is running on port 44301 with deployment id of 775bfbd3-2d18-435b-86c6-e9fbe7303cad
Result from server : {
  "output" : [ [ 0.035723433, 0.27029678, 0.69397974 ] ]
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
  - '@type': "ONNX"
    modelUri: "/home/zulfadzli/KS2/konduit-serving-examples/java/target/classes/models/onnx/iris/iris.onnx"
    inputNames:
    - "input"
    outputNames:
    - "output"
```

