# CLASSIFIER\_OUTPUT

`ClassifierOutputStep` takes as input a numerical 2d NDArray \(i.e., float/double etc. type\) with shape \[minibatch, numClasses\] which represents the softmax predictions for a standard classifier and returns based on this array:

1. The predicted class label - as a String
2. The predicted class index - as an integer \(long\)
3. The predicted class probability - as a Double

| Configs | Descriptions |
| :--- | :--- |
| inputNames | Optional. If set: this represents the NDArray. If not set: use DataUtils.inferField to find an NDArray field. Default is `null`. |
| returnLabel | Default is true; if false, don't return label. |
| returnIndex | Default is true; if false, don't return index. |
| returnProb | Default is true; if false, don't return probability. |
| labelName | Output names for the labels. Default value is `label`. |
| indexName | Output names for the index. Default value is `index`. |
| probName | Output names for the labels probabilities. Default value is `prob`. |
| labels | As a List&lt;String&gt;. Optional. If not specified, the predicted class index as a string is used - i.e., `\"0\"`, `\"1\"`, etc. Default value is `null`. |
| topN | Integer, `null` by default. If non-null and &gt; 1, we return `List<String>`, `List<Long>`, `List<Double>` for the predicted class/index/probability instead of String/Long/Double. |
| allProbabilities | If `true`, also returns a `List<List<Double>>` of all probabilities \(basically, convert NDArray to list\). `false` by default. |

