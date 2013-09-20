A new [[version 7.0 tutorial|v7.0_tutorial.pdf]] is available. It covers the basics and most common options, how to use VW and the data format for different types of problems, such as Binary Classification, Regression, Multiclass Classification, Cost-Sensitive Multiclass Classification, "Offline" Contextual Bandit and Sequence Predictions. Many more advanced options in terms of flags and the data format are not covered. You can refer to previous tutorials for these more advanced details.

The [[version 6.1 tutorial|v6.1_tutorial.pdf]] and various pieces below covers some topics not covered in the version 7 tutorial, as most of these haven't change in the latest version:

* The [[introduction|v6.1_tutorial.pdf]].
* Description and use of [[L-BFGS|L-BFGS.pdf]].
* [[Cluster parallel learning|Cluster_parallel.pdf]].
* [[Active Learning|active.pdf]] (v5.0 presentation, but little changed)
* [[Latent Dirichlet Allocation|lda.pdf]] (v5.0 presentation, little changed) See also [[Latent Dirichlet Allocation]]
* <a href="http://zinkov.com/posts/2013-08-13-vowpal-tutorial/">Vowpal Wabbit tutorial for the Uninitiated</a> by Rob Zinkov

### Older stuff
The [[version 5.1 tutorial|v5.1_tutorial.pdf]] with a <a href="http://talks.eharmony.com/video/31936897">video</a>. 

<a href="http://videolectures.net/nipsworkshops2010_langford_vow/">Version 5.0 Videolecture</a>.

* The [[main|main.pdf]] piece (v5.0)
* The [[importance weight invariant|iw.pdf]] update rule. (covered in 6.1 intro)

### A Step by step introduction

The first step is downloading a version of VW.  We'll use the <a href="https://github.com/JohnLangford/vowpal_wabbit">github master</a>, which should generally work as squashing bugs is first priority.  A alternate choice is to use an <a href="https://github.com/JohnLangford/vowpal_wabbit/archives/master">existing version</a>.  The examples below are command-line, inside a terminal, prompts are dropped for clarity.

    git clone git://github.com/JohnLangford/vowpal_wabbit.git

Now we compile:

    cd vowpal_wabbit
    make

This should "just work", at least on Linux and OS-X, and plausibly on any POSIX platform.  If it fails, you most likely need to install the boost program options headers and library.

Boost installation on Debian/Linux distributions:

    sudo apt-get install libboost-program-options-dev

