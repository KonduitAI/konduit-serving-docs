# KERAS

`KerasStep` is a pipeline step that configures a Keras model that is to be executed.

| Configs | Descriptions |
| :--- | :--- |
| modelUri | Specifies the location of a saved model file. |
| inputNames | A list of names of the input placeholders \(computation graph, with multiple inputs. Where values from the input data keys are mapped to the computation graph inputs\). |
| outputNames | A list of names of the output placeholders \(computation graph, with multiple outputs. Where the values of these output keys are mapped from the computation graph output - `INDArray[]` to data keys\). |



