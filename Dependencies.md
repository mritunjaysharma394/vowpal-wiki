[Getting started](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Getting-started) **>** [**[Dependencies]**](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies) **>** [Building](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building) **>** [Installing](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Installing) **>** [Tutorial](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Tutorial)

<hr>

Before building VW from source the following dependencies must be satisfied.

## Ubuntu
Ensure you have the source:
```
git clone https://github.com/VowpalWabbit/vowpal_wabbit.git
cd vowpal_wabbit
```
Get dependencies:
```shell
# Core
sudo apt install libboost-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev libboost-math-dev libboost-test-dev zlib1g-dev cmake

# Optional: test
pip install six

# Optional: Python bindings
sudo apt install libboost-python-dev

# Optional: Java
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt update
sudo apt install openjdk-11-jdk
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64/"
```

**Note:** If using *WSL* and you cloned the repo in Windows you will need to run `git submodule update --init --recursive` before running cmake. Otherwise the submodule will be downloaded with git in bash and the line endings will break git commands from Windows. Otherwise, submodules are automatically downloaded by cmake.

[Next step: building on Linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#linux)

## MacOS
Ensure you have the source:
```
git clone https://github.com/VowpalWabbit/vowpal_wabbit.git
cd vowpal_wabbit
```
#### HomeBrew
```shell
brew install cmake
brew install boost

# If you want to statically link VW
brew install zlib

# If you want to build with python 2 support
brew install boost-python

# If you want to build with python 3 support
brew install boost-python3
```

#### MacPorts
```bash
# Build Boost for Mac OS X 10.8 and below
port install boost +no_single -no_static +openmpi +python27 configure.cxx_stdlib=libc++ configure.cxx=clang++

# Build Boost for Mac OS X 10.9 and above
port install boost +no_single -no_static +openmpi +python27
```

The rest of the Linux instructions should apply to MacOS too.

[Next step: building on Linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#linux)

## Windows
Ensure you have the source:
```
git clone https://github.com/VowpalWabbit/vowpal_wabbit.git
cd vowpal_wabbit
```
Download submodules:
```
git submodule update --init --recursive
```
On Windows dependencies are managed with either Nuget, open `vowpalwabbit/vw.sln` in Visual Studio 2017 and restore Nuget dependencies.

Note: VC++ v14.00  toolset must be installed. It can be done either as Visual Studio 2017 feature or as part of Visual Studio 2015. Windows 8.1 SDK must be installed as well.

[Next step: building on Windows](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#windows)

#### Experimental: CMake build system on Windows
Note: The CSharp projects are not yet converted to CMake for Windows. So the CMake generated solution is only for C++ projects for the time being. For this reason the existing solution can not yet be deprecated. 

When using CMake for Windows, dependencies are easily managed through [Vcpkg](https://github.com/Microsoft/vcpkg). Install the following dependencies with Vcpkg:
```cmd
vcpkg install zlib:x64-windows
vcpkg install boost-system:x64-windows
vcpkg install boost-program-options:x64-windows
vcpkg install boost-test:x64-windows
vcpkg install boost-align:x64-windows
vcpkg install boost-foreach:x64-windows
vcpkg install boost-python:x64-windows
vcpkg install boost-math:x64-windows
vcpkg install boost-thread:x64-windows
```

[Next step: building on Windows with CMake](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#experimental-using-cmake-on-windows)
