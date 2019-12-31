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

## GDB - Pretty printing
The [pretty printing script](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/vowpalwabbit/gdb_pretty_printers.py) allows GDB to visualize types that it cannot by default, such as `v_array`. This works both with command line and using GDB in VSCode. The same thing is done in Visual Studio on Windows using the [natvis file](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/vowpalwabbit/vw_types.natvis), which is included in the project and automatically loaded. 

In order to use the GDB pretty printers, they need to be loaded. This can either be done automatically when GDB is started or each time.

To load the script into a running gdb session you can run:
```
(gdb) python exec(open('path/to/vowpalwabbit/gdb_pretty_printers.py').read())
```

To automatically load it, create the file if it does not exist `~/.gdbinit`, and add the script load into that file:
```
python exec(open('path/to/vowpalwabbit/gdb_pretty_printers.py').read())
```

If using VSCode, ensure the launch target contains the following properties:
```
MIMode": "gdb",
"setupCommands": [
    {
        "description": "Enable pretty-printing for gdb",
        "text": "-enable-pretty-printing",
        "ignoreFailures": true
    }
]
```

## Python tools
### Installation
```sh
pip3 install yapf mypy pylint --user
```

### Tools
- `yapf` - Code formatter
    - Usage: `python3 -m yapf -i python/vowpalwabbit/*.py`
    - By default uses PEP8 style
    - Any other formatter can be used
- `mypy` - Static type checking
    - Usage: `python3 -m mypy vowpalwabbit/*.py --ignore-missing-imports`
        - `--ignore-missing-imports` is required as the native extension, `pylibvw, does not have type stubs.
    - `mypy` supports mixed dynamic and static typed Python, so the code can be annotated one function at a time
- `pylint` - Python code linting
    - Usage: `python3 -m pylint vowpalwabbit/*.py`