Boost installation on Mac OS-X is a little bit more involved:

  1. Download the source from [[http://sourceforge.net/projects/boost/files/boost/1.50.0/]]
  2. Execute the shell script with `sudo ./bootstrap.sh`
  3. Next run the command `sudo ./bjam --layout=tagged install`
  4. You should be good to go

Now test the newly built vw executable:

    make test

Everything should pass.  If you see: 

    minor (<0.001) precision differences ignored

That's ok.  One of the things we do for speed is use the `-ffast-math` option which implies that floating point arithmetic does not round exactly the same way on all platforms.

### A first data-set

Now, let's create a data-set.  Suppose we want to predict whether a house will require a new roof in the next 10 years.

    echo "0 | price:.23 sqft:.25 age:.05 2006
    1 2 'second_house | price:.18 sqft:.15 age:.35 1976
    0 1 0.5 'third_house | price:.53 sqft:.32 age:.87 1924" > house_dataset

There is quite a bit going on here.  The first thing is a label, `0`, corresponding to no roof replacement.  The bar `|` separates label from features.   The features are `price`, `sqft`, `age`, and `2006`.  The first 3 features use an index derived from a hash function while the last feature uses index 2006 directly and has a default value of 1.

The next example, on the next line, is similar, but the label information is more complex.  The `1` is the label indicating a roof-replacement is required.  The `2` is an importance weight which implies that this example counts twice.  Importance weights come up in many settings.  A missing importance weight defaults to 1. `'second_house` is the tag, it is used elsewhere to identify the example.

The 3rd example is straightforward, except there is an additional number: `0.5` following the weight, in the label information.  This is an initial prediction.  Sometimes you have multiple interacting learning systems and want to be able to predict an offset rather than an absolute value.

Next, we learn:

    ./vw house_dataset

### VWs diagnostic information

There is a burble of diagnostic information which you can turn off with `--quiet`.  However, it's worthwhile to first understand it.

    using no cache

This says you are not using a cache.  If you use multiple passes with `--passes`, you would need to also pass `-c` so vw can cache the data in a faster to handle format (passes > 1 should be much faster). By default, the cache file name will be the data-set file with `.cache` appended. In this case: `house_dataset.cache`.  You may also override the default cache file name by passing: `--cache_file housing.cache`.  A cache file can greatly speed up training when you run many experiments (with different options) on the same data-set even if each experiment is only a single pass. So if you want to experiment with the same data-set over and over, it is highly recommended to pass `-c` every time you train.  If the cache exists and is newer than the data-set, it will be used, if it doesn't exist, it'll be created the first time `-c` is used.

    Reading from house_dataset

There are many different ways to input data to VW.  Here we're just using a simple text file and vw tells us the source of the data.  Alternative sources include cache files (from previous VW runs), stdin, or a tcp socket.

    num sources = 1

There is only one input file in our example.  But you can specify multiple files.

    Num weight bits = 18

Only 18 bits of the hash function will be used.  That's much more than necessary for this example.  You could adjust the number of bits using `-b bits`

    learning rate = 10

The default learning rate is 10.  This is too aggressive when there is noise, but it's a good default because when there is noise you'll need a larger data-set and/or multiple passes to predict well.  On these larger data-sets, our learning rate will by default decay towards 0 as we run through examples.  You can adjust with `-l rate`.

    initial_t = 1

Learning rates should often decay over time, and this specifies the initial time.  You can adjust with `--initial_t time`, although this is rarely necessary these days.

    power_t = 0.5

This specifies the power on the learning rate decay.  You can adjust this `--power_t p` where _p_ is in the range [0,1].  0 means the learning rate does not decay, which can be helpful when state tracking, while 1 is very aggressive, but plausibly optimal for IID data-sets.  0.5 is a minimax optimal choice. A different way of stating this is: stationary data-sets where the fundamental relation between the input features and target label are not changing over time, should benefit from a high (close to 1.0) `--power_t` while learning against changing conditions, like learning against an adversary who continuously changes the rules-of-the-game, would benefit from low (close to 0) `--power_t` so the learner can react quickly to these changing conditions. For many problems, 0.5, which is the default, seems to work best.

Next, there is a bunch of header information.  VW is going to print out some live diagnostic information. 

The first column, `average loss` computes the <a href="http://hunch.net/~jl/projects/prediction_bounds/progressive_validation/coltfinal.pdf">progressive validation</a> loss.  The critical thing to understand here is that progressive validation loss deviates like a test set, and hence is a reliable indicator of success on the first pass over any data-set.

`since last` is the progressive validation loss since the last printout.

`example counter` tells you which example is printed out.  In this case, it's example 2.

`example weight` tells you the sum of the importance weights of examples seen so far.  In this case it's 3, because the second example has an importance weight of 2.  

`current label` tells you the label of the second example.

`current predict` tells you the prediction (before training) on the current example.

`current features` tells you how many features the current example has.  This is great for debugging, and you'll note that we have 5 features when you expect 4.  This happens, because VW has a default constant feature which is always added in.  Use the `--noconstant` command-line option to turn it off.

VW prints a new line with an exponential backoff.  This is very handy, because often you can debug some problem before the learning algorithm finishes going through a data-set.

At the end, some more straightforward totals are printed.  The only mysterious one is:
`best constant` and `best constant's loss`  These really only work if you are using squared loss, which is the default.  They compute the best constant's predictor and the loss of the best constant predictor.  If `average loss` is not better than `best constant's loss`, something is wrong.  In this case, we have too few examples to generalize.

If we want to overfit like mad, we can simply use:

    ./vw house_dataset -l 10 -c --passes 25 --holdout_off

The progress section of the output is:

    average    since         example     example  current  current  current
    loss       last          counter      weight    label  predict features
    0.666667   0.666667            2         3.0   1.0000   0.0000        5
    0.558318   0.477056            5         7.0   1.0000   0.3314        5
    0.475057   0.329351            8        11.0   1.0000   0.6017        5
    0.239335   0.023256           17        23.0   1.0000   0.9784        5
    0.125118   0.000023           33        44.0   0.0000   0.0001        5
    0.063278   0.000000           65        87.0   1.0000   1.0000        5

You'll notice that by example 65 (25 passes over 3 examples result in 75 examples), the `since last` column has dropped to 0, implying that by looking at the same (3 lines of) data 25 times we have reached a perfect predictor. This is unsurprising with 3 examples having 5 features each.  The reason we have to add `--holdout_off` (new option in version 7.3, added August 2013) is that when running multiple-passes, vw automatically switches to 'over-fit avoidance' mode by holding-out 10% of (the period "one in 10" can be changed using `--holdout_period period`) the data and evaluating performance on the held-out data instead of using the online-training progressive loss.

### Saving your model (a.k.a. regressor) into a file

By default vw learns the weights of the features and keeps them in a memory vector. If you want to save the final regressor weights into a file, add **-f _filename_**:

    ./vw house_dataset -l 10 -c --passes 25 --holdout_off -f house.model

### Getting predictions

We want to make predictions of course:

    ./vw house_dataset -p /dev/stdout --quiet

The output is:

    0.000000
    0.000000 second_house
    1.000000 third_house

The 1st output line `0.000000` is for the 1st example which has an empty tag.

The 2nd output `0.000000 second_house` is for the 2nd example.  You'll notice the tag appears here. This is the primary use of the tag: mapping predictions to the examples they belong to.

The 3rd output `1.000000 third_house` is for the 3rd example.  Clearly, some learning happened, because the prediction is now `1.000000` while the initial prediction was set to `0.5`.

Note that in this last example, we predicted _while we learned_.  The model was being incrementally built in memory as vw went over the examples.

Alternatively, and more commonly, we would first learn and save the model into a file. Later we would predict using the saved model.

You may load a initial model to memory by adding `-i house.model`.  You may also want to specify `-t` which stands for "test-only" (do no learning):

    ./vw -i house.model -t house_dataset -p /dev/stdout --quiet

Which would output:

    0.000000
    1.000000 second_house
    0.000000 third_house

Obviously the results are different this time, because in the first prediction example, we learned as we went, and made only one pass over the data, whereas in the 2nd example we first loaded an over-fitted (25 pass) model and used our data-set `house_dataset` with `-t` (testing only mode).  In real prediction settings, one should use a different data-set for testing vs training.

### Auditing

When developing a new ML application, it's very helpful to debug.  VW can help a great deal with this using the `--audit` option.

    ./vw house_dataset --audit --quiet

Every example uses two lines.  The first line has the prediction, and the second line has one entry per feature.  Looking at the first feature, we see:

    ^price:43641:0.23:0@0.25

The `^price` is the original feature.  If you use a namespace, it appears before `^`.  Namespaces are an advanced feature which allows you to group features and operate them in the core of VW with `-q` and `--ignore`.

`43641` is the index of the feature, computed by a hash function on the feature name.

`0.23` is the value of the feature.

`0` is the value of the feature's weight.

`@0.25` represents the sum of gradients squared for that feature when you are using per-feature adaptive learning rates.

Examining further, you'll notice that the feature `2006` uses the index 2006.  This means that you can freely use the hashing or pre-compute indices as is common in for other machine learning programs.

The advantage of using unique integer-based feature-names is that they are guaranteed not to collide after hashing.  The advantage of free-text (non integer) feature names is readability and self-documentation. Since only `:`, `|`, and _spaces_ are special to the vw parser, you can give features extremely readable names like:  `height>2  value_in_range[1..5]  color=red`  and so on.  Feature-names may even start with a digit, e.g.: `1st-guess:0.5    2nd-guess:3`  etc.

### What's next?

The above only scratches the surface of VW.  You can learn with other loss functions, with other optimizers, with other representations, with clusters of 1000s of machines, and even do ridiculously fast active learning.  Your primary resources for understanding these are:

1. The presented tutorials at the top of the page.
2. The <a href="https://github.com/JohnLangford/vowpal_wabbit/wiki/Examples">examples</a>.
3. The <a href="https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments">commandline specification</a>.
4. The <a href="http://tech.groups.yahoo.com/group/vowpal_wabbit/">mailing list</a>.
5. You. If something isn't covered adequately, ask questions and consider creating an example or a test case.