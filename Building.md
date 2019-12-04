[Getting started](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Getting-started) **>** [Dependencies](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies) **>** [**[Building]**](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building) **>** [Installing](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Installing) **>** [Tutorial](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Tutorial)

<hr>

## Linux 
Make sure you have the [dependencies installed](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies).

On most systems you should be able to build with:
```
make
```
which calls cmake under the hood and creates a binary in the /build directory.

### CMake Configuration
CMake allows for extensive configuration of the build definition. The following sections explain some of the most common configuration options available.
#### Options
The CMake definition supports the following options that can be set when invoking CMake:

| Option  | Description | Values | Default |
| ------- | ----------- |------- |-------- |
| CMAKE_BUILD_TYPE | Controls base flags for building. Release includes optimization, Debug is unoptimized  | Debug, Release  | Release |
| PROFILE | Turn on flags required for profiling | ON, OFF | OFF |
| VALGRIND_PROFILE | Turn on flags required for profiling with valgrind in gcc | ON, OFF | OFF |
| GCOV | Turn on flags required for code coverage in gcc | ON, OFF | OFF |
| WARNINGS | Turn on warning flags | ON, OFF | ON |
| STATIC_LINK_VW | Link VW executable statically | ON, OFF | OFF |
| VW_INSTALL | Add install targets | ON, OFF | ON |
| BUILD_TESTS | Build and enable tests | ON, OFF | ON |
| BUILD_JAVA | Add Java targets | ON, OFF | OFF |
| BUILD_PYTHON | Add Python targets | ON, OFF | OFF |
| PY_VERSION | Python version to build Python bindings for | x.x | 2.7 |
| BUILD_DOCS | Add documentation targets | ON, OFF | OFF |

Options can be specified at configuration time on the command line, for example:
```
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=OFF -DVW_INSTALL=OFF
``` 
#### C++ Optimization

The default build type is release. To perform a release build this should be specified at configure time.

```
cmake .. -DCMAKE_BUILD_TYPE=Debug
```

#### Using Vcpkg with Cmake
If using vcpkg for dependencies the toolchain file needs to be supplied to cmake at configuration time:
```
cmake .. -DCMAKE_TOOLCHAIN_FILE=<vcpkg root>/scripts/buildsystems/vcpkg.cmake
```

#### Using Clang

`clang` can be used instead of `gcc` by specifying the compiler prior to configuring.

```
export CC=clang
export CXX=clang++

cmake ..
```

#### Compiling a Static Executable

A statically linked `vw` executable that is not sensitive to boost
version upgrades and can be safely copied between different Linux
versions (e.g. even from Ubuntu to Red-Hat) can be built with:

```
mkdir build
cd build
cmake .. -DSTATIC_LINK_VW=ON
make vw-bin -j `nproc`
```

#### Running Test Suite
The following command will build all dependencies and then run the test suite:
```
make test_with_output -j `nproc`
```
Everything should pass. If you see:
```
minor (<0.001) precision differences ignored
```
That's ok. Floating point arithmetic does not round exactly the same way on all platforms.

[Next step: installing on Linux](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Installing#linux)

## Windows
Make sure you have the [dependencies installed](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies).
### Using Visual Studio
1. Open `vowpalwabbit\vw.sln` in Visual Studio 2017
2. Do not retarget projects if prompted
3. Set startup project as vw (or the test project)
4. Select x64 platform (Configuration Manager>Active solution platform)
5. Select x64 as test platform (Test>Test settings>Default Processor Architecture)
6. Build solution by clicking Build>Rebuild Solution

###  Using command line
Available configurations are "Release" and "Debug". Available platforms are "x64" and "Win32":
```
msbuild /p:Configuration="Release" /p:Platform="x64" vw.sln
```
### Experimental: Using CMake on Windows
#### CMake GUI
1. Open CMake GUI
2. Specify the following entries:
    1. `CMAKE_TOOLCHAIN_FILE=<vcpkg_root>/scripts/buildsystems/vcpkg.cmake`
    2. `VCPKG_TARGET_TRIPLET=x64-windows`
3. Configure
4. Choose generator (for example, Visual Studio 15 2017 Win64)
5. Generate
6. Open Project

#### Command line
```cmd
mkdir build
cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=<vcpkg_root>\scripts\buildsystems\vcpkg.cmake -DVCPKG_TARGET_TRIPLET=x64-windows
.\vowpal_wabbit.sln
```

After the solution is open, building works the same as described in the [Visual Studio section](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Building#using-visual-studio).

[Next step: installing on Windows](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Installing#windows)