This is a tutorial for the new Vowpal Wabbit C# binding. Here's a list of major features:

* Very efficient serialization from managed to native space using runtime compilation.
* Declarative specification of example data structure.
* Thread-saftey through object pooling and shared models.
* Example level caching (prediction only).
* Improved memory management.

The binding exposes three different options to interact with native vowpal wabbit, each having pros and cons:

1. User defined data types: use [VW.VowpalWabbit<TUserType>] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/VowpalWabbit.cs)
2. Generic data structures (e.g. records consisting of key/value/type tuples): use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h) and [VW.VowpalWabbitNamespaceBuilder] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)
3. String based examples: use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)

# Usage

Install the [Vowpal Wabbit NuGet Package](https://www.nuget.org/packages/VowpalWabbit/) using

```PowerShell
Install-Package VowpalWabbit
```

The nuget includes:

* C++ part of vowpal wabbit compiled for Windows x64 Release
* C++/CLI wrapper
* C# wrapper supporting declarative data to feature conversion
* PDB debug symbols
* Source
* IntelliSense documentation

Note: I'm aware of symbolsource.org, but due to some PDB reference to system headers such as undname.h, I was unable to upload a -symbols.nupkg nuget. 

# Examples
Through out the examples the following dataset from [[Rcv1-example]] is used:

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

* FeatureGroup: it's the first character of the namespace in the string format. Default: 0
* Namespace: concatenated with the FeatureGroup. Default: hash() = 0
* Name: name of the feature (e.g. 13, 24, 69 from the example above). Default: property name.
* Enumerize: if true, features will be converted to string and then hashed. e.g. VW line format: Age_15 (Enumerize=true), Age:15 (Enumerize=false). Default: false. 
* Order: feature serialization order. Useful for comparison with VW command line version.

Furthermore the serializer will recursively traverse all properties of the supplied example type on the search for more \[Feature\] attributed properties (recursive data structures are not supported). Feature groups and namespaces are inherited from parent properties, but can be overridden. Finally all properties are flattened and put into the corresponding namespace.

```c#
using VW.Serializer.Attributes;

public class ParentRow
{
	[Feature(FeatureGroup = 'f')]
        public CommonFeatures UserFeatures { get; set; }

        [Feature(FeatureGroup = 'f')]
        public String Country { get; set; }

        [Feature(FeatureGroup = 'g', Enumerize=true)]
        public int Age { get; set; }
}

public class CommonFeatures
{
        [Feature]
        public int A { get; set; }

        [Feature(FeatureGroup = 'g', Name="Beta")]
        public float B { get; set; }
}
```

```c#
var row = new ParentRow
{
	UserFeatures = new CommonFeatures
	{
		A = 2,
		B = 3.1f
	},
	Country = "Austria",
	Age = 25
};
```

The vowpal wabbit string equivalent of the above instance is

```
|f A:2 Country:Austria |g Beta:3.1 Age_25
```

```c#
// make sure to properly dispose to free native memory 
using (var vw = new VW.VowpalWabbit<Row>("-f rcv1.model"))
{
	var userExample = new Row { /* ... */ };
	
        using (var vwExample = vw.ReadExample(userExample))
        {
        	vwExample.Learn();
        }
}
```

Serializers are globally cached per type (read static variable). Native example memory is cached using a pool per VW.VowpalWabbit instance. Each ReadExample call will either get memory from the pool or allocate new memory. Disposing VowpalWabbitExample returns the native memory to the pool. Thus if you loop over many examples and dispose them immediately the pool size will be equal to 1.

## Generic data structures
Pro | Cons
--- | ----
most performant variant | results might not be reproducible using VW binary as it allows for feature representation not expressible through the string format 
provides maximum flexibility with feature representation | verbose
suited for generic data structures (e.g. records, data table, ...) | --affix is not supported, though easy to replicate in C#

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
Pro | Cons
--- | ----
no pitfalls when it comes to reproducibility/compatibility when used together with VW binary | slowest variant due to string marshaling  
supports affixes |

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