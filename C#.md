This is a tutorial for the new Vowpal Wabbit C# binding. Here's a list of the major features:

* Very efficient serialization from managed to native space using runtime compiled serializers.
* Declarative specification of example data structure.
* Thread-saftey through object pooling and shared models.
* Example level caching (prediction only).
* Improved memory management.

The C# binding is structured in layers and enables multiple use cases ordered by the level of abstraction:

1. User defined data types: use [VW.VowpalWabbit<TUserType>] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/VowpalWabbit.cs)
2. Generic data structures (e.g. records consisting of key/value/type tuples): use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h) and [VW.VowpalWabbitNamespaceBuilder] (https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)
3. String based examples: use [VW.VowpalWabbit](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vw_clr/vw_clr.h)

Through out the samples the following dataset from [[Rcv1-example]] is used:

<pre>
1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02 85:6.1853945e-02 ... 
0 |f 9:8.5609287e-02 14:2.9904654e-02 19:6.1031535e-02 20:2.1757640e-02 ... 
...
</pre>

# Generic data structures
Pro: most performant variant
Pro: provides maximum flexibility with feature representation.
Con: results might not be reproducible using VW binary as it allows for feature representation not exposed through the string format. 

# String based examples
Pro: no pitfalls when it comes to reproducibility/compatibility when used together with VW binary.
Pro: supports affixes
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