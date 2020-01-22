---
Description: >-
  Konduit Serving supports data transformations defined by the DataVec
  vectorization and ETL library.
---

# DataVec

```java
import ai.konduit.serving.InferenceConfiguration;
import ai.konduit.serving.config.ServingConfig;
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.pipeline.step.TransformProcessStep;
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.JsonNode;
import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;
import org.apache.commons.io.FileUtils;
import org.datavec.api.transform.TransformProcess;
import org.datavec.api.transform.schema.Schema;
```

{% hint style="info" %}
A reference Java project is provided in the Example repository \( https://github.com/KonduitAI/konduit-serving-examples \) with a Maven pom.xml dependencies file. If using the IntelliJ IDEA IDE, open the java folder as a Maven project and run the main function of InferenceModelStepDataVec the class.
{% endhint %}

## Data transformations with DataVec

### **Schema \(**[**source**](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/transform/schema/Schema.java)**\)**

A `Schema` specifies the structure of your data. In DataVec, a `TransformProcess` requires the `Schema` of the data to be specified.
Both schema and transform process classes come with a helper Builder class which are useful for organizing code and avoiding complex constructors.

`Schema` objects have a number of methods that define different data types for columns: `addColumnsString()`, `addColumnInteger()`, `addColumnLong()`, `addColumnFloat()`, `addColumnDouble()` and `addColumnCategorical()`.

### **TransformProcess \(**[**source**](https://github.com/deeplearning4j/DataVec/blob/master/datavec-api/src/main/java/org/datavec/api/transform/TransformProcess.java)**\)**

`TransformProcess` provides a number of methods to manipulate your data. The following methods are available in the Datavec API:

* Reduce the number of rows: `filter()`
* General data transformations: `replaceStringTransform()`, `replaceMapTransform()`,
* Type casting: `stringToTimeTransform()`, `transform()`, `categoricalToInteger()`,
* Combining/reducing the values in each column: `reduce()`
* String operations: `appendStringColumnTransform()`, `toLowerCase()`, `toUpperCase()`, `stringRemoveWhitespaceTransform()`, `replaceStringTransform()`, `stringMapTransform()`
* Column selection/renaming: `removeColumns()`, `removeAllColumnsExceptFor()`, `renameColumn()`
* One-hot encoding: `categoricalToOneHot()`, `integerToOneHot()`

In this short example, we append the string `two` to the end of values in the string column `first`. As an initial step, define the input and output Schema with string column:

```java
Schema inputSchema = new Schema.Builder()
    .addColumnString("first")
    .build();

Schema outputSchema = new Schema.Builder()
    .addColumnString("first")
    .build();

TransformProcess transformProcess = new TransformProcess.Builder(inputSchema).
    appendStringColumnTransform("first", "two").build();
```


## Configure the step

The `TransformProcess` can now be defined in the Konduit Serving configuration with a `TransformProcessStep`. Here, we

* **configure the inputs and outputs**: the schema, column names and data types should be defined here.
* **declare the `TransformProcess`** using the `.transformProcess()` method.

Note that `Schema` data types are not defined in the same way as `PythonStep` data types. See the [source](https://github.com/KonduitAI/konduit-serving/blob/78851701004ebb3dbf079889d46b79a9db8fac60/konduit-serving-api/src/main/java/ai/konduit/serving/util/SchemaTypeUtils.java#L154-L195) for a complete list of supported Schema data types:

* `NDArray`
* `String`
* `Boolean`
* `Categorical`
* `Float`
* `Double`
* `Integer`
* `Long`
* `Bytes`

You should define the Schema data types in `TransformProcessStep()` as strings.

```java
TransformProcessStep transformProcessStep = new TransformProcessStep(transformProcess, outputSchema);
```

## Configure the server

Configure the Server using `ServingConfig` to define the port using the `httpPort` argument.

```java
int port = Util.randInt(1000, 65535);

ServingConfig servingConfig = ServingConfig.builder()
    .httpPort(port)
    .build();

InferenceConfiguration inferenceConfiguration = InferenceConfiguration.builder()
    .step(transformProcessStep).servingConfig(servingConfig).build();
```

The `inferenceConfiguration` is stored as a JSON File.

```java
File configFile = new File("config.json");
FileUtils.write(configFile, inferenceConfiguration.toJson(), Charset.defaultCharset());
```

## Inference

The `Client` should be configured to match the Konduit Serving instance. As this example is run on a local computer, the server is located at host `'http://localhost'` and port `port`.
And Finally, we run the Konduit Serving instance with the saved **config.json** file path as `configPath` and other necessary server configuration arguments.. Recall that the `TransformProcessStep()` appends a string `two` to strings in the column `first`.

A Callback Function onSuccess is implemented in order to post the Client request and get the HttpResponse, only after the successful run of the KonduitServingMain Server.

```java
KonduitServingMain.builder()
    .onSuccess(() -> {
        try {
            HttpResponse<JsonNode> response = Unirest.post(String.format("http://localhost:%s/raw/json", port))
                    .header("Content-Type", "application/json")
                    .body("{\"first\" :\"value\"}").asJson();

            System.out.println(response.getBody().toString());
            System.exit(0);
        } catch (UnirestException e) {
            e.printStackTrace();
            System.exit(0);
        }
    })
    .build()
    .runMain("--configPath", configFile.getAbsolutePath());
```

## Confirm the output

After executing the above, in order to confirm the successful start of the Server, check for the below output text:

```text
Jan 08, 2020 1:36:01 PM ai.konduit.serving.configprovider.KonduitServingMain
INFO: Deployed verticle ai.konduit.serving.verticles.inference.InferenceVerticle
```

The Output of the program is as follows:

```java
System.out.println(response.getBody().toString());
```

```text
{"first":"valuetwo"}
```
The complete inference configuration in JSON format is as follows:

```java
System.out.println(inferenceConfiguration.toJson());
```

```text
SLF4J: Actual binding is of type [ch.qos.logback.classic.util.ContextSelectorStaticBinder]
{
  "memMapConfig" : null,
  "servingConfig" : {
    "httpPort" : 15614,
    "listenHost" : "localhost",
    "logTimings" : false,
    "metricTypes" : [ "CLASS_LOADER", "JVM_MEMORY", "JVM_GC", "PROCESSOR", "JVM_THREAD", "LOGGING_METRICS", "NATIVE" ],
    "outputDataFormat" : "JSON",
    "uploadsDirectory" : "file-uploads/"
  },
  "steps" : [ {
    "@type" : "TransformProcessStep",
    "inputColumnNames" : {
      "default" : [ "first" ]
    },
    "inputNames" : [ "default" ],
    "inputSchemas" : {
      "default" : [ "String" ]
    },
    "outputColumnNames" : {
      "default" : [ "first" ]
    },
    "outputNames" : [ "default" ],
    "outputSchemas" : {
      "default" : [ "String" ]
    },
    "transformProcesses" : {
      "default" : {
        "actionList" : [ {
          "transform" : {
            "@class" : "org.datavec.api.transform.transform.string.AppendStringColumnTransform",
            "columnName" : "first",
            "toAppend" : "two"
          }
        } ],
        "initialSchema" : {
          "@class" : "org.datavec.api.transform.schema.Schema",
          "columns" : [ {
            "@class" : "org.datavec.api.transform.metadata.StringMetaData",
            "name" : "first"
          } ]
        }
      }
    }
  } ]
}
```
