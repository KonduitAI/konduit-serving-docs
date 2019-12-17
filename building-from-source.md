# Building from source

## JAR

A Java Archive \(JAR\) file is used to bundle a Java program. 

{% hint style="info" %}
Building the Konduit Serving JAR requires Maven and JDK 8.
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

Once the `konduit` package is installed, you have access to a command line interface \(CLI\) tool called `konduit`.

The `init` command:

1. gets the latest Konduit Serving code, 
2. builds the Java dependencies needed for`konduit`, then 
3. exports the location of the Konduit Serving JAR as an environment variable. 

It assumes that you have `git` installed on your system and that `python` is available.

Run:

```bash
konduit init --os <your-platform>
```

where `<your-platform>` is picked from `windows-x86_64`, `linux-x86_64`, `linux-x86_64-gpu`,  `macosx-x86_64`, `linux-armhf` and `windows-x86_64-gpu`, depending on your operating system and architecture.

To rebuild the Konduit Serving JAR without adding the `KONDUIT_JAR_PATH` environment variable, run `konduit build` instead with the appropriate flags.

{% hint style="info" %}
### Known issues

* `konduit init` fails for  `linux-86_64-gpu` \([\#115](https://github.com/KonduitAI/konduit-serving/issues/115)\)
{% endhint %}

