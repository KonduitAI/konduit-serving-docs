---
description: Instructions for installing and setting up built binaries
---

# Installing Binaries

## System requirements

### **OPERATING SYSTEMS**

Konduit Serving is supported on 

* Linux
* MacOS
* Windows

### **DEPENDENCIES**

* Ensure that you have JDK 8.0 installed
* Install Python 3.7.x to be used with the Python Step

### **HARDWARE REQUIREMENTS**

Binaries are provided for

* Intel/x86 architectures
* ARM

### **GPU SUPPORT** 

Hardware acceleration with 

* CUDA version 11.0

## INSTALLATION INSTRUCTIONS

Installation for each type of distribution is as follows: 

### TAR/ZIP

After extracting, just put the `bin` folder into the `PATH` environment variable.

### DEB

```text
dpkg -i konduit-serving-deb/target/**/*.deb
```

### RPM

```text
rpm -i konduit-serving-rpm/target/**/*.rpm
```

### EXE

Just put the folder containing the `konduit.exe` file into the `PATH` environment variable.

## CONFIGURATIONS

### LINUX/MACOS

The configuration file is present inside the `conf/konduit-serving-env.sh` file for the `TAR/ZIP` distro. For Konduit-Serving distro installed from the `DEB` and `RPM` packages, the configuration file is present inside the location `/opt/konduit/conf/konduit-serving-env.sh`. Just uncomment the variables you want to set and specify the value you like.

### WINDOWS

The configuration file for the `TAR/ZIP` distro is present inside the `conf/konduit-serving-env.cmd` file. Just uncomment the variables you want to set and specify the value you like.

