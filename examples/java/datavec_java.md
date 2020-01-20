---
description: >-
  Konduit Serving supports data transformations defined by the DataVec
  vectorization and ETL library.
---

# DataVec

```java
import ai.konduit.serving.configprovider.KonduitServingMain;
import ai.konduit.serving.pipeline.step.TransformProcessStep;
import org.datavec.api.transform.TransformProcess;
import ai.konduit.serving.config.SchemaType;
import ai.konduit.serving.config.ServingConfig;
import org.apache.commons.io.FileUtils;
```
DataVec transformations can be defined in TransformProcess class of DataVec, which can be which can be accessed through DataVec api from Java Language Obect extends:

```text
import org.datavec.api.transform.TransformProcess;
```


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
* String operations: `appendStringColumnTransform()`, `toLowerCase()`, `toUpperCase()`, `concat()`, `stringRemoveWhitespaceTransform()`, `replace_empty_string()`, 
  `replaceStringTransform()`, `stringMapTransform()`
* Column selection/renaming: `removeColumns()`, `removeAllColumnsExceptFor()`, `renameColumn()`
* One-hot encoding: `categoricalToOneHot()`, `integerToOneHot()`

In this short example, we append the string `two` to the end of values in the string column `first`.

```java
 Schema inputDataSchema = new Schema.Builder()
	.addColumnString("first")
	.build();
	
TransformProcess transformProcess = new TransformProcess.Builder(inputSchema).
    appendStringColumnTransform("first", "two").build();
```


## Configure the step

The `TransformProcess` can now be defined in the Konduit Serving configuration with a `TransformProcessStep`. Here, we

* **configure the inputs and outputs**: the schema, column names and data types should be defined here. 
* **declare the `TransformProcess`** using the `.transformProcess()` method. 

Note that `Schema` data types are not defined in the same way as `JavaStep` data types. See the [source](https://github.com/KonduitAI/konduit-serving/blob/78851701004ebb3dbf079889d46b79a9db8fac60/konduit-serving-api/src/main/java/ai/konduit/serving/util/SchemaTypeUtils.java#L154-L195) for a complete list of supported Schema data types:

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
String column_names[] = new String[5];
column_names[0] = "first";

SchemaType types[] = new SchemaType[5];
types[0] = SchemaType.String;

TransformProcessStep transformProcessStep = new TransformProcessStep()
	.setInput(inputSchema.toString(), column_names, types)
	.setOutput(inputSchema.toString(), column_names, types)
	.transformProcess(transformProcess);
	
List tpS    tepList = new ArrayList();
    tpStepList.add(transformProcessStep);
```

## Configure the server

Configure the Server using `ServingConfig` to define the port using the `httpPort` argument and data formats using the `inputDataFormat` and `outputDataFormat` arguments.

```java
int port = Util.randInt(1000, 65535);

ServingConfig servingConfig = ServingConfig.builder().httpPort(port).	
	build();
	
InferenceConfiguration inferenceConfiguration = InferenceConfiguration.builder()
	.step(tpStepList).servingConfig(servingConfig).build();
```
The complete configuration is as follows:

```java
System.out.println(inferenceConfiguration.toJson());
```

```text
{'@type': 'InferenceConfiguration',
 'steps': [{'@type': 'TransformProcessStep',
   'inputSchemas': {'default': ['String']},
   'outputSchemas': {'default': ['String']},
   'inputNames': ['default'],
   'outputNames': ['default'],
   'inputColumnNames': {'default': ['first']},
   'outputColumnNames': {'default': ['first']},
   'transformProcesses': {'default': {'actionList': [{'transform': {'@class': 'org.datavec.api.transform.transform.string.AppendStringColumnTransform',
        'columnName': 'first',
        'toAppend': 'two'}}],
     'initialSchema': {'@class': 'org.datavec.api.transform.schema.Schema',
      'columns': [{'@class': 'org.datavec.api.transform.metadata.StringMetaData',
        'name': 'first'}]}}}}],
 'servingConfig': {'@type': 'ServingConfig',
  'httpPort': 47964,
  'inputDataFormat': 'JSON',
  'outputDataFormat': 'JSON',
  'logTimings': True}}
```

## Start the server 

```java
KonduitServingMain.main("--configPath", configFile.getAbsolutePath());
```

```text
Starting server..

Server has started successfully.
```

## Inference

The `Client` should be configured to match the Konduit Serving instance. As this example is run on a local computer, the server is located at host `'http://localhost'` and port `port`.
And Finally, we run the Konduit Serving instance. Recall that the `TransformProcessStep()` appends a string `two` to strings in the column `first`:

```java
HashMap<String, String> data_input = new HashMap<>();
data_input.put("first", "value");

String response = Unirest.post("http://localhost:3000/raw/String")
	.field("input", data_input)
	.asString().getBody();

System.out.print(response);
```
```text
{'first': 'valuetwo'}
```