[Getting started](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Getting-started) **>** [**[Dependencies]**](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies) **>** [Building](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building) **>** [Installing](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Installing) **>** [Tutorial](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Tutorial)

<hr>

Before building VW from source the following dependencies must be satisfied.

## Ubuntu
Ensure you have the source:
```
git clone --recursive https://github.com/VowpalWabbit/vowpal_wabbit.git
cd vowpal_wabbit
```
Get dependencies:
```shell
# Core
sudo apt install libboost-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev libboost-math-dev libboost-test-dev zlib1g-dev cmake g++

# Optional: test
pip install six

# Optional: Python bindings
sudo apt install libboost-python-dev

# Optional: Java, Ubuntu 16.04 or newer
sudo apt install default-jdk
```

**Note:** If using *WSL* and you cloned the repo in Windows you will need to run `git submodule update --init --recursive` before running cmake. Otherwise the submodule will be downloaded with git in bash and the line endings will break git commands from Windows. Otherwise, submodules are automatically downloaded by cmake.

[Next step: building on Linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#linux)

## MacOS
Ensure you have the source:
```
git clone --recursive https://github.com/VowpalWabbit/vowpal_wabbit.git
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
git clone --recursive https://github.com/VowpalWabbit/vowpal_wabbit.git
cd vowpal_wabbit
```
- Either Visual Studio 2017 or 2019 can be used, VS2019 is recommended
- Visual Studio 2017 (v141) toolset must be installed if using VS 2019
- `10.0.16299.0` Windows 10 SDK must be installed as well.
- Nuget is used for other dependencies. Open `vowpalwabbit/vw.sln` in Visual Studio and restore Nuget dependencies

[Next step: building on Windows](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#windows)

#### Experimental: CMake build system on Windows
Note: The CSharp projects are not yet converted to CMake for Windows. So the CMake generated solution is only for C++ projects for the time being. For this reason the existing solution can not yet be deprecated. 

When using CMake for Windows, dependencies are easily managed through [Vcpkg](https://github.com/Microsoft/vcpkg). Install the following dependencies with Vcpkg:
```cmd
vcpkg --triplet x64-windows install zlib boost-system boost-program-options boost-test boost-align boost-foreach boost-python boost-math boost-thread python3
```

[Next step: building on Windows with CMake](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#experimental-using-cmake-on-windows)
