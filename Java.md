# Welcome lonely traveler.

# Usage
Look [here](https://github.com/VowpalWabbit/vowpal_wabbit/tree/master/java)

## Build & Release

CI build pipeline can be found [here](https://dev.azure.com/vowpalwabbit/Vowpal%20Wabbit/_build?definitionId=33&_a=summary). 

It's a dry-run of the same publish [pipeline](https://dev.azure.com/vowpalwabbit/Vowpal%20Wabbit/_build?definitionId=27) w/o credentials set. Run this manually to trigger a Maven/Sonatype release of the jar file.

Once the jar is published to sonatype it needs another approval step. Visit [Sonatype](https://oss.sonatype.org/#stagingRepositories) and "close & release" the staged jar.
