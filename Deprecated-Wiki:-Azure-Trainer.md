Vowpal Wabbit can be hosted on Azure using a cloud service to train models.
Data can be fed using [Azure EventHubs](https://azure.microsoft.com/en-us/services/event-hubs/) and by creating [JSON](JSON) formatted examples.

_Note:_ As of today the trainer expects contextual bandit-style data, as it performs evaluation. With minor modifications one should be able to train arbitrary models.

# Configuration
At deployment time [cscfg](https://github.com/eisber/vowpal_wabbit/blob/master/cs/azure_service/ServiceConfiguration.Cloud.cscfg) or 
at runtime time through the Azure portal. The trainer observes configuration changes and restarts as required. 


- **APPINSIGHTS_INSTRUMENTATIONKEY**: An [Application Insights](https://azure.microsoft.com/en-us/services/application-insights/) key used for central logging and performance monitoring.
- **Diagnostics.ConnectionString**: Azure storage connection string for startup logging.
- **StorageConnectionString**: Azure storage connection string used to output models.
- **JoinedEventHubConnectionString**: EventHubs connection string used for data ingestion.
- **EvalEventHubConnectionString**: EventHubs connection string used to output evaluation.
- **AdminToken**: The authorization token required to invoke the trainers REST API.
- **CheckpointIntervalOrCount**: The trainer checkpoints in regular intervals. If the configuration contains ':' it is treated as time span (see [format](https://msdn.microsoft.com/en-us/library/ee372286(v=vs.110).aspx)), otherwise a number representing the number of examples after which checkpointing shall be performed. Checkpoint data includes the model, a list of events since the last checkpoint (trackback) and the EventHubs position.
- **EnableExampleTracing**: Traces each example in VW string format to AppInsights. 

# Upgrade an existing deployment

* download cscfg from Management console using this [powershell script](https://github.com/eisber/vowpal_wabbit/blob/master/cs/azure_service/DownloadServiceConfiguration.ps1)
* download the cspkg from [GitHub release page](https://github.com/eisber/vowpal_wabbit/releases). Make sure you select the correct VM size.
* navigate to Azure portal and your trainer instance, select "Update"
* Select: _Deploy even if one or more roles contain a single instance_
* Select: _Allow the update if role sizes change or if the number of roles change_
* Hit start

# REST API
Each request expects a HTTP authorization header: "Authorization: <Insert AdminToken here>".
- _<trainer url>/reset_ Resets the current model.
- _<trainer url>/status_ returns a status and live performance counter values.

One can inject an offline trained model (aka warmstart) using

```bash
 curl  -v --request PUT --data-binary @<filename>  "<trainer url>/reset" --header "Authorization: <Insert AdminToken here>"
```
# Links
- [Provisioning](https://github.com/multiworldtesting/ds-provisioning/blob/master/templates/OnlineTrainerTemplate.json) using [ARM](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/). The packageLink.uri needs to point to blob on Azure storage and contain a SAS token.
- [Binaries](https://github.com/eisber/vowpal_wabbit/releases)
- [Code](https://github.com/eisber/vowpal_wabbit/tree/master/cs/azure)

# Debugging
To run the debug version on Azure, connect using Remote Desktop and copy C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\redist\Debug_NonRedist\x64\Microsoft.VC120.DebugCRT\* to D:\Windows\System32