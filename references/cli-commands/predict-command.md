# Predict Command

```text
$ konduit predict  [-d] [-it <value>] [-ot <value>] [-p <value>] [-r
       <value>]  server-id data
```

**OPTIONS**

| Command Flags | Description |
| :--- | :--- |
| -d,--debug | Debug with additional output. |
| -it,--input-type | Input type. Choices are: \[`json`, `json-file`, `binary`, `binary-file`, `multipart`\]. Default is: `json`. |
| -ot,--output-type | Output type. Choices are: \[`json`, `binary`\]. Default is: `json`. |
| -p,--protocol | Server Protocol. Choices are: \[`HTTP`, `GRPC`, `MQTT`\]. Default is: `HTTP`. |
| -r,--repeat | Repeat requests the given number of times. |

 **ARGUMENTS**

| Argument Position | Name | Description |
| :--- | :--- | :--- |
| 1 | server\_id | Konduit server id  Data to send to the server. |
| 2 | data | Accepts JSON/Binary data string as the input. |

