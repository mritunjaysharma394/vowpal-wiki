Input files for VW should have one example per line. Each example should be formatted as follows. 
<pre>Label [Importance] [Tag] |Namespace Features |Namespace Features | ... |Namespace Features \n </pre>
Where 
<pre>Label</pre> is a real number that we are trying to predict.
<pre>Importance</pre> is a non-negative real number indicating the relative importance of this example over the others. In particular, omitting this give a default importance of 1 to that example. 
<pre>Tag</pre> is a string that serves as an identifier for the example. It is reported back when predictions are made. It doesn't have to be unique. The default value if it is not provided is the empty string.