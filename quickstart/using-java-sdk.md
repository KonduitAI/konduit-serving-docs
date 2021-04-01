---
description: Guide to start using Konduit-Serving with Java SDK
---

# Using Java SDK

This quickstart article will show you to begin your project in the Java environment. Konduit-Serving provides Java SDK, a developer tool that enable you to write the code with more ease, effectiveness and efficiency. Let's start building and installing Konduit-Serving from the source and take a look to demonstrate the examples of Konduit-Serving in Java.

### Prerequisites

You will need following prerequisites to follow along

* Maven 3.x
* JDK 8
* Git
* [IntelliJ](https://www.jetbrains.com/idea/)

## Installation from Sources

The following section explains how to clone and build Konduit-Serving from sources. To build from source, follow the guide below:

{% page-ref page="../building-from-source.md" %}

Once you've installed the Konduit-Serving to your local maven repository, you can now include it in your build tool's dependencies. Follow the instructions below for an example of Konduit-Serving with Java SDK.

## Java SDK

Let's look at the examples how to use Konduit-Serving in Java

### Cloning Examlpe Repository

Let's clone the `konduit-serving-examples` repository:

```text
$ git clone https://github.com/KonduitAI/konduit-serving-examples
```

You'll see the following files in `konduit-serving-examples`:

```text
konduit-serving-examples/
├── data
├── java
├── monitoring
├── notebooks
├── python
├── quickstart
├── README.md
├── utils
└── yaml
```

You'll need to open the "java" file in IntelliJ as a project, and you'll find two examples under the `./src/main/java` subfolder. Let's look at the examples of Konduit-Serving to create a configuration and deploy a server.

### Example

Let’s start by defining the inference configuration and sequence pipeline that define the configuration of the server:

```java
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();

SequencePipeline sequencePipeline = SequencePipeline
        .builder()
        .add(new LoggingStep().log(LoggingStep.Log.KEYS_AND_VALUES))
        .build();
```

The inference configuration should have a pipeline to deploy the server, include pipeline with:

```java
inferenceConfiguration.pipeline(sequencePipeline);

System.out.format(inferenceConfiguration.toYaml());
```

You'll see the printed server configuration as a YAML:

```yaml
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
  - '@type': "LOGGING"
    logLevel: "INFO"
    log: "KEYS_AND_VALUES"
```

Let's deploy the server by using `DeplotKonduitServing.deploy()` includes created inference configuration with the pipeline into the deployment:

```java
DeployKonduitServing.deploy(
        new VertxOptions(),
        new DeploymentOptions(),
        inferenceConfiguration,
        handler -> {
            if (handler.succeeded()) {
                InferenceDeploymentResult inferenceDeploymentResult = handler.result();
                int runningPort = inferenceDeploymentResult.getActualPort();
                String deploymentId = inferenceDeploymentResult.getDeploymentId();

                System.out.format("%nPort is %s and Deployment Id is %s.%n", runningPort, deploymentId);

                try {
                    String result = Unirest.post(String.format("http://localhost:%s/predict", runningPort))
                            .header("Content-Type", "application/json")
                            .header("Accept", "application/json")
                            .body(new JSONObject().put("input_key", "input_value"))
                            .asString().getBody();

                    System.out.format("Result: %s%n", result);

                    System.exit(0);
                } catch (UnirestException e) {
                    e.printStackTrace();
                    System.exit(1);
                }
            } else {
                System.out.println(handler.cause().getMessage());
                System.exit(1);
            }
        }

);
```

The handler expression is a callback after a successful or failed server deployment which can be implemented inside the handler block, as shown: 

```aspnet
Port is 38019 and Deployment Id is cc5e2081-81e4-4d3e-9734-3201af512641.
Result: {
  "input_key" : "input_value"
}

Process finished with exit code 0
```

Congratulation! You've deployed Konduit-Serving using Java SDK. 

