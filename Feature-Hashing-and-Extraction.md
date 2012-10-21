# Vowpal Wabbit Uses Hashing of Feature Names.

## Why Hash?

###(sparse) Dimension Reduction and fast feature lookups.
the default is hashing / projecting feature names to the machine architecture unsigned word using a variant of the murmurhash v3 (32-bit only) algorithm which then is XORed with (2^k)-1  (ie it is projected down to the first k lower order bits with the rest 0'd out). by default k=18 (ie 2^18 entries in the feature vector), with a max number of bits on 32-bit machines of k=29, and on a 64 bit machine, up to k=32). 

### how is it implemented 
for a consolidated model of the hashing code for feature string names, features with name spaces, and quadratic features over pairs of (name space, feature name) 
see [this gist](https://gist.github.com/2903178) for details. Note that this code as written is a model for the 32-bit implementation of the hashing.


## But I want to know the model weights for my input features, what can I do?
Well, by default, Vowpal Wabbit does not hash integer feature names (ie the feature name is written as a positive base 10 number). So if you wish to be able to unambiguously relate vowpal model weights to your original feature data, we recommend the following:

*  form your features (including any quadratic or other higher order ones) directly beforehand, so that vowpal just needs to work linearly over the features. Assign each feature a unique integer key. Also be sure to assign an integer for the constant feature. Store this feature <-> integer map for later usage. 
* write your data to a file in the VW format (the integer representation thereof!)
* run vw over  your data, and be sure to include both the --noconstant flag (so that vowpal does not include its own special constant feature), and also include the human readable flag `--readable_model filename.model` to get the output model weights in an easy to parse format.

A simpler way is now (July 9, 2012) supported. Use the utl/vw-varinfo script which in the source tree on your training-set file:

    vw-varinfo mydata.train

The output will look like this (example):

    FeatureName        HashVal   MinVal   MaxVal    Weight   RelScore
    ^e                  180798     0.00     1.00   +5.0000    100.00%
    ^d                  193030     0.00     1.00   +4.0000     80.00%
    ^c                  140873     0.00     1.00   +3.0000     60.00%
    ^b                  244212     0.00     1.00   +2.0000     40.00%
    ^a                   24414     0.00     1.00   +1.0000     20.00%
    Constant            116060     0.00     0.00   +0.0000      0.00%


enjoy!

### Notes on the hashing algorithm

* VW has switched from jenkins hash to murmur hash (v2) in 2009.
* VW has switched from murmur hash v2 to murmur hash v3 in August 2012

hash function switching was driven by considering both better collision avoidance and faster run-time performance.

In the first case we improved both, in the second case collision avoidance was improved at the expense of a small (~3%) runtime degradation.

It is important to note that we are only using the 32-bit version of the hash

Collisions can be avoided by mapping your original features into natural numbers using a precomputed dictionary and using the numbers as feature names.  VW will use the integer as the value of the hash if the feature name is a natural number.  The command line `--hash [strings|all]` can force all feature names to be hashed.  The default is `--hash strings` which will hash feature names that look like strings but not those that look like integers.