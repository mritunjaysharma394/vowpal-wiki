This is a tutorial for the new Vowpal Wabbit C# binding. Here's a list of the major features:

* Very efficient serialization from managed to native space using runtime compilation.
* Declarative specification of example data structure.
* Thread-saftey through object pooling and shared models.
* Example level caching (prediction only).
* Improved memory management.

The C# binding is structured in layers and enables multiple use cases ordered by the level of abstraction:

1. User defined data types: use [VW.VowpalWabbit<TUserType>] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/VowpalWabbit.cs)
2. Generic data structures (e.g. records consisting of key/value/type tuples): use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h) and [VW.VowpalWabbitNamespaceBuilder] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)
3. String based examples: use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)

# Usage

```PowerShell
Install-Package VowpalWabbit
```

The package includes:

* C++ part of vowpal wabbit compiled for Windows x64 Release
* C++/CLI wrapper
* C# wrapper supporting declarative data to feature conversion
* PDB debug symbols
* Source
* IntelliSense documentation

Note: I'm aware of symbolsource.org, but due to some weird reference to system headers such as undname.h, I was unable to upload a symbols nuget. 

# Examples
Through out the samples the following dataset from [[Rcv1-example]] is used:

<pre>
1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02 85:6.1853945e-02 ... 
0 |f 9:8.5609287e-02 14:2.9904654e-02 19:6.1031535e-02 20:2.1757640e-02 ... 
...
</pre>

## User defined data types
Pro | Cons
--- | ----
very performant | one-time overhead of rather expensive serializer compilation
declarative data to feature conversion using attributes and type information | 

The following class Row is an example of a user defined type usable by the serializer.

```c#
using VW.Interfaces;
using VW.Serializer.Attributes;
using System.Collections.Generic;

public class Row : IExample
{
	[Feature(FeatureGroup = 'f', Namespace = "eatures", Name = "const", Order = 2)]
        public float Constant { get; set; }

        [Feature(FeatureGroup = 'f', Namespace = "eatures", Order = 1)]
        public IList<KeyValuePair<string, float>> Features { get; set; }

        public string Line { get; set; }

        public ILabel Label { get; set;}
}
```

The serializer follows an opt-in model, thus only properties annotated using \[Feature\] are transformed into vowpal wabbit features. The feature attribute supports the following properties:

* FeatureGroup: it's the first character of the namespace in the string format.
* Namespace: concatenated with the FeatureGroup

## Generic data structures
Pro: most performant variant.  
Pro: provides maximum flexibility with feature representation.  
Pro: suited for generic data structures (e.g. records, data table, ...).  
Con: results might not be reproducible using VW binary as it allows for feature representation not expressible through the string format.   
Con: verbose.

```c#
using (var vw = new VW.VowpalWabbit("-f rcv1.model"))
{
	VW.VowpalWabbitExample example = null;
        try
        {
            // 1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02
            using (var exampleBuilder = new VW.VowpalWabbitExampleBuilder(vw))
            {
                var ns = exampleBuilder.AddNamespace('f');
                var namespaceHash = vw.HashSpace("f");

                var featureHash = vw.HashFeature("13", namespaceHash);
                ns.AddFeature(featureHash, 8.5609287e-02f);

                featureHash = vw.HashFeature("24", namespaceHash);
                ns.AddFeature(featureHash, 3.4781646e-02f);

                featureHash = vw.HashFeature("69", namespaceHash);
                ns.AddFeature(featureHash, 4.6296168e-02f);

                exampleBuilder.Label = "1";

                // hand over of memory management
                example = exampleBuilder.CreateExample();
            }

            example.Learn();
        }
        finally
        {
            if (example != null)
            {
                example.Dispose();
            }
        }
}
``

## String based examples
Pro: no pitfalls when it comes to reproducibility/compatibility when used together with VW binary.  
Pro: supports affixes.  
Con: slowest variant due to string marshaling.  
Con: --affix is not supported, though easy to replicate in C#.

```c#
// make sure to properly dispose to free native memory 
using (var vw = new VW.VowpalWabbit("-f rcv1.model"))
{
	vw.Learn("1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02");
	// read more data ...

	var prediction = vw.Predict<VW.VowpalWabbitScalarPrediction>("0 |f 9:8.5609287e-02 14:2.9904654e-02 19:6.1031535e-02 20:2.1757640e-02");
	System.Console.WriteLine("Prediction: " + prediction.Value);
}
```