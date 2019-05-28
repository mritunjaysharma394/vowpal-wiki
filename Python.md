## Installing
### Windows
If `pip install vowpalwabbit` does not work the most likely reason is because of missing dependencies. In that case use the following instructions to build the package.
1. [Install vcpkg ](https://github.com/Microsoft/vcpkg)
2. [Install dependencies with vcpkg](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#experimental-cmake-build-system-on-windows)
    ```bash
    vcpkg install zlib:x64-windows
    vcpkg install boost-system:x64-windows
    vcpkg install boost-program-options:x64-windows
    vcpkg install boost-test:x64-windows
    vcpkg install boost-align:x64-windows
    vcpkg install boost-foreach:x64-windows
    vcpkg install boost-python:x64-windows
    vcpkg install boost-math:x64-windows
    vcpkg install boost-thread:x64-windows
    ```
3. ` cd <repo_root>/python`
4. `python setup.py --vcpkg-root C:\path\to\vcpkg install`
    - Where `C:\path\to\vcpkg` is the root directory of where you cloned vcpkg
