# LOGGING

`LoggingStep` is a pipeline step that simply logs the input data keys \(and optionally values\) and returns the input data unchanged.

| Configs | Descriptions |
| :--- | :--- |
| logLevel | This is similar to how standard logging frameworks define logging categories. Default value is `INFO`. Another available options are `TRACE`, `DEBUG`, `WARN` and `ERROR`. |
| log | An enum specifying what part of a data instance should be logged. Default value is `KEYS`. |
| keyFilterRegex | A regular expression that allows filtering of keys - i.e., only those that match the regex will be logged. |



