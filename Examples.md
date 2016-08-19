### Here are some examples of using vowpal-wabbit

 - [[RCV1 example]].  Here we use online gradient descent on TFIDF transformed features while taking advantage of a cache file.
 - [[Malicious URL example]].  A sequential binary classification problem with temporally correlated data.
 - VW has a test suite.  `less +/^__DATA test/RunTests` provides many little examples with input data-sets and expected reference outputs.
 - [[Matrix factorization example]]
 - Multi Class Examples
    - [[One Against All (oaa) multi class example]]
    - [[Cost Sensitive One Against All (csoaa) multi class example]]
    - [[Error Correcting Tournament (ect) multi class example]]
 - [[Predicting probabilities]] (for both binary and multi-class classification)
 - [[Weighted All Pairs (wap) multi class example]]
 - [[Truncated gradient descent example]]
 - [[Contextual Bandit Example]]
 - [[daemon example]] - how to run vw in daemon mode and get predictions

### Utilities
 - [[Using active_interactor.py]] an interactive way to interface with vw's active learning
 - [[Using vw-hypersearch]] a vw wrapper to search for "optimal" (argmin) parameters
 - [[using vw varinfo]] - a vw wrapper to inspect your training-set & model
 - [[Using vw regr]] - a script for aiding regression applications
 - [vw-top-errors: online learning debugging for better models](https://github.com/arielf/vowpal_wabbit/wiki/vw-top-errors:-online-learning-debugging-for-better-models)
 - [vw-lda - a friendly high-level wrapper for `vw` LDA](https://github.com/JohnLangford/vowpal_wabbit/blob/master/utl/vw-lda)

### Demos

The source tree has a sub-directory `demo` which includes somewhat more involved examples:

 - [DNA splice Site recognition dataset from the 2008 Pascal Large Scale Learning Challenge](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/dna)
 - [Using Searn for Entity Relation Recognition](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/entityrelation)
 - [MNIST handwritten digit recognition with `--nn`](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/mnist)
 - [low-ranked quadratic `--lrq` demo on the movielens data-set](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/movielens)
 - [`--normalized` effect on SGD online learning](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/normalized)
 - [Ability of `vw` to separate signal from artificially injected noise](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/random-noise)

### External sites examples

 - Zygmunt Zając's [FastML](http://fastml.com/blog/categories/vw/) has several examples of using Vowpal Wabbit in practice.
