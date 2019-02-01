### When using WSL (Windows Subsytem for Linux)
- If the repo is cloned in Windows and used in the Linux environment, shell scripts will have CRLF line endings and will need to be converted to work.
- A strange bug was seen that caused the `vw_jni` target to fail to build. A full fix isn't known but the following were factors:
  - CMake version 3.5.1
  - WSL Ubuntu 16.04
  - Java was installed in Windows and added to the Windows path when compiling `vw_jni`
  - Setting JAVA_HOME caused CMake to display the right dependency at configure time but the Windows files were actually used