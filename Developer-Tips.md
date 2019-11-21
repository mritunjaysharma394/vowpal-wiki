Clang Tidy can be run by:
```
mkdir build
cd build
cmake .. -DCMAKE_EXPORT_COMPILE_COMMANDS=On
cd ..
clang-tidy -p build --checks=* <file_here>
```

This command line works pretty well:
```
clang-tidy -p build --checks=-*,readability-*,modernize-*,performance-*,-modernize-use-trailing-return-type,-readability-uppercase-literal-suffix <file_here>
```