# Profile Command

```text
$ konduit profile [-ad <additional_dependencies>] [-at <append_type>] [-a
       <cpu_architecture>]  [-d <device>] [-en <env_name>] [-o
       <operating_system>] [-pp <python_path>] [-pt <python_type>] [-st
       <server_types>]  [profile_name] [sub_command]
```

**OPTIONS**

| Command Flags | Description |
| :--- | :--- |
| -ad,--addDep | One or more space separated values \(maven coordinates\) indicating additional dependencies to be included with the server launch. The pattern of additional dependencies should be either &lt;group\_id&gt;:&lt;artifact\_id&gt;:&lt;version&gt; or &lt;group\_id&gt;:&lt;artifact\_id&gt;:&lt;version&gt;:&lt;classifier&gt;. |
| -at,--append\_type | Override property for python config for specifying append type with javacpp cpython library paths. Available values are: \[`BEFORE`, `NONE`, `AFTER`\].  |
| -a,--arch | Name of the cpu architecture. Accepted values are: \[`x86`, `x86_64`, `x86_avx2`, `x86_64-avx2`, `x86_64_avx2`, `x86_avx512`, `x86_64-avx512`, `x86_64_avx512`, `armhf`, `arm64`, `ppc64le`\].  |
| -d,--device | Compute device to use with the server. Accepted values are: \[`CPU`, `CUDA_10.0`, `CUDA_10.1`, `CUDA_10.2`\]. |
| -en,--env\_name | Override property for python config for selecting environment name for python type `CONDA`. Ignored for python type \[`CUSTOM`, `JAVACPP`, `VENV`, `PYTHON`\]. |
| -o,--os | Operating system the server needs to run at. Accepted values are: \[`windows`, `linux`, `mac`\]. Defaults to the current OS value. |
| -pp,--python\_path | Override property for python config for selecting specifying python path id for python type \[`PYTHON`, `CONDA`, `VENV`\] and absolute path for python type `CUSTOM`. Ignored for python type `JAVACPP`. |
| -pt,--python\_type | Override property for python config for selecting python install type. Available values are: \[`JAVACPP`, `CUSTOM`, `PYTHON`, `CONDA`, `VENV`\]. |
| -st,--server\_types | One or more space separated values, indicating the backend server type. Accepted values are: \[`HTTP`, `GRPC`, `MQTT`\]. |

**ARGUMENTS**

| Argument Position | Name | Description |
| :--- | :--- | :--- |
| 1 | profile\_name | Name of the profile to create, view, edit or delete. |
| 2 | sub\_command | Sub command to be used with the profile command. Sub commands are: \[`default`, `create`, `list`, `view`, `edit`, `delete`\]. Defaults is: `list`. |

