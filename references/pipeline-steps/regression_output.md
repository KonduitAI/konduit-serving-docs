# REGRESSION\_OUTPUT

`ResgressionOutputStep` is a adapter - extracts values from an NDArray to double values in the output Data instance, with names as specified. For example, `input=Data{"myArray"=<ndarray>}`, `output=Data{"x"=ndarray[0], "y"=ndarray[7]}`.

| Configs | Descriptions |
| :--- | :--- |
| inputName | Optional. If set: this represents the NDArray. If not set: use `DataUtils.inferField` to find an NDArray field. |
| names | `Map<String,Integer>` where the key is the output name, and the value is the index in the array. Default value is `null`. |

