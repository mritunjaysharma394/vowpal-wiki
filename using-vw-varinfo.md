# Introduction

vw-varinfo is a small wrapper around vw which exposes all variables of a model in human readable form. The output includes the input variable names, including name-spaces where applicable, the vw hash value, the range [min, max] of the variable values in the training-set, the final model (regressor) weight, and the relative distance of each variable from the best constant prediction.

The wrapper is written in perl and can be found under _utl/vw-varinfo_ in the source tree.  _vw-varinfo_ calls _vw_ so vw should be installed somewhere in your _PATH_ for _vw-varinfo_ to work properly.

Here's a self-explanatory example output showing which foods affect increase vs decrease in daily weight during a diet:

    FeatureName                HashVal   MinVal   MaxVal    Weight   RelScore
    ^bread                      220390     0.00     2.00   +0.0984     55.36%
    ^icecream-sandwich           39873     0.00     1.00   +0.0951     53.44%
    ^snapple                    129594     0.00     2.00   +0.0867     48.61%
    ^trailmix                   215350     0.00     1.00   +0.0708     39.53%
    ^peanut                     187714     0.00     8.00   +0.0464     25.49%
    ^pizza                       32162     0.00     4.00   +0.0420     23.00%
    Constant                    116060     0.00     0.00   +0.0020      0.00%
    ^peas                       126345     0.00     1.00   +0.0002     -1.01%
    ^quinoa                     179283     0.00     1.00   +0.0002     -1.01%
    ^salmon                     215140     0.00     2.00   -0.1015    -59.42%
    ^chicken                    189058     0.00     1.00   -0.1133    -66.17%
    ^salad                      171971     0.00     2.00   -0.1256    -73.24%
    ^mayo                       187932     0.00     1.00   -0.1570    -91.28%
    ^oliveoil                    69559     0.00     1.00   -0.1570    -91.28%
    ^egg                           565     0.00     1.00   -0.1722   -100.00%

The example shows that based on the (simplistic) training set that was passed to vw-varinfo, eating egg and olive oil has the biggest negative correlation with weight-increase, while bread, icecream and sweetened drinks are the biggest enemies of weight loss. YMMV.

# Usage:

    vw-varinfo data.train

where data.train is a standard vw training-set.
Just like vw itself, you may call vw-varinfo without any arguments to get a brief usage message.

## more elaborate usage examples:

If you want to call vw with more arguments, simply pass them through to the training phase of vw like this:

    vw-varinfo --l1 0.0005 -c --passes 40 data.train

Another example. Say you want to find the strength of certain interactions between two groups of features with respect to the output label.  Assuming your data-set has two input-feature name-spaces starting with 'X' and 'Y', which separate your input features into two groups, you may run:

    vw-varinfo -q XY your_data_set

And vw-varinfo will output all the pairs of interactions between features in name-space X and the features in name-space Y, ordered by their relative effect.  You may add additional parameters to pass to vw training phase.

## Sanity check of vw-varinfo using a contrived example

Here's a contrived example showing how vw-varinfo performs on a perfect linear model.

Step 1) we write a script _generate-trainset.pl_ which loops 1000 times, in each loop iteration, it generates 5 random variables named 'a' through 'e' each with a random value in the interval [0 .. 1] and calculates the label y as:

    y = a + 2*b + 3*c + 4*d + 5*e

Obviously, 'e' is 5 times more "important" than 'a' in affecting the value of y.

Here's the _generate-trainset.pl_ script:

    #!/usr/bin/perl -w
    my $N = 1000;
    for ($i = 1; $i <= $N; $i++) {
        my $a = rand(1);
        my $b = rand(1);
        my $c = rand(1);
        my $d = rand(1);
        my $e = rand(1);
        my $y = $a + 2*$b + 3*$c + 4*$d + 5*$e;
        printf "%g | a:%g b:%g c:%g d:%g e:%g\n", $y, $a, $b, $c, $d, $e;
    }

Step 2) We run the script and save its output in a training-set:

    $ generate-trainset.pl > abcde.train

    
Step 3) Running vw-varinfo on the training-set we get:

    $ vw-varinfo  abcde.train
    FeatureName        HashVal   MinVal   MaxVal    Weight   RelScore
    ^e                  180798     0.00     1.00   +5.0000    100.00%
    ^d                  193030     0.00     1.00   +4.0000     80.00%
    ^c                  140873     0.00     1.00   +3.0000     60.00%
    ^b                  244212     0.00     1.00   +2.0000     40.00%
    ^a                   24414     0.00     1.00   +1.0000     20.00%
    Constant            116060     0.00     0.00   +0.0000      0.00%

which is exactly what was expected.

IOW: vowpal_wabbit perfectly figured out our formula, without knowing it in advance, by looking at the training data alone. QED.

## Exercise:

* modify the _generate-trainset.pl_ so that y is always larger or smaller by some constant value.
* how do you expect the result to be affected?
* run your modified script, run vw-varinfo on the newly generated train-set and verify your hypothesis.

## Credit & History:

`vw-varinfo` was written by Ariel Faigon, with help and guidance from John Langford.
It was the main tool used for a [little weight-loss project](https://github.com/arielf/weight-loss) in its very early days.

## Known issues:

`vw-varinfo` was written at about 2012, many options have been added to `vw` since. It is known not to work as-expected when some of the newer options are used, in particular, `--cb` and derivatives. Unfortunately, the code base is overly-complex and considered non-salvageable by its author.

There's a much cleaner, simpler and newer version of `vw-varinfo` in [here, called vw-varinfo2](https://github.com/arielf/weight-loss/blob/master/vw-varinfo2). This version is a rewrite in python, is more efficient and more general. It is almost completely agnostic to `vw` options, so it should be easier to maintain going forward, but it doesn't yet support multi-class.  If you want to enhance it, I (ariel) suggest that you start with this code-base.
