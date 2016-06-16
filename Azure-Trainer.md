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

# REST API
Each request expects a HTTP authorization header: "Authorization: <Insert AdminToken here>".
- _<trainer url>/reset_ Resets the current model.

# Links
- [Provisioning](https://github.com/multiworldtesting/ds-provisioning/blob/master/templates/OnlineTrainerTemplate.json) using [ARM](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/). The packageLink.uri needs to point to blob on Azure storage and contain a SAS token.
- [Binaries](https://github.com/eisber/vowpal_wabbit/releases)
- [Code](https://github.com/eisber/vowpal_wabbit/tree/master/cs/azure)