# Building from source

Konduit Serving sources are hosted on GitHub. If you have `git` installed, clone the [konduit-serving repository](https://github.com/KonduitAI/konduit-serving) using the `git clone` command:

```text
git clone https://github.com/KonduitAI/konduit-serving.git
```

## Python module

To install the `konduit` Python module from source, in the `python` directory, after installing Cython, run

```text
pip install .
```

To install all extensions needed for development run

```text
pip install -e '.[tests,codegen,dev]'
```

The `dev` dependencies use `black` as a pre-commit hook to lint your code automatically. To activate this functionality, run `pre-commit install` on the command line first.

### Running tests

Install test dependencies using `pip install 'konduit[tests]'` if you want to run tests.

On Windows, compiling the test dependencies requires Visual Studio Build Tools 14.0, which can be installed from [here](https://visualstudio.microsoft.com/downloads/). You may also need to install the Windows 8.1 / 10 SDK. See Python's [_WindowsCompilers_](https://wiki.python.org/moin/WindowsCompilers) page for details.

The tests also require `bert_mrpc_frozen.pb` to be placed in the `python/tests` folder. Run the following code in `python/tests`:

```text
curl https://deeplearning4jblob.blob.core.windows.net/testresources/bert_mrpc_frozen_v1.zip --output bert.zip
unzip bert.zip
```

The resulting JAR will be generated at the base of the `konduit` project. To copy that JAR into the `tests` folder and prepare the documentation \(in the `docs` folder\) to be tested within the testing framework, run:

```text
cd tests
./prepare_doc_tests.sh
```

The tests are then run with `pytest`:

```text
cd python/tests
python -m pytest .
```

To quickly run unit tests \(recommended before each commit\), or run the full set of integration tests, you can do:

```text
pytest -m unit
pytest -m integration
```

To also run documentation tests with `doctest` for an individual file, simply run:

```text
 python -m doctest ../konduit/server.py -v
```

## JAR

A Java Archive \(JAR\) file is used to bundle a Java program.

{% hint style="info" %}
Building the Konduit Serving JAR requires Maven and JDK 8.
{% endhint %}

### Manual build

Run the following commands in the root directory of konduit-serving:

```text
python build_jar.py --os <your-platform>
```

where `<your-platform>` is picked from `windows-x86_64`,`linux-x86_64`,`linux-x86_64-gpu`, `macosx-x86_64`, `linux-armhf` and `windows-x86_64-gpu`, depending on your operating system and architecture. Use the `--help` flag to view the full list of arguments.

An additional `--spin` argument provides the option to package Python \(`python`\), PMML \(`pmml`\), both \(`all`\) or neither \(`minimal`\). By default, both Python and PMML are packaged. Python bundling is not encouraged on ARM platforms, and PMML bundling is not encouraged if [AGPL licensing](https://www.gnu.org/licenses/agpl-3.0.en.html) is an issue.

### Building with the command line interface

Once the `konduit` Python package is installed, you have access to a command line interface \(CLI\) tool called `konduit`.

The `init` command:

1. gets the latest Konduit Serving code, then
2. builds the Java dependencies needed for`konduit`.

It assumes that you have `git` installed on your system and that `python` is available.

Run:

```bash
konduit init --os <your-platform>
```

where `<your-platform>` is picked from `windows-x86_64`, `linux-x86_64`, `linux-x86_64-gpu`, `macosx-x86_64`, `linux-armhf` and `windows-x86_64-gpu`, depending on your operating system and architecture.

An additional `--spin` argument provides the option to package Python \(`python`\), PMML \(`pmml`\), both \(`all`\) or neither \(`minimal`\). By default, both Python and PMML are packaged. Python bundling is not encouraged on ARM platforms, and PMML bundling is not encouraged if [AGPL licensing](https://www.gnu.org/licenses/agpl-3.0.en.html) is an issue.

To rebuild the Konduit Serving JAR without re-downloading sources, run `build` instead of `init` with the appropriate flags.

{% hint style="info" %}
### Known issues

* `konduit init` fails for  `linux-86_64-gpu` \([\#115](https://github.com/KonduitAI/konduit-serving/issues/115)\)
{% endhint %}

## Linux builds

Generally, the Linux builds of Konduit Serving perform the following tasks on installation:

1. Copy Konduit Serving JAR file to `/opt/konduit/serving/`;
2. Create the necessary environment variables; and
3. Install a Konduit Serving-specific Conda distribution with `install-python.sh`. 

### RPM \(CentOS, Redhat, etc.\)

Konduit Serving RPM packages are generated using the [RPM Maven Plugin](https://www.mojohaus.org/rpm-maven-plugin/).

First, install required packages with `yum`:

```text
sudo yum install -y java-1.8.0-openjdk-devel which rpm-build redhat-rpm-config
```

This command installs the developer tools for developing Java programs using JDK 8, the `which` package to locate a program file's path, tools to build RPM files and Red Hat-specific RPM configuration files.

In the root folder of the `konduit-serving` project, run the following command to build RPM files using Maven Wrapper:

```text
./mvnw clean package -Ppython,pmml,uberjar,tar,rpm -Dmaven.test.skip=true -Djavacpp.platform=linux-x86_64 -Dchip=cpu
```

The Maven Wrapper `mvnw` script allows Maven to be used even if `mvn` is not available on the system PATH. This command runs the Maven goals `clean` and `install` with the following arguments:

* `maven.test.skip=true`
* Profiles: `uberjar,tar,rpm` \(ensure this is specified without spaces in between\). The profiles `python` and `pmml` are optional. 
* `chip`: `cpu` \(use `gpu` to enable CUDA support\)
* `javacpp.platform`: `linux-x86_64`

The `clean install` command first deletes previously compiled Java sources and resources; then compiles, tests and packages the Java project and copies it into the relevant folder. The path where the RPM file is saved depends on the `spin.version` \(default `custom`\) and the chip \(`cpu` or `gpu`\) .

Use the YUM command `yum localinstall`to install the RPM file.

```text
# replace <spin.version> with the spin version specified
cd konduit-serving-rpm/target/rpm/konduit-serving-<spin.version>-cpu/RPMS/x86_64/
sudo yum localinstall -y *.rpm
```

### DEB \(for Ubuntu and other Debian-based systems\)

Konduit Serving Debian packages are generated with the [jdeb](https://github.com/tcurdt/jdeb) library.

Install JDK 8 using `apt-get`:

```text
sudo apt-get install openjdk-8-jdk curl
```

In the root directory of the Konduit Serving project, run the `mvnw` script with parameters:

```text
./mvnw clean package -Ppython,pmml,uberjar,tar,deb -Dmaven.test.skip=true -Djavacpp.platform=linux-x86_64 -Dchip=cpu
```

* `maven.test.skip=true`
* Enable the profiles `uberjar,tar,deb` \(ensure this is specified without spaces in between\). The `python` and `pmml` profiles are optional. 
* `chip`: `cpu` \(use `gpu` to enable CUDA support\)
* `javacpp.platform`: `linux-x86_64`

{% hint style="info" %}
The error `java.io.IOException: This archives contains unclosed entries.` usually indicates insufficient disk space \(see [tcurdt/jdeb\#234](https://github.com/tcurdt/jdeb/issues/234)\).
{% endhint %}

Finally, use `dpkg` to install the built package:

```text
sudo dpkg -i konduit-serving-deb/target/konduit-serving-custom-cpu_0.1.0-SNAPSHOT.deb
```

Note that `dpkg` does not support dependencies. If you run into missing dependencies, run

```text
sudo apt-get install -f
```

to install dependencies. Alternately, use the `gdebi` package to install the local DEB package \(see this [StackExchange thread ](https://unix.stackexchange.com/questions/159094/how-to-install-a-deb-file-by-dpkg-i-or-by-apt)for details\), or simply [use `apt-get install` to install the local package](https://askubuntu.com/a/795048) \(apt 1.1 and above\):

```text
cd konduit-serving-deb/target
sudo apt-get install ./*.deb
```

### Tarball

Konduit Serving can also be built as a tarball, where the JAR file and associated scripts are packaged in a gzip-compressed tar file. To build a Konduit Serving tar file, run the following Maven Wrapper command in the root folder of the Konduit Serving project:

```text
./mvnw clean package -Ppython,pmml,uberjar,tar -Dmaven.test.skip=true -Djavacpp.platform=linux-x86_64 -Dchip=cpu
```

This generates two compressed files in the `target` directory of the `konduit-serving-tar`folder: a tar \(`.tar.gz`\) and a zip \(`.zip)` file. In addition to the JAR file, the tar file contains a script to install a Conda distribution \(`ìnstall-python.sh`\) and a script to set environment variables \(`bin/konduit-serving`\).

After extracting the tar file, first run the `konduit-serving`shell script:

```text
cd bin
chmod u+x konduit-serving # allow user to execute script
./konduit-serving
```

then the `ìnstall-python.sh` script:

```text
cd .. 
chmod u+x install-python.sh 
./install-python.sh
```

## Konduit Serving Conda distribution

The following packages are included in this Conda distribution:

| Package | Version |
| :--- | :--- |
| NumPy | 1.16.4 |
| Jupyter | 1.0.0 |
| SciPy | 1.3.3 |
| requests | 2.22.0 |
| pandas | 0.24.2 |
| TensorFlow | 1.15.0 |
| Keras | 2.2.4 |
| konduit | 0.1.3rc1 |
| scikit-learn | 0.22 |
| matplotlib | 3.1.2 |
| PyTorch | 1.3.1 |
| torchvision | 0.4.2 |
| CUDA Toolkit | 10.1 |
| OpenJDK | 8 |

Note that packages are sourced from the following Anaconda channels, in descending order of priority: [pytorch](https://anaconda.org/pytorch), [conda-forge](https://anaconda.org/conda-forge), [anaconda](https://anaconda.org/anaconda), [konduitai](https://anaconda.org/konduitai).

