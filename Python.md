## Installing
### Windows
If `pip install vowpalwabbit` does not work the most likely reason is because of missing dependencies. In that case use the following instructions to build the package.
1. [Install vcpkg ](https://github.com/Microsoft/vcpkg)
2. [Install dependencies with vcpkg](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#experimental-cmake-build-system-on-windows)
    ```bat
    vcpkg --triplet x64-windows install zlib boost-system boost-program-options boost-test boost-align boost-foreach boost-python boost-math boost-thread python3
    ```
3. ` cd <repo_root>`
4. `python setup.py --vcpkg-root C:\path\to\vcpkg install`
    - Where `C:\path\to\vcpkg` is the root directory of where you cloned vcpkg

### OSX
1. Clone the vowpal wabbit repository
2. [Install dependencies](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Dependencies#macos)
3. `cd <repo_root>`
4. `python setup.py install`