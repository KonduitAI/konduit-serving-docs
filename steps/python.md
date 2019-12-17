---
description: >-
  You can integrate Python into a Konduit Serving instance by defining
  PythonConfig objects as steps to PythonPipelineStep.
---

# Python pipeline steps

Konduit Serving uses [JavaCPP Presets](https://github.com/bytedeco/javacpp-presets/) to execute Python scripts using the [CPython API](https://github.com/bytedeco/javacpp-presets/tree/master/cpython). This allows you to build custom Konduit Serving pipeline steps by writing Python scripts to be run within a Konduit Serving Java process. 

![](../.gitbook/assets/image%20%281%29.png)

## **`PythonConfig`**

Define the configuration for a `PythonStep`.

{% tabs %}
{% tab title="Python" %}
A `PythonConfig` takes on the following parameters:

* `python_path`: Optional. The search path for Python modules, as defined by [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path). See the [Python modules](python.md#pythonpath) section for details. 
* `python_code_path`: Optional. Specify the location of your Python script.
* `python_code`: Optional. A string that contains Python commands.
* `python_inputs`: A dictionary with input names as keys and corresponding value types as values. Value types should be specified as [one of the following strings](https://github.com/KonduitAI/konduit-serving/blob/71260366719840bcc2fd58698cebe471267de4bb/python/konduit/inference.py#L156-L162): `"INT"`, `"STR"`, `"FLOAT"`, `"BOOL"`, `"NDARRAY"`.
* `python_outputs`: A dictionary with output names as keys and corresponding value types as values. Values types should be specified as per `python_inputs`. 
{% endtab %}

{% tab title="Java" %}
In Java, we can define a `PythonConfig` object using the Lombok builder API. 

A `PythonConfig` takes on the following parameters:

* `pythonPath`: Optional. The search path for Python modules, as defined by [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path). See the [Python modules](python.md#pythonpath) section for details. 
* `pythonCodePath`: Optional. Specify the location of your Python script.
* `pythonCode`: Optional. Takes a String that contains Python commands.
* `pythonInput`: Specify the name of the input and the type of value, as defined by the `name()` method of a `PythonVariables.Type` enumeration, namely: `"INT"`, `"STR"`, `"FLOAT"`, `"BOOL"`, `"NDARRAY"`, `"LIST"`. ``
* `pythonOutput`: Specify the name of the output and the type of value as per `pythonInput`. 
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
NumPy array subclasses and some NumPy array data types are not supported. 

Unsupported NumPy array data types are as follows:`np.uint8`, `np.uint16`, `np.uint32`, `np.uint64`, `np.uintp`, `np.complex64`, `np.complex128`, `np.int8`, `np.int16`, `np.bool`, `np.byte`, `np.ubyte`, `np.ushort`, `np.uintc`, `np.uint`, `np.ulonglong`, `np.half`, `np.csingle`, `np.cdouble`, `np.clongdouble`. 

For output type NDARRAY, convert your output to a regular NumPy array and supported data type using `np.array()/np.ndarray()` and/or the `astype()` method. Also, ensure that the output is a NumPy **array** and not a NumPy scalar: see the documentation for [`np.isscalar`](https://docs.scipy.org/doc/numpy/reference/generated/numpy.isscalar.html) for details. 
{% endhint %}

####  **Example 1: Using Python script** 

{% tabs %}
{% tab title="Python" %}
```python
python_config = PythonConfig(
    python_code_path="scripts/loadimage.py", 
    python_inputs={"x": "STR"}, 
    python_outputs={"y": "STR"}
)
```
{% endtab %}

{% tab title="Java" %}
```java
String pythonCodePath = new ClassPathResource("scripts/loadimage.py") 
    .getFile() 
    .getAbsolutePath();

PythonConfig pythonConfig = PythonConfig.builder() 
    .pythonCodePath(pythonCodePath) 
    .pythonInput("x", PythonVariables.Type.STR.name()) 
    .pythonOutput("y", PythonVariables.Type.STR.name()) 
    .build();
```
{% endtab %}
{% endtabs %}

#### Example 2: Specifying a custom Python path

{% tabs %}
{% tab title="Python" %}
```java
python_config = PythonConfig(
    python_path=pythonPath,
    python_code="y = x + 2", 
    python_inputs={"x": "NDARRAY"}, 
    python_outputs={"y": "NDARRAY"}
)
```
{% endtab %}

{% tab title="Java" %}
```java
PythonConfig pythonConfig = PythonConfig.builder()        
    .pythonPath(pythonPath)        
    .pythonCode("y = x + 2")        
    .pythonInput("x", PythonVariables.Type.NDARRAY.name())        
    .pythonOutput("y", PythonVariables.Type.NDARRAY.name())        
    .build();
```
{% endtab %}
{% endtabs %}

## `PythonStep`

For most use cases,`PythonStep` can be set up as follows:

{% tabs %}
{% tab title="Python" %}
```java
python_step = PythonStep().step(python_config)
```

Note that by default, the default name for each step is `default`. You will need this when specifying your data inputs via the `.predict()` method of the `Client` class. 
{% endtab %}

{% tab title="Java" %}
The  `step` method of the`PythonStep` class is used to define the Python configuration. 

```java
PythonStep pythonStep = new PythonStep()
    .step(pythonConfig);
```
{% endtab %}
{% endtabs %}

Finally, the `PythonStep` object can be passed to an `InferenceConfiguration` object, which is used to configure the Konduit Serving instance:

{% tabs %}
{% tab title="Python" %}
```java
inference_config = InferenceConfiguration(
    serving_config=serving_config,
    pipeline_steps=[python_step]
)
```
{% endtab %}

{% tab title="Java" %}
```java
InferenceConfiguration config = InferenceConfiguration.builder()
        .pipelineStep(pythonStep)
        .build();
```

To test a `PythonStep` without starting a Konduit Serving instance, the output of the Python pipeline step can be retrieved as a `Writable[][]` object. Apply the`getRunner()` method to the `PythonStep` object, which gets the runner for the configuration, followed by the `transform()` method, which applies the transformations defined by the `PythonConfig` object. For example, 

```java
Writable[][] output = pythonStep.getRunner().transform(imagePath);
```
{% endtab %}
{% endtabs %}

Some models may require the server to transform more than one set of inputs. For instance, to serve object detection models, annotations and images may have to be transformed in a single `PythonStep`. This requires a unique name to be specified for each `PythonConfig`:

{% tabs %}
{% tab title="Python" %}
```java
python_config_1 = PythonConfig(
    python_path=pythonPath, 
    python_code="y = x + 2", 
    python_inputs={"x": "INT"}, 
    python_outputs={"y": "INT"}
)

python_config_2 = PythonConfig(
    python_path=pythonPath, 
    python_code="b = a + 3", 
    python_inputs={"a": "INT"}, 
    python_outputs={"b": "INT"}
)

python_step = (PythonStep()
    .step("stepOne", python_config_1)
    .step("stepTwo", python_config_2))
```
{% endtab %}

{% tab title="Java" %}
```java
PythonConfig pythonConfig1 = PythonConfig.builder()
        .pythonPath(pythonPath)
        .pythonCode("y = x + 2")
        .pythonInput("x", PythonVariables.Type.INT.name())
        .pythonOutput("y", PythonVariables.Type.INT.name())
        .build();

PythonConfig pythonConfig2 = PythonConfig.builder()
        .pythonPath(pythonPath)
        .pythonCode("b = a + 3")
        .pythonInput("a", PythonVariables.Type.INT.name())
        .pythonOutput("b", PythonVariables.Type.INT.name())
        .build();

PythonPipelineStep pythonPipelineStep = new PythonPipelineStep()
        .step("stepOne", pythonConfig1)
        .step("stepTwo", pythonConfig2);

Writable[][] output = pythonPipelineStep
        .getRunner()
        .transform(
                new Object[] {3}, 
                new Object[] {3}
        );

System.out.println(Arrays.deepToString(output));
```
{% endtab %}
{% endtabs %}

## YAML configuration

Python steps can take any argument that can be passed to `PythonConfig`.The following is a basic example of specifying a Python step in a YAML configuration: 

```yaml
steps: 
  python_step: 
    type: PYTHON
    python_code: simple.py
```

* `type`: specify this as PYTHON
* `python_code`: if you want to specify your Python code directly in your YAML file. The following [documentation](http://blogs.perl.org/users/tinita/2018/03/strings-in-yaml---to-quote-or-not-to-quote.html) may be helpful for specifying multi-line Python code, specifically the section on literal block scalars.
* `python_code_path`: specify the path of a Python `.py` script. 
* `python_inputs`: name-value pairs specifying the data types for each of the inputs referenced in the script 
* `python_outputs`: name-value pairs specifying the data types for each of the outputs referenced in the script
* `python_path`: location of the Python modules. Generally, if your script only requires NumPy, setting a custom `python_path` is not necessary. Refer to the [Python modules](https://serving.oss.konduit.ai/python#python-modules-and-the-pythonpath-argument) documentation on setting a custom Python path with additional modules. 

The names referenced in `python_inputs` and `python_outputs` correspond with `inputColumnNames` and `outputColumnNames`. Modifying `python_inputs` and `python_outputs` does not modify the input and output name of the step. `input_names` and `output_names` are arguments to `PythonStep` which cannot be accessed through the YAML configuration, and default to the name `default`.

## Python modules and the `pythonPath` argument 

If the `pythonPath` is not specified, you will still be able to import modules cached for NumPy by [JavaCPP Presets](https://github.com/bytedeco/javacpp-presets/) in your Python script\(s\). 

In Java, you can find the location of the default modules by printing 

```java
Arrays.toString(cachePackages())
```

where `cachePackages` is imported as a static variable: 

```java
import static org.bytedeco.numpy.presets.numpy.cachePackages;
```

If you require additional modules, you can set a custom`pythonPath` by running the following command in your Terminal and setting the output as your `pythonPath`:

{% tabs %}
{% tab title="Windows" %}
```java
:: note: if you use Python via Anaconda, 
:: you may have to run this code in Anaconda Prompt
python -c "import sys; print(';'.join([path.strip() for path in sys.path if path.strip()]))"
```
{% endtab %}

{% tab title="Linux, macOS" %}
```bash
python -c "import sys; print(':'.join([path.strip() for path in sys.path if path.strip()]))"
```
{% endtab %}
{% endtabs %}

Custom `pythonPath` follows the format defined by `sys.path.`The first element is the location of the script used to invoke the Python interpreter, and the remaining elements specify where Python should search for modules. 

{% hint style="info" %}
On Windows, the semicolon \(;\) separator is used instead of the colon \(:\) separator used on Linux and macOS.
{% endhint %}

{% hint style="info" %}
To list the modules that you can access, run `help("modules")` in your Python interpreter.
{% endhint %}

