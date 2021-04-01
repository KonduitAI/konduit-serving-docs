---
description: Simple example to deploy a server with Konduit-Serving
---

# Server

In this example, we'll deploy a server with Konduit-Serving. 

* First, let's start with creating a complete configuration of the server.

```java
InferenceConfiguration inferenceConfiguration = new InferenceConfiguration();
inferenceConfiguration.pipeline(
        SequencePipeline
                .builder()
                .add(new LoggingStep().log(LoggingStep.Log.KEYS_AND_VALUES))
                .build()
);
```

* Let's deploy the server with the configuration made above. The successful server deployment will give the port number and host of the server:

```java
DeployKonduitServing.deploy(
                new VertxOptions(), // Default vertx options
                new DeploymentOptions(), // Default deployment options
                inferenceConfiguration, // Inference configuration with logging step
                handler -> { // this block will be called when server finishes the deployment
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
                                    .body(new JSONObject().put("input_key", "input_value"))
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

You'll be able to see the output similar to this once the server successfully deployed:

```aspnet
The server is running on port 37663 with deployment id of 59d5d475-be83-4348-8983-4d3e7328e71d
Result from server : {
  "input_key" : "input_value"
}

Process finished with exit code 0
```

