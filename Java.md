# Welcome lonely traveler.

# Usage
Look [here](https://github.com/VowpalWabbit/vowpal_wabbit/tree/master/java)

## Build

CI build pipeline can be found [here](https://dev.azure.com/vowpalwabbit/Vowpal%20Wabbit/_build?definitionId=33&_a=summary). 

## Performing a release

1. Make sure the sub-version is set correctly and does **not** include -SNAPSHOT (e.g. .0) in the [pom.xml.in](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/java/pom.xml.in#L7)
2. Manually run the build [pipeline](https://dev.azure.com/vowpalwabbit/Vowpal%20Wabbit/_build?definitionId=27).
3. Visit [Sonatype](https://oss.sonatype.org/#stagingRepositories) and "close & release" the staged jar.
