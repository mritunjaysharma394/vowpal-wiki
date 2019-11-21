Clang Tidy can be run by:
```
mkdir build
cd build
cmake .. -DCMAKE_EXPORT_COMPILE_COMMANDS=On
cd ..
clang-tidy -p build --checks=* vowpalwabbit/*.cc vowpalwabbit/*.h
```