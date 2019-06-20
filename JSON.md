The native C++ codebase can ingest JSON by passing _--json_

The following JSON format can be ingested into VW:

* Top-level properties are considered features for the default namespace.
* Top-level properties of type object or array are considered namespaces.
* Features are JSON strings, integer, float, boolean, arrays of integers and/or floats.
* Top-level properties starting with _ are ignored, except if they match a special property (e.g. "_label", "_multi", "_text", "_tag").
* Labels can be passed using top-level "_label" property. This is also supported for multiline examples, but the label needs to be part of one of the multiline examples.
 * If the JSON value is either a string, integer or float is converted to a string and passed directly to VW label parser.
 * If the JSON value is an object, the first property needs to match one of the JSON properties of [SimpleLabel](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/cli/vw_label.h#L169) or [ContextualBanditLabel](https://github.com/JohnLangford/vowpal_wabbit/blob/master/cs/cli/vw_label.h#L37).  
* Tags can be applied by using the "_tag" property.
* Special text handling through "_text": properties named "_text" are processed using string splitting and not string escaping (see sample below).
* Multi-line examples as used by contextual bandits are specified by using the "_multi" property. Each entry itself is an example as described above and can optionally contain a label and/or tag. The top-level properties are used for the optional shared example.

The C# layer can ingest
* JSON strings 
* [JSON.NET's](http://www.newtonsoft.com/json) JsonReader
* C# objects serializable to the above JSON format using JSON.NET serializing rules. Thus JsonProperty annotations are inspected and so on. This is particularly useful if one needs to score a given object, then serialize it JSON and train from the JSON serialization as it circumvents the de-serialization for the scoring part. 

# Examples
<table><tr><th>JSON</th><th>VW String</th></tr>
<tr><td> 
  <pre lang="json"> 
{
 "f1":25,"f2":true,
 "_aux":"some ignored info"
} </pre> </td><td>
  <pre> | f1:25 f2</pre>
</td></tr>
<tr><td> 
  <pre lang="json"> 
{
 "ns1":{"location":"New York"},
 "f2":[1,0.2,3]
} </pre> </td><td>
  <pre> |ns1 locationNew_York | :1 :.2 :.3</pre>
</td></tr>
<tr><td> 
  <pre lang="json">
{
 "ns1":{"location":"New York"},
 "ns2":{"f2":3.4},"_label":1
} </pre> </td><td>
  <pre>1 |ns1 locationNew_York |ns2 f2:3.4</pre>
</td></tr>

<tr><td> 
  <pre lang="json"> 
{
 "ns1":{"location":"New York", "f2":3.4},
 "_label":{"Label":2,"Weight":0.3}
} </pre> </td><td>
  <pre>2 0.3 |ns1 locationNew_York f2:3.4</pre>
</td></tr>

<tr><td> 
  <pre lang="json"> 
{
 "x":2,
 "_text":"elections US iowa"
} </pre> </td><td>
  <pre>| x:2 elections US iowa</pre>
</td></tr>

<tr><td> 
  <pre lang="json"> 
{
 "UserAge":15,
 "_multi":[
   {"_text":"elections maine", "Source":"TV"},
   {"Source":"www", "topic":4, "_label":"2:3:.3"}
 ]
} </pre> </td><td>
  <pre>
shared | UserAge:15
| elections maine SourceTV
2:3:.3 | Sourcewww topic:4
</pre>
</td></tr>
</table>