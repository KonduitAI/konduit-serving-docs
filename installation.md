# Installation

[![PyPI version](https://badge.fury.io/py/konduit.svg)](https://badge.fury.io/py/konduit)

Install `konduit` from PyPI with 

```bash
pip install konduit
```

{% hint style="warning" %}
A version of Konduit Serving with the command line interface \(CLI\) is not currently available on PyPI. To obtain the CLI, clone the [konduit-serving](https://github.com/KonduitAI/konduit-serving) repository, and in the `python` folder run `python setup.py install`. 
{% endhint %}

You may need to install Cython before installing Konduit with 

```text
pip install cython 
```

We recommend using Python 3.7+.

## Building the Konduit JAR 

{% hint style="info" %}
Building the Konduit JAR requires Maven and JDK 8. 
{% endhint %}

### Manual build 

First, clone the [konduit-serving repository](https://github.com/KonduitAI/konduit-serving): 

```text
git clone https://github.com/KonduitAI/konduit-serving.git
```

Then, run the following commands in the root directory of konduit-serving: 

```text
mvn -N io.takari:maven:0.7.6:wrapper
python build_jar.py --os <your-platform>
```

where `<your-platform>` is picked from `windows-x86_64`,`linux-x86_64`,`linux-x86_64-gpu`, `macosx-x86_64`, `linux-armhf` and `windows-x86_64-gpu`, depending on your operating system and architecture. 

### Building with the command line interface

Once the konduit package is installed, you have access to a command line interface \(CLI\) tool called `konduit`. 

The `init` command:

1. gets the latest Konduit Serving code, 
2. builds the Java dependencies needed for`konduit`, then 
3. exports the location of the Konduit JAR as an environment variable. 

It assumes that you have `git` installed on your system and that `python3` is available. 

Run:

```bash
konduit init --os <your-platform>
```

where `<your-platform>` is picked from `windows-x86_64`,`linux-x86_64`,`linux-x86_64-gpu`, `macosx-x86_64`, `linux-armhf` and `windows-x86_64-gpu`, depending on your operating system and architecture. 

To rebuild the Konduit Serving JAR without adding the `KONDUIT_JAR_PATH` environment variable, run `konduit build` instead with the appropriate flags. 

{% hint style="info" %}
### Known issues

* `konduit init` fails for  `linux-86_64-gpu` \([\#115](https://github.com/KonduitAI/konduit-serving/issues/115)\)
{% endhint %}

### Set environment variables manually

You can set a default location for the Konduit Serving JAR using environment variables. 

{% tabs %}
{% tab title="Windows" %}
Use [setx.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/setx): 

```bash
setx KONDUIT_JAR_PATH "C:\Users\User\konduit-serving\konduit.jar"
```
{% endtab %}

{% tab title="Linux, macOS" %}
```bash
export KONDUIT_JAR_PATH="~/konduit-serving/konduit.jar"
```
{% endtab %}
{% endtabs %}

## Command line interface

You can test the `konduit` command line tool by typing the following into your command line:

```text
konduit --help
```

which should prompt all currently available commands:

```text
Usage: konduit [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  build          Build the underlying konduit.jar (again).
  init           Initialize the konduit-python CLI.
  predict-numpy  Get predictions for your pipeline from numpy input.
  serve          Serve a pipeline from a konduit.yaml
  stop-server    Stop the Konduit server associated with a given config...
```

For more help on individual commands you can run

```text
konduit serve --help
```

to get help for the `serve` command \(and all others in a similar way\)

```text
Usage: konduit serve [OPTIONS]

  Serve a pipeline from a konduit.yaml

Options:
  --config TEXT          Relative or absolute path to your konduit serving YAML
                       file.
  --start_server TEXT  Whether to start the server instance after 
                       initialization.
  --help               Show this message and exit.
```

## Common installation issues 

1. Installing`jnius` returns `WARNING: Not able to assign machine() = AMD64 to a cpu value! Using cpu = 'i386' instead!` Fix: Ensure your JAVA environment variables point to a 64-bit version of Java if you're using a 64-bit version of Python. 

