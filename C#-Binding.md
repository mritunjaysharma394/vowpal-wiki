*** The latest version of the C# Binding can be found in https://github.com/eisber/vowpal_wabbit **

This is a tutorial for the Vowpal Wabbit C# binding. Here's a list of major features:

* Very efficient serialization from managed to native space using runtime compilation.
* Declarative specification of example data structure.
* Thread-safety through object pooling and shared models.
* Example level caching (prediction only).
* Improved memory management.

The binding exposes three different options to interact with native Vowpal Wabbit, each having pros and cons:

1. User defined data types: use [VW.VowpalWabbit\<TUserType\>] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/VowpalWabbit.cs)
2. Generic data structures (e.g. records consisting of key/value/type tuples): use [VW.VowpalWabbit\<TUserType\>](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/serializer/VowpalWabbitSerializerFactory.cs)
3. String based examples: use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vowpalwabbit.h)

# Usage

Install the [Vowpal Wabbit NuGet Package](https://www.nuget.org/packages/VowpalWabbit/) using

```PowerShell
Install-Package VowpalWabbit
```

The nuget includes:

* C++ part of vowpal wabbit compiled for Windows x64 Release
* C++/CLI wrapper
* C# wrapper supporting declarative data to feature conversion
* PDB debug symbols (for Windows newbies, PDB here is a program database file, not the Python debugger)
* Source files
* IntelliSense documentation
* zlib native dll 

Note: I'm aware of symbolsource.org, but due to some PDB references to system headers such as undname.h, I was unable to create a "symbolsource.org" valid -symbols.nupkg. 

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
very performant | one-time overhead of serializer compilation
declarative data to feature conversion | 

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

The serializer follows an opt-in model, thus only properties annotated using [\[Feature\]](https://github.com/eisber/vowpal_wabbit/blob/master/cs/Serializer/Attributes/FeatureAttribute.cs) are transformed into Vowpal Wabbit features. The [\[Feature\]](https://github.com/eisber/vowpal_wabbit/blob/master/cs/Serializer/Attributes/FeatureAttribute.cs) attribute supports the following properties:

Property | Description | Default
-------- | ----------- | -------
FeatureGroup | it's the first character of the namespace in the string format | Space
Namespace | concatenated with the FeatureGroup | 0 = hash(Namespace)
Name | name of the feature (e.g. 13, 24, 69 from the example above) | property name
Enumerize | if true, features will be converted to string and then hashed. e.g. VW line format: Age_15 (Enumerize=true), Age:15 (Enumerize=false) | false
Order | feature serialization order. Useful for comparison with VW command line version | 0
StringProcessing | String features are either escaped (spaces are replaced with underscores) or split by space producing individual features. | Split
AddAnchor | use with dense features and --interact to mark the beginning of a set of dense features | false
Dictify | when generating Vowpal Wabbit string formatted examples, this will replace the annotated feature with a surrogate. The serialized feature will be stored and checked against a dictionary passed to VowpalWabbitSerializer.SerializeToString(). | false

Furthermore the serializer will recursively traverse all properties of the supplied example type on the search for more [\[Feature\]](https://github.com/eisber/vowpal_wabbit/blob/master/cs/Serializer/Attributes/FeatureAttribute.cs) attributed properties (Note: recursive data structures are not supported). Feature groups, namespaces and dictify are inherited from parent properties and can be overridden for sub trees. Finally all annotated properties are put into the corresponding namespaces.

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

// ...

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
using (var vw = new VW.VowpalWabbit<Row>("-f rcv1.model"))
{
	var userExample = new Row { /* ... */ };
        	
        vw.Learn(userExample, new SimpleLabel { / *... */ });

        var prediction = vw.Predict(userExample, VowpalWabbitPredictionType.Scalar);
}
```

* Serializers are globally cached per type (read: static variable).  I.e., there's a static dictionary from user-defined types to serializers.
* Native example memory is cached using a pool per VW.VowpalWabbit instance. Each Learn/Predict call will either get memory from the pool or allocate new memory.

The serialization infrastructure is extensible by providing type based custom featurizers (VowpalWabbitSettings.CustomFeaturizer). Consider the following example:

```c#
    public class CustomClass
    {
        public int X { get; set; }
    }

    public class MyContext
    {
        [Feature]
        public CustomClass Feature { get; set; }
    }

    public class CustomFeaturizer
    {
        public void MarshalFeature(VowpalWabbitMarshalContext context, Namespace ns, Feature feature, CustomClass value)
        {
            var featureHash = context.VW.HashFeature("prefix"+ feature.Name, ns.NamespaceHash);
            context.NamespaceBuilder.AddFeature(featureHash, value.X);

            context.AppendStringExample(feature.Dictify, " prefix{0}:{1}", feature.Name, value.X);
        }
    }


            var context = new MyContext() { Feature = new CustomClass() { X = 5 }};
            using (var vw = new VowpalWabbit<MyContext>(new VowpalWabbitSettings(customFeaturizer: new List<Type> { typeof(CustomFeaturizer) })))
            {
                vw.Learn(context);
            }
```

In the above example features of type CustomClass will be marshalled using CustomFeaturizer. The serializer infrastructure looks for methods of the form __public void MarshalFeature(VowpalWabbitMarshalContext context, Namespace ns, Feature feature, CustomClass value)__ among others. A good reference is [VowpalWabbitDefaultMarshaller](https://github.com/eisber/vowpal_wabbit/blob/master/cs/Serializer/VowpalWabbitDefaultMarshaller.cs) which is internally added to the same featurizer list. Custom featurizers are given priority over the default marshaller.
Serializing data to string format (see VowpalWabbitMarshalContext.AppendStringExample) is optional when working through the native interface only, but considered good practice to ease debugging.

## Generic data structures
Pro | Cons
--- | ----
very performant | results might not be reproducible using VW binary as it allows for feature representation not expressible through the string format 
provides maximum flexibility with feature representation | System.Linq.Expression based - a bit harder use
suited for generic data structures (e.g. records, data table, ...) | --affix is not supported, though easy to replicate in C#

Let me point out that using VowpalWabbitDefaultMarshaller is another option. The biggest upside is that it optionally generates the corresponding VW string features, but maybe less flexible than the direct approach below. 
* * overriding which feature resolution (VowpalWabbitSettings.AllFeatures)

```c#
using (var vw = new VW.VowpalWabbit("-f rcv1.model"))
{
   VW.VowpalWabbitExample example = null;
   try
   {
      // 1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02
      using (var exampleBuilder = new VW.VowpalWabbitExampleBuilder(vw))
      {
         // important to dispose the namespace builder at the end, as data is only added to the example
         // if there is any feature added to the namespace
         using (var ns = exampleBuilder.AddNamespace('f'))
         {
            var namespaceHash = vw.HashSpace("f");

            var featureHash = vw.HashFeature("13", namespaceHash);
            ns.AddFeature(featureHash, 8.5609287e-02f);
        
            featureHash = vw.HashFeature("24", namespaceHash);
            ns.AddFeature(featureHash, 3.4781646e-02f);
        
            featureHash = vw.HashFeature("69", namespaceHash);
            ns.AddFeature(featureHash, 4.6296168e-02f);
	 }

         exampleBuilder.ParseLabel("1");
        
         // hand over of memory management
         example = exampleBuilder.CreateExample();
      }

      vw.Learn(example);
   }
   finally
   {
      if (example != null)
      {
         example.Dispose();
      }
   }
}
```

## String based examples
Pro | Cons
--- | ----
no pitfalls when it comes to reproducibility/compatibility when used together with VW binary | slowest variant due to string marshaling  (and character encoding differences between the C# and C++ worlds)
supports affixes |

```c#
using (var vw = new VW.VowpalWabbit("-f rcv1.model"))
{
	vw.Learn("1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02");
	// read more data ...

	var prediction = vw.Predict<VW.VowpalWabbitScalarPrediction>("|f 9:8.5609287e-02 14:2.9904654e-02 19:6.1031535e-02 20:2.1757640e-02");
	System.Console.WriteLine("Prediction: " + prediction.Value);
}
```

# Multi-threading
VW.VowpalWabbit are not thread-safe, but by using object pools and shared models we can enable multi-thread scenarios without multiplying the memory requirements by the number of threads.

Consider the following excerpt from [TestSharedModel Unit Test](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs_unittest/TestCbAdf.cs)

```c#
var vwModel = new VowpalWabbitModel("-t -i m1.model");
using (var pool = new VowpalWabbitThreadedPrediction<Row>(vwModel))
{
     using (var vw = pool.GetOrCreate())
     {
          vw.Value.Predict(example);
     }

     pool.UpdateModel(new VowpalWabbitModel("-t -i m2.model"));
}
```

vwModel is the shared model. Each call to vwPool.Get() will either get a new instance spawned of the shared model or re-use an existing.  
A very common scenario when scoring is to rollout updates of new models. The [ObjectPool](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/ObjectPool.cs) class allows safe updating of the factory and proper disposal. After the call to vwPool.UpdateFactory(), vwPool.Get() will only return instances spawned of the new shared model (newVwModel). Not-in-use VowpalWabbit instances are disposed as part of UpdateFactory(). VowpalWabbit instances currently in-use are diposed upon return to the pool (PooledObject.Dispose). 

# Example level caching
To improve performance especially in scenarios using action dependent features, examples can be cached on a per VowpalWabbit instance base. To enable example level cache simply annotate the type using the [\[Cachable\]](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/Serializer/Attributes/CacheableAttribute.cs) attribute. This can **only** be used for **predictions** as labels cannot be updated once an example is created.
The cache size can be configured using [VowpalWabbitSerializerSettings](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/Serializer/VowpalWabbitSerializerSettings.cs).

It's considered best practice to use the same annotated user types at training and scoring time. As example level caching is only supported for predictions, one must disable caching at training time using

```c#
new VowpalWabbit("", new VowpalWabbitSerializerSettings { EnableExampleCaching = false })
```