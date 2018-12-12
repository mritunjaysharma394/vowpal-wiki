## Linux 
On most systems you should be able to build with:
```
mkdir build
cd build
cmake ..
make
```

Note: to enable parallel builds pass `-j <parallel_jobs>` to make

#### CMake options
The CMake definition supports the following options that can be set when invoking CMake:

| Option  | Description | Values | Default |
| ------- | ----------- |------- |-------- |
| CMAKE_BUILD_TYPE | Controls base flags for building. Release includes optimization, Debug is unoptimized  | Debug, Release  | Debug |
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

Options can be specified at configuration time on the command line:
```
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=OFF -DVW_INSTALL=OFF
``` 

#### Using Vcpkg with Cmake
If using vcpkg for dependencies the toolchain file needs to be supplied to cmake at configuration time:
```
cmake .. -DCMAKE_TOOLCHAIN_FILE=<vcpkg root>/scripts/buildsystems/vcpkg.cmake
```
[Next step: running tests on Linux]()

## Windows
### Using Visual Studio
1. Open `vowpal_wabbit\vowpalwabbit\vw.sln` in Visual Studio 2017
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

[Next step: running tests on Windows]()