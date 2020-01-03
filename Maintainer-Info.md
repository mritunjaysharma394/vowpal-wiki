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
Note: `--checks=*` produces way to many irrelevant warnings, see below for a recommended list.

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
    - Usage: `python3 -m pylint vowpalwabbit/*.py

## Obtaining new CMake version 
### Ubuntu
Instructions from: https://apt.kitware.com/
```sh
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates gnupg software-properties-common wget
wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | sudo apt-key add -

# 16.04
sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ xenial main'

# 18.04
sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main'

sudo apt-get update

# Optional, ensure keyring stays up to date as it is rotated.
sudo apt-get install kitware-archive-keyring
sudo apt-key --keyring /etc/apt/trusted.gpg del C1F34CDD40CD72DA

sudo apt-get install cmake
```

### Windows
Download and install from: https://cmake.org/download/