## Installing
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