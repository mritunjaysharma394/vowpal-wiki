# Input format

VW supports 2 input formats: plain text (this page) and [JSON](JSON).
The raw (plain text) input data for VW should have one example per line.  Each example should be formatted as follows. 

    [Label] [Importance] [Base] [Tag]|Namespace Features |Namespace Features ... |Namespace Features

where

    Namespace=String[:Value]

and

    Features=(String[:Value] )*

* `Label` is the real number that we are trying to predict for this example.  If the label is omitted, then no training will be performed with the corresponding example, although VW will still compute a prediction.
* `Importance` (importance weight) is a non-negative real number indicating the relative importance of this example over the others.  Omitting this gives a default importance of 1 to the example.
* `Base` is used for residual regression.  It is added to the prediction before computing an update.  The default value is 0.
* `Tag` is a string that serves as an identifier for the example.  It is reported back when predictions are made.  It doesn't have to be unique.  The default value if it is not provided is the empty string. If you provide a tag without a weight you need to disambiguate: either make the tag touch the `|` (no trailing spaces) or mark it with a leading single-quote `'`. If you don't provide a tag, you need to have  a space before the `|`.
* `Namespace` is an identifier of a source of information for the example optionally followed by a float (e.g., `MetricFeatures:3.28`), which acts as a global scaling of all the values of the features in this namespace.  If value is omitted, the default is 1.  It is important that the namespace not have a space between the separator `|` as otherwise it is interpreted as a feature.
* `Features` is a sequence of whitespace separated strings, each of which is optionally followed by a float (e.g., `NumberOfLegs:4.0 HasStripes`).  Each string is a feature and the value is the feature value for that example. Omitting a feature means that its value is zero.  Including a feature but omitting its value means that its value is 1.

Currently, the only characters that can't be used in feature or namespace names are vertical bar, colon, space, and newline.

## Example

    1 1.0 |MetricFeatures:3.28 height:1.5 length:2.0 |Says black with white stripes |OtherFeatures NumberOfLegs:4.0 HasStripes

## Example with a tag

    1 1.0 zebra|MetricFeatures:3.28 height:1.5 length:2.0 |Says black with white stripes |OtherFeatures NumberOfLegs:4.0 HasStripes


## Notes
When using logistic or hinge loss, the labels need to be from the set {+1,-1}  (documented in the V6.1 tutorial slide deck, but not elsewhere)

The spacing around the `|` characters is important and significant:
* Around the 1st `|` if there's no space preceding it, the string that touches the `|` is considered a tag (id of example)

After any `|`
* If there's a space, the next non-space token is considered a regular feature name
* If there's no space, the next non-space token is considered a name-space

name-spaces are considered as feature name prefixes, they are prepended to all feature names in the name-space

## multi-class algorithm formats

Some of the vw algorithms support multiple classes.  For these algorithms the input format is expanded to support multiple-classes per example (input line).

### weighted-all-pairs & cost-sensitive-one-against-all format

The label information for the weighted-all-pairs algorithm (--wap) and the cost-sensitive-one-against-all (--csoaa) algorithm are the same.  This format is a sparse specification of costs per label (see csoaa.cc:parse_label() in the source tree for reference).

Here's an example:

    echo "1:0 2:3 3:1.5 4:1 |f input features come here" | vw --csoaa 4

Preceding the 1st `|` char we have 4 classes: 1, 2, 3, 4  each of them has a cost (the number after the colon).  It is important to specify the number of classes as an argument to vw (--csoaa 4) and have class labels in the range [1,N] in the input (N=4 in this example).  Since the representation is sparse, there's no need to have all labels in all lines.

## On-demand model saving
This feature is useful especially in the daemon mode, where you can decide in any moment of the training that you want to save the current model in arbitrary file, using a dummy example whose Tag starts with _save_ and optionally specifies also the filename (no label or features are needed in this dummy example), e.g.:

     vw --daemon --port 9999
     cat data1.vw | nc localhost:9999
     echo save_/tmp/my1.model | nc localhost:9999
     cat data2.vw | nc localhost:9999
     echo save_/tmp/my1and2.model | nc localhost:9999

## Format validation

You can check that vw is correctly parsing your input by pasting a few lines into the [VW validator](http://hunch.net/~vw/validate.html).

## LibSVM format
[LibSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvm/) uses a simpler format than VW, which can be easily converted to VW format just by adding a pipe symbol between the label and the features.

    perl -pe 's/\s/ | /' data.libsvm | vw -f model

For other formats (csv) and preprocessing see e.g. [Phraug](https://github.com/zygmuntz/phraug2).
