Clang Tidy can be used by configuring the CMake project and adding the following argument:
```
"-DCMAKE_CXX_CLANG_TIDY=clang-tidy -checks=-*,readability-*"
```