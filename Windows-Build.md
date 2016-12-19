# Build requirements

* Visual Studio 2013 **AND** Visual Studio 2015
* Run the following from Windows Command prompt (cmd.exe) to restore nugets. Just restoring in Visual Studio isn't enough, as the build definition already depends on one of the nugets (ANTLR). 
* **Don't** upgrade the solution to Visual Studio 2015. The C++ is not compatible yet.

```Batchfile
cd vowpalwabbit
.nuget\NuGet.exe restore vw.sln
```

## Optional

* [Azure SDK](https://www.microsoft.com/en-us/download/details.aspx?id=51657)
* [WiX](https://wix.codeplex.com/releases/view/624906) to edit the installer 
* To run the tests you need to install [Java Runtime](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
* To avoid space/tab/.. reformatting, install [Editor Config VS Plugin](https://visualstudiogallery.msdn.microsoft.com/c8bccfe2-650c-4b42-bc5c-845e21f96328) 

# Azure Unit Tests
Create required Azure resources using [<img src="http://azuredeploy.net/deploybutton.png">](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Feisber%2Fvowpal_wabbit%2Fmaster%2Fcs%2Funittest%2Fazuredeploy.json)    

Create vw_azure.config in your local checkout folder (or any parent of it):

```
  storageConnectionString = <storage account key>
  inputEventHubConnectionString = <eventhub connection string>
  evalEventHubConnectionString = <eventhub connection string>
```

# Random tips

* Select x64 platform (Configuration Manager \ Active solution platfrom)
* Select x64 as test platform (Test \ Test settings \ Default Processor Architecture)
* Don't trust the Visual Studio 2013 debugger on stack frames in C++/CLI. We observed bogus values. Moving to C++ or C# everything was correct.