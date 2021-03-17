# CLI Commands

This is the full command reference documentation for the Konduit-Serving CLI. See [local command](../../examples/cli/commands/) for some common usage examples.

```text
$ konduit [COMMAND] [OPTIONS] [arg...]
```

**COMMANDS**

| Commands | Description |
| :--- | :--- |
| [build](build-command.md) | Command line interface for performing Konduit-Serving builds. |
| [config](config-command.md) | A helper command for creating boiler plate JSON/YAML for inference configuration. |
| [inspect](inspect-command.md) | Inspect the details of a particular Konduit server. |
| list | Lists the running Konduit servers. List all Konduit servers launched through the `serve` command. This command does not need any arguments to use with `konduit` CLI. |
| [logs](logs-command.md) | View the logs of a particular Konduit server. |
| metrics | Shows the running metrics for a particular server. Prints the calculate metrics for a particular server. Useful for getting a quick insight into the running server. For example, shows metrics of a server, named 'id\_server',`konduit metrics id_server`. |
| [predict](predict-command.md) | Run inference on Konduit servers using given inputs. |
| [profile](profile-command.md) | Command to list, view, edit, create and delete Konduit-Serving run profiles. |
| [pythonpaths](pythonpaths-command.md) | A utility command to manage system installed and manually registered python binaries. |
| [serve](serve-command.md) | Start a Konduit server application. |
| stop | Stop a running Konduit server. This command stops a Konduit server started with the `serve` command. The command requires the serving id as an argument. For example, stops a server named 'id\_server',`konduit stop id_server`. |
| version | Displays Konduit-Serving version by printing the Konduit-Serving version used by the application along with other build details. For example: `konduit --version`. |

