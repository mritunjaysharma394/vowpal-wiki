[Getting started](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Getting-started) **>** [**[Dependencies]**](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies) **>** [Building](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building) **>** [Installing](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Installing) **>** [Tutorial](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Tutorial)

Before building VW from source the following dependencies must be satisfied.

## Ubuntu
```shell
# Core
sudo apt-get install libboost-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev libboost-math-dev zlib1g-dev cmake

# Optional: test
sudo apt-get install libboost-test-dev  
pip install six

# Optional: Python bindings
sudo apt-get install libboost-python-dev

# Optional: Java
sudo apt-get install openjdk-8-jdk
```

[Next step: building on Linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#linux)

## MacOS
```shell
brew install cmake
brew install boost --with-python

# TODO Java dependency
```

The rest of the Linux instructions should apply to MacOS too.

[Next step: building on Linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#linux)

## Windows
On Windows dependencies are managed with either Nuget, open `vowpalwabbit/vw.sln` in Visual Studio 2017 and restore Nuget dependencies.

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
