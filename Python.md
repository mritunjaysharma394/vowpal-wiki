# Installing
### Windows
Windows has wheels deployed to pypi for Python 3.6 (64-bit) and Python 3.7 (64-bit).
Simply run `pip install vowpalwabbit`

### OSX
1. [Clone repo and Install dependencies](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#macos)
2. `python setup.py install`

### Linux
The commands listed below will be using apt, replace this for the package manager used by your preferred flavor.
1. `sudo apt update`
2. `sudo apt install libboost-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev libboost-math-dev libboost-test-dev libboost-python-dev zlib1g-dev cmake python3 python3-pip`
3. `pip3 install vowpalwabbit`

## Troubleshooting
### Windows
1. If `pip install vowpalwabbit` fails, ensure you are using the 64-bit version of Python 3.6 or 3.7 (The default download bitness for Windows is 32-bit)

2. If you need to build VowpalWabbit from source on Windows, use the following instructions
    1. [Install vcpkg ](https://github.com/Microsoft/vcpkg)
    2. [Install dependencies with vcpkg](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#experimental-cmake-build-system-on-windows)
        - Python 3.6 is built against [commit aa095559](https://github.com/microsoft/vcpkg/tree/aa095559917a495b160986e9ad50556431509ace)
        - Python 3.7 is built against [commit 8c3e093](https://github.com/microsoft/vcpkg/tree/8c3e093d0509fb0c7cc325692834fc1583a05390)
    ```bat
    vcpkg --triplet x64-windows install zlib boost-system boost-program-options boost-test boost-align boost-foreach boost-python boost-math boost-thread python3
    ```
   3. ` cd <repo_root>`
   4. `python setup.py --vcpkg-root C:\path\to\vcpkg install`
        - Where `C:\path\to\vcpkg` is the root directory of where you cloned vcpkg

### OSX
There are several known issues regarding the VowpalWabbit installation for OSX.
1. Building and installing VowpalWabbit with Anaconda will crash on OSX 10.14 or later. See [this issue](https://github.com/VowpalWabbit/vowpal_wabbit/issues/2100) for details
    - Uninstall Anaconda before attempting to install VowpalWabbit
2. CMake can't find Boost
    - Ensure the [required Boost libraries](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#macos) are installed
    - If your Boost version is 1.70 or earlier, install using the command `python setup.py --enable-boost-cmake install`


# Debugging Python/C++

The Python bindings can be debugged in a mixed mode fashion by connecting two debuggers. This lets you set breakpoints in both languages.

## Requirements
- VSCode
- [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
- [C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

## Steps
1. Edit `launch.json` to add the [required targets](#launchjson)
2. Ensure VW bindings are installed as debug
    2. Right now this means editing line 88 of `setup.py` and setting `config` to `'debug'`
    3. Then run `setup.py install -f` under the repo root to install the bindings
3. Open the Python file to debug and run ``"Launch current Python file"`` debugger target
4. While the Python debugger is attached and broken at some breakpoint run ``"Attach GDB to Python"`` debugger target and select the Python process
    - If you get an error message like (I think this only happens in WSL):
        ```sh
        Error getting authority: Error initializing authority: Could not connect: No such file or directory
        [1] + Done(127)                  /usr/bin/pkexec "/usr/bin/gdb" --interpreter=mi --tty=${DbgTerm} 0<"/tmp/Microsoft-MIEngine-In-zxx2mqu5.eh4" 1>"/tmp/Microsoft-MIEngine-Out-3ynea04r.784"
        ```
    - Then run the following:     `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope`

**Note:** you cannot step over the language boundary. To move between you need to set breakpoints on either side and use `"continue"`.
## Resources
### launch.json
```json
{
  "name": "Launch current Python file",
  "type": "python",
  "request": "launch",
  "pythonPath": "/usr/bin/python",
  "program": "${file}",
  "cwd": "${workspaceFolder}",
  "console": "integratedTerminal"
},
{
  "name": "Attach GDB to Python",
  "type": "cppdbg",
  "request": "attach",
  "program": "/usr/bin/python",
  "processId": "${command:pickProcess}",
  "MIMode": "gdb"
}
```

### Links
- [Original instructions](https://gist.github.com/asroy/ca018117e5dbbf53569b696a8c89204f)
- [Issue describing issue of using external terminal with WSL](https://github.com/Microsoft/vscode-python/issues/2732)
- [Issue describing elevation issue on WSL](https://github.com/microsoft/vscode-remote-release/issues/99 )