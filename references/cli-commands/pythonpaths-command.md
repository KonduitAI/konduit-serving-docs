# Pythonpaths Command

```text
$ konduit pythonpaths  [-p <install_path>]  -t <type> [-wip] [sub_command]
```

**OPTIONS**

| Command Flags | Description |
| :--- | :--- |
| -p,--path | Absolute path of the python installation. For `conda` and `venv` types this refers to the absolute path of the root installation folder. |
| -t,--type | Name of the python type. For the `add` subcommand, accepted values are: \[`python`, `conda`, `venv`\]. For the `list` subcommand, accepted values are: \[`all`, `javacpp`, `python`, `conda`, `venv`\]. For `config` subcommand the accepted values are: \[`custom`, `javacpp`, `python`, `conda`, `venv`\]. |
| -wip,--with-installed-packages | Absolute path of the python installation. For `conda` and `venv` types this refers to the absolute path of the root installation folder. |

**ARGUMENT**

| Argument Position | Name | Description |
| :--- | :--- | :--- |
| 1 | sub\_command | Sub command to be used with the `pythonpaths` command. Sub commands are: \[`add`, `list`, `config`\]. Defaults is: `list`. |

