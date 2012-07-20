The [[version 6.1 tutorial|v6.1_tutorial.pdf]].  This the most uptodate version,  Various pieces are covered separately, including:

* The [[introduction|v6.1_tutorial.pdf]].
* Description and use of [[L-BFGS|L-BFGS.pdf]].
* [[Cluster parallel learning|Cluster_parallel.pdf]].
* [[Active Learning|active.pdf]] (v5.0 presentation, but little changed)
* [[Latent Dirichlet Allocation|lda.pdf]] (v5.0 presentation, little changed) See also [[Latent Dirichlet Allocation]]

### Older stuff
The [[version 5.1 tutorial|v5.1_tutorial.pdf]] with a <a href="http://talks.eharmony.com/video/31936897">video</a>. 

<a href="http://videolectures.net/nipsworkshops2010_langford_vow/">Version 5.0 Videolecture</a>.

* The [[main|main.pdf]] piece (v5.0)
* The [[importance weight invariant|iw.pdf]] update rule. (covered in 6.1 intro)

### A Step by step introduction

The first step is downloading a version of VW.  We'll use the <a href="https://github.com/JohnLangford/vowpal_wabbit">github master</a>, which should generally work as squashing bugs is first priority.  A alternate choice is to use an <a href="https://github.com/JohnLangford/vowpal_wabbit/archives/master">existing version</a>.

&gt;  **git clone git://github.com/JohnLangford/vowpal_wabbit.git**

Now we compile.

&gt; **cd vowpal_wabbit; make**

This should "just work", at least on Linux and OSX, and plausibly on any Posix platform.  If it fails, you most likely need to install the boost program options header or library.

Boost installation for Debian/Linux distributions use the command - "sudo apt-get install libboost-program-options-dev"
Boost installation for Mac OSX is a little bit more involved: 
   1) Download the source at http://sourceforge.net/projects/boost/files/boost/1.50.0/
   2) Execute the shell script with "./bootstrap.sh"
   3) Next run the command "bjam install"
   4) You should be good to go

Next, we test the result.

&gt; **make test**

Everything should pass.  If you see: 

_minor (<0.001) precision differences ignored_

That's ok.  One of the things we do for speed is use the -ffast-math option which implies that floating point arithmetic does not round exactly the same way on all platforms.

### A first dataset

Now, let's create a dataset.  Suppose we want to predict whether a house will require a new roof in the next 10 years.

&gt; **echo "0 | price:.23 sqft:.25 age:.05 2006**

**1 2 'second_house | price:.18 sqft:.15 age:.35 1976**

**0 1 0.5 'third_house | price:.53 sqft:.32 age:.87 1924" > house_dataset**

There is quite a bit going on here.  The first thing is a label, **0**, corresponding to no roof replacement.  The bar **|** separates label from features.   The features are **price**, **sqft**, **age**, and **2006**.  The first 3 features use an index derived from a hash function while the last feature uses index 2006 directly and has a default value of 1.

The next example, on the next line, is similar, but the label information is more complex.  The **2** is an importance weight which implies that this example counts twice.  Importance weights come up in many settings.  A missing importance weight defaults to 1. **'second_house** is the tag---it is used elsewhere to identify the example.

The third example is straightforward except there is a **0.5** in the label information.  This is an initial prediction.  Sometimes you have multiple interacting learning systems and want to be able to predict an offset rather than an absolute value.

Next, we learn:

&gt; **./vw house_dataset**

### VWs diagnostic information

There is a burble of diagnostic information which you can turn off with **--quiet**.  However, it's worthwhile to first understand it.

_using no cache_

This says you are not using a cache.  If you want to use multiple passes with --passes, you'll need to create a cache file with **-c** or **--cache_file housing.cache**.

_Reading from house_dataset_

There are many different ways to input data to VW.  We're just using a simple text file.  Alternatives include cache files (from previous VW runs), stdin, or a tcp socket.

_num sources = 1_

There is only one input file.  In general, you can specify multiple files.

_Num weight bits = 18_

Only 18 bits of the hash function will be used.  That's much more than necessary for this example.  You could adjust the number of bits using **-b bits**

_learning rate = 10_

The default learning rate is 10.  This is too aggressive when there is noise, but it's a good default because when there is noise you'll need a larger dataset and/or multiple passes to predict well.  On these larger datasets, our learning rate will by default decay towards 0 as we run through examples.  You can adjust with **-l rate**.

_initial_t = 1_

Learning rates often decay over time, and this specifies the initial time.  You can adjust with **--initial_t time**, although this is rarely necessary these days.

