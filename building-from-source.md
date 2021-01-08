---
description: Instructions for building Konduit-Serving binaries from source
---

# Building from source

## Pre-requisites

* JDK 1.8+
* Maven 3+
* Git

## Cloning the repository

Konduit Serving sources are hosted on GitHub. If you have [git](https://git-scm.com/) installed, clone the [konduit-serving repository](https://github.com/KonduitAI/konduit-serving) using the `git clone` command:

```text
git clone https://github.com/KonduitAI/konduit-serving.git
```

## Using the Build Script

After cloning the repository, run `./build.sh --help` to see the available options:

```text
$ ./build.sh --help
-------------------------------------------------------------------
A command line utility for building konduit-serving distro packages.

Usage: bash build.sh [CPU|GPU] [linux|windows|macosx] [tar|zip|exe|rpm|deb]
Example: bash build.sh GPU linux tar,deb
-------------------------------------------------------------------
```

You can create CPU/GPU builds for available platforms by executing their respective commands. 

## Example

An example of creating **Ubuntu \(deb\)** build is as follows:

```text
$ ./build.sh CPU linux deb
-------------------------------------------------------------------
Building project version: 0.1.0-SNAPSHOT
Building a konduit-serving distributable JAR file...
Selecting CHIP=CPU
Building CPU version of konduit-serving for linux with distro types: (deb) ...
Running command: mvn clean install -Dmaven.test.skip=true -Denforcer.skip=true -Djavacpp.platform=linux-x86_64 -Ppython,uberjar,tar,deb -Ddevice=CPU
[INFO] Scanning for projects...
.
.
.
.
.
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  08:10 min
[INFO] Finished at: 2021-01-08T13:40:01+05:00
[INFO] ------------------------------------------------------------------------
----------------------------------------
DEB distro is available at: 
konduit-serving-deb/target/konduit-serving-custom-CPU_0.1.0-SNAPSHOT.deb
-------------------------------------------------------------------
```

