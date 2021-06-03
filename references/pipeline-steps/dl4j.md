# DEEPLEARNINGL4J

`DL4JStep` is a pipeline step that configures a DL4J model that is to be executed.

| Configs | Descriptions |
| :--- | :--- |
| modelUri | Specifies the location of a saved model file. |
| inputNames | A list of names of the input placeholders \(mainly for DL4J - computation graph, with multiple inputs. Where values from the input data keys are mapped to the computation graph inputs\). |
| outputNames | A list of names of the output placeholders \(mainly for DL4J - computation graph, with multiple outputs. Where the values of these output keys are mapped from the computation graph output - `INDArray[]` to data keys\). |
| loaderClass | Optional, usually unnecessary. Specifies a class used to load the model if customization in how model loading is performed, instead of the usual `MultiLayerNetwork.load` or `ComputationGraph.load` methods. Must be a `java.util.Function<String,MultiLayerNetwork>` or `java.util.Function<String,ComputationGraph>` |