_power_t = 0.5_

This specifies the power on the learning rate decay.  You can adjust this **--power_t p** where p in the range [0,1] can make sense.  0 means the learning rate does not decay, which can be helpful when state tracking, while 1 is very aggressive, but plausibly optimal for IID datasets.  0.5 is a minimax optimal choice.

Next, there is a bunch of header information.  VW is going to print out some live diagnostic information. 

The first column, _average loss_ computes the <a href="http://hunch.net/~jl/projects/prediction_bounds/progressive_validation/coltfinal.pdf">progressive validation</a> loss.  The critical thing to understand here is that progressive validation loss deviates like a test set, and hence is a reliable indicator of success on the first pass over any dataset.

_since last_ is the progressive validation loss since the last printout.

_example counter_ tells you which example is printed out.  In this case, it's example 2.

_example weight_ tells you the sum of the importance weights of examples seen so far.  In this case it's 3, because the second example has an importance weight of 2.  

_current label_ tells you the label of the second example.

_current predict_ tells you the prediction (before training) on the current example.

_current features_ tells you how many features the current example has.  This is great for debugging, and you'll note that we have 5 features when you expect 4.  This exists, because VW has a default constant feature which is always added in.  Use **--noconstant** to turn it off.

VW prints a new line with an exponential backoff.  This is very handy, because often you can debug some problem before the learning algorithm finishes going through a dataset.

At the end, some more straightforward totals are printed.  The only mysterious one is:
_best constant_ and _best constant's loss_  These really only work if you are using squared loss, which is the default.  They compute the best constant's predictor and the loss of the best constant predictor.  If _average loss_ is not better than _best constant's loss_, something is wrong.  In this case, we have to few examples to generalize.

If we want to overfit like mad, we can simply use:
&gt; **./vw house_dataset -c --passes 25**

You'll notice that the _since last_ column drops to 0, implying that we have a perfect predictor, as is unsurprising with 3 examples having 5 features each.

### Saving your model (a.k.a. regressor) into a file

By default vw learns the weights of the features and keeps them in a memory vector. If you want to save the final regressor weights into a file, add **-f _filename_**:

&gt; **./vw house_dataset -c --passes 25 -f house.model**

### Getting predictions

We want to make predictions of course.  A simple way to do this is:
&gt; **./vw house_dataset -p /dev/stdout --quiet**

The first output _0.000000_ is for the first example.

The second output _0.000000 second_house_ is for the second example.  You'll notice the tag appears here, and this is the primary use of the tag.

The third output _1.000000 third_house_ is for the third example.  Clearly, some learning happened, because the prediction is now 1.

Note that in this last example, we predicted _while we learned_.

Alternatively, and more commonly, we would first learn and save the model into a file. Later we would predict using the saved model.

You may load a initial model we would add **-i house.model** (load initial model), and also **-t** which stands for test-only (do no learning):

&gt; **./vw -i house.model -t house_dataset -p /dev/stdout --quiet**

Obviously the results are different this time, because in the first prediction example, we learned as we went, and made only one pass over the data, whereas in the 2nd example we first loaded an over-fitted (25 pass) model and used our data-set _house_dataset_ for testing only.

### Auditing

When developing a new ML application, it's very helpful to debug.  VW can help a great deal with this using the --audit option.

&gt; **./vw house_dataset --audit --quiet**

Every example uses two lines.  The first line has the prediction, and the second line has one entry per feature.  Looking at the first feature, we see:

_^price:43641:0.23:0_

The _^price_ is the original feature.  If you use a namespace, it appears before _^_.  Namespaces are an advanced feature which allows you to group features and operate them in the core of VW with **-q** and **--ignore**.

_43641_ is the index of the feature, computed by a hash.

_0.23_ is the value of the feature.

_0_ is the value of the feature's weight.

Examining further, you'll notice that the feature 2006 uses the index 2006.  This means that you can freely use the hashing or precompute indicies as is common in for other machine learning programs.

### What's next?

The above only scratches the surface of VW.  You can learn with other loss functions, with other optimizers, with other representations, with clusters of 1000s of machines, and even do ridiculously fast active learning.  Your primary resources for understanding these are:

1. The presented tutorials at the top of the page.
2. The <a href="https://github.com/JohnLangford/vowpal_wabbit/wiki/Examples">examples</a>.
3. The <a href="https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments">commandline specification</a>.
4. The <a href="http://tech.groups.yahoo.com/group/vowpal_wabbit/">mailing list</a>.
5. You. If something isn't covered adequately, ask questions and consider creating an example or a test case.