# Build requirements

* Run the following to restore nugets. Just restoring in Visual Studio isn't enough, as the build definition already depends on one of the nugets (ANTLR). 

```Batchfile
cd vowpalwabbit
nuget\NuGet.exe restore vw.sln
```

* To run the tests you need to install [Java Runtime](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)

# Random tips

* Select x64 platform (Configuration Manager \ Active solution platfrom)
* Select x64 as test platform (Test \ Test settings \ Default Processor Architecture)