This is a tutorial for the new Vowpal Wabbit C# binding. Here's a list of the major features:

* Very efficient serialization from managed to native space using runtime compiled serializers.
* Declarative specification of example data structure.
* Thread-saftey through object pooling and shared models.
* Example level caching (prediction only).
* Improved memory management.

The C# binding is structured in layers and enables multiple use cases:

1. String based examples: use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)
2. Generic data structures (e.g. records consisting of key/value/type tuples): use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h) and VW.VowpalWabbitNamespaceBuilder