# Build requirements

* Visual Studio 2013 or above. It builds using VS 2015, but some issues occurred with the debugger.
* Run the following from Windows Command prompt (cmd.exe) to restore nugets. Just restoring in Visual Studio isn't enough, as the build definition already depends on one of the nugets (ANTLR). 

```Batchfile
cd vowpalwabbit
nuget\NuGet.exe restore vw.sln
```

* To run the tests you need to install [Java Runtime](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
* To avoid space/tab/.. reformatting, install [Editor Config VS Plugin](https://visualstudiogallery.msdn.microsoft.com/c8bccfe2-650c-4b42-bc5c-845e21f96328) 

# Random tips

* Select x64 platform (Configuration Manager \ Active solution platfrom)
* Select x64 as test platform (Test \ Test settings \ Default Processor Architecture)
* Don't trust the Visual Studio 2013 debugger on stack frames in C++/CLI. We observed bogus values. Moving to C++ or C# everything was correct.