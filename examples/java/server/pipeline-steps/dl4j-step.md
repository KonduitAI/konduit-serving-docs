---
description: Example of applying DL4J Step
---

# DL4J Step

This example splits  into two parts which are configuring the inference configuration and running the server. 

```java
import ai.konduit.serving.examples.utils.Train;
import ai.konduit.serving.models.deeplearning4j.step.DL4JStep;
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
import java.util.Arrays;
```

{% hint style="info" %}
A reference Java project is provided in the Example repository from [https://github.com/KonduitAI/konduit-serving-examples](https://github.com/KonduitAI/konduit-serving-examples) with a Maven pom.xml dependencies file. If using the IntelliJ IDEA IDE, open the java folder as a Maven project and run the main function of Example\_1\_Dl4jStep class.
{% endhint %}

### Configure the step

Let's start from the main function by getting the trained model.

```java
//get the file of trained model
Train.ModelTrainResult modelTrainResult = Train.dl4jIrisModel();
```

Create an inference configuration by default.

```java
//a default Inference Configuration
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();
```

We'll need to include `DL4JStep` into the pipeline and bind with the inference configuration. Specify the following:

* `modelUri` : the model file path
* `inputNames` : names for model's input layer
* `outputNames` : names for model's output layer

```java
//include pipeline step into the Inference Configuration
inferenceConfiguration.pipeline(SequencePipeline.builder()
        .add(new DL4JStep() //add DL4JStep into pipeline
                .modelUri(modelTrainResult.modelPath())
                .inputNames(modelTrainResult.inputNames())
                .outputNames(modelTrainResult.outputNames())
        ).build()
);
```

### Deploy the server

Let's deploy the model in the server by calling `DeployKonduitServing` with the configuration made before. The handler, a callback function, is implemented to capture a successful or failed server deployment state.

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
                            .body(new JSONObject().put("layer0",
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
The server is running on port 39615 with deployment id of 2fe69a2d-1276-4a1d-b5af-50191640019f
Result from server : {
  "layer2" : [ [ 5.287693E-4, 0.02540398, 0.9740673 ] ]
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
  - '@type': "DEEPLEARNING4J"
    modelUri: "/tmp/model6027458615639981189zip"
    inputNames:
    - "layer0"
    outputNames:
    - "layer2"
```

