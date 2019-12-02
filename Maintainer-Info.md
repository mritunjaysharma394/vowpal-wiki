## Issue filters:
- [Remove work item like issues from list](https://github.com/VowpalWabbit/vowpal_wabbit/issues?page=1&q=is%3Aissue+is%3Aopen+-label%3A%22Feature+Request%22+-label%3A%22Atomization%22+-label%3A%22Continuous+Action%22+-label%3A%22Technical+Debt%22&utf8=%E2%9C%93) 

## Clang Tidy
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