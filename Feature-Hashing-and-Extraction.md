# Vowpal Wabbit Uses Hashing of Feature Names.

## Why Hash?

###(sparse) Dimension Reduction and fast feature lookups.
the default is hashing / projecting feature names to the machine architecture unsigned word using a variant of the murmurhash2 algorithm which then is XORed with (2^k)-1  (ie it is projected down to the first k lower order bits with the rest 0'd out). by default k=18 (ie 2^18 entries in the feature vector), with a max number of bits on 32-bit machines of k=29, and on a 64 bit machine, up to k=32). 

### how is it implemented 
for a consolidated model of the hashing code for feature string names, features with name spaces, and quadratic features over pairs of (name space, feature name) 
see [this gist](https://gist.github.com/2903178) for details. Note that this code as written is a model for the 32-bit implementation of the hashing.


## But I want to know the model weights for my input features, what can I do?
Well, by default, Vowpal Wabbit does not hash integer feature names (ie the feature name is written as a positive base 10 number). So if you wish to be able to unambiguously relate vowpal model weights to your original feature data, we recommend the following:

*  form your features (including any quadratic or other higher order ones) directly beforehand, so that vowpal just needs to work linearly over the features. Assign each feature a unique integer key. Also be sure to assign an integer for the constant feature. Store this feature <-> integer map for later usage. 
* write your data to a file in the VW format (the integer representation thereof!)
* run vw over  your data, and be sure to include both the --noconstant flag (so that vowpal does not include its own special constant feature), and also include the human readable flag `--readable_model filename.model` to get the output model weights in an easy to parse format.

A simpler way is now (July 9, 2012) supported. Use the utl/vw-varinfo script which in the source tree on your training-set file:

    vw-varscore data.train

The output will look like this (example):

    FeatureName        HashVal   MinVal   MaxVal    Weight   RelScore
    ^e                  180798     0.00     1.00   +5.0000    100.00%
    ^d                  193030     0.00     1.00   +4.0000     80.00%
    ^c                  140873     0.00     1.00   +3.0000     60.00%
    ^b                  244212     0.00     1.00   +2.0000     40.00%
    ^a                   24414     0.00     1.00   +1.0000     20.00%
    Constant            116060     0.00     0.00   +0.0000      0.00%


enjoy!



