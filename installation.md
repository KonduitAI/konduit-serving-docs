# Installation

[![PyPI](https://img.shields.io/pypi/v/konduit?style=for-the-badge)](https://pypi.org/project/konduit/0.1.2/)[![Conda \(channel only\)](https://img.shields.io/conda/vn/konduitai/konduit?color=%233EB049&style=for-the-badge)](https://anaconda.org/konduitai/konduit)

## System requirements

**Operating systems** Konduit Serving is supported on Linux, macOS and Windows.

**Dependencies** Ensure that you have JDK 8.0 installed. To use the Python SDK, install Python 3.7 and above.

**Hardware requirements** Binaries are provided for Intel/x86 architectures. For ARM support, see the [_Building from source_](building-from-source.md#manual-build) page.

**GPU**: Hardware acceleration with CUDA version 10.1 \(included in GPU build\) is supported.

## Installation

Install `konduit` from PyPI with

```bash
pip install konduit
```

{% hint style="warning" %}
A version of Konduit Serving with the command line interface \(CLI\) is not currently available on PyPI. To obtain the CLI, clone the [konduit-serving](https://github.com/KonduitAI/konduit-serving) repository, and in the `python` folder run

`pip install`.
{% endhint %}

If using the Anaconda distribution, you may install `konduit` from the `konduitai` Anaconda channel. First add the `konduitai` channel:

```text
conda config --add channels konduitai
```

then install `konduit` with:

```text
conda install -c konduitai konduit
```

You may need to install Cython before installing `konduit` using

```text
pip install cython
```

We recommend using Python 3.7+.

{% hint style="warning" %}
`konduit` PyPI wheels and conda packages do not currently ship with Konduit Serving JARs. Refer to the [_Building from source_](building-from-source.md#manual-build) page for instructions on compiling a Konduit Serving JAR.
{% endhint %}

## Set environment variables manually

In the absence of the `KONDUIT_JAR_PATH` environment variable, the Python SDK looks for the Konduit Serving JAR file in `~/.konduit/konduit-serving`. To overwrite this default, you can set a default location for the Konduit Serving JAR using environment variables.

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

## Common installation issues

1. Installing `pyjnius` returns

   ```text
   WARNING: Not able to assign machine() = AMD64 to a cpu value! Using cpu = 'i386' instead!
   ```

   Fix: Ensure your JAVA environment variables point to a 64-bit version of Java if you're using a 64-bit version of Python, or a 32-bit version of Java if you're using a 32-bit version of Python \(see [kivy/pyjnius\#390](https://github.com/kivy/pyjnius/issues/390)\).

2. When running `konduit` commands on Windows, the following error message is returned:

   ```text
   ImportError: DLL load failed: The specified module could not be found.
   ```

   Fix: On Windows, `pyjnius` requires an additional PATH variable to locate `jvm.dll`. Refer to the [pyjnius documentation](https://pyjnius.readthedocs.io/en/stable/installation.html#installation-for-windows) for details.

