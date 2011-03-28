Input files for VW should have one example per line. Each example should be formatted as follows. 
    [Label] [Importance] [Tag] |Namespace Features |Namespace Features | ... |Namespace Features

where
    Namespace=String[:Value]

and
    Features=(String[:Value] )*

* `Label` is the real number that we are trying to predict for this example.  If the label is omitted, then no training will be performed with the corresponding example, although VW will still compute a prediction.
* `Importance` (importance weight) is a non-negative real number indicating the relative importance of this example over the others.  Omitting this gives a default importance of 1 to the example.
* `Tag` is a string that serves as an identifier for the example.  It is reported back when predictions are made.  It doesn't have to be unique.  The default value if it is not provided is the empty string.
* `Namespace` is an identifier of a source of information for the example optionally followed by a float (e.g., `Superfeatures:2.5`), which acts as a global scaling of all the values of the features in this namespace.  If value is omitted, the default is 1.
* `Features` is a sequence of whitespace separated strings, each of which is optionally followed by a float (e.g., `NumberOfLegs:4.0 HasStripes`).  Each string is a feature and the value is the feature value for that example. Omitting a feature means that its value is zero.  Including a feature but omitting its value means that its value is 1.
