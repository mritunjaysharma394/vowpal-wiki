Running VW without any arguments produces a message which briefly explains each argument. Below arguments are grouped according to their function and each argument is explained in more detail.

# Input options
    -d [ --data ] arg                Example Set
    --daemon                         read data from port 39523
    --port arg                       port to listen on
    --passes arg (=1)                Number of Training Passes
    -c [ --cache ]                   Use a cache.  The default is <data>.cache
    --cache_file arg                 The location(s) of cache_file.
    --compressed                     use gzip format whenever appropriate. If a 
                                     cache file is being created, this option 
                                     creates a compressed cache file. A mixture 
                                     of raw-text & compressed inputs are 
                                     supported if this option is on

Raw training data can be passed to VW using the `-d` or `--data` options which expect a file name as an argument (specifying a file name that is not associated with any option also works), or via `stdin` or via a TCP/IP port if the `--daemon` option is specified. The port itself is specified by `--port` otherwise the default port 39523 is used. 

Parsing raw training data is slow so there are options to create or load data in VW's native format. Files containing data in VW's native format are called caches. It is important to understand that the exact contents of a cache file depend both on the input as well as the other options that are passed to VW during the creation of the cache. This implies that using the cache file with different options might cause VW to rebuild the cache. The easiest way to use a cache is to always specify the `-c` option. This way, VW will first look for a cache file and create it if it doesn't exist. To override the default cache file name use `--cache_file` followed by the file name.

`--compressed` can be used for reading gzipped raw training data, writing gzipped caches, and reading gzipped caches.

`--passes` takes as an argument the number of times the algorithm will cycle over the data (epochs). 

# Output Options
    -a [ --audit ]                   print weights of features  
    -p [ --predictions ] arg         File to output predictions to
    -r [ --raw_predictions ] arg     File to output unnormalized predictions to
    --sendto arg                     send example to <hosts>
    --quiet                          Don't output diagnostics
    --min_prediction arg             Smallest prediction to output
    --max_prediction arg             Largest prediction to output

The `-a` or `--audit` option is useful for debugging and for accessing the features and values for each example as well as the values in VW's weight vector. The format depends on the mode VW is running on. The format used for the non-LDA case is: 
    prediction tag (namespace^feature:hashindex:value:weight[@ssgrad] )*

`prediction` is VW's prediction on the example with tag `tag`. Then there's a list of feature information. `namespace` is the namespace where the feature belongs, `feature` is the name of the feature, `hashindex` is the position where it hashes, `value` is the value of the feature, `weight` is the current learned weight associated with that feature and finally `ssgrad` is the sum of squared gradients (plus 1) if adaptive updates are used.

# Manipulation Options
    -t [ --testonly ]                Ignore label information and just test
    -q [ --quadratic ] arg           Create and use quadratic features

    --sort_features                  turn this on to disregard order in which 
                                     features have been defined. This will lead 
                                     to smaller cache sizes

    --ngram arg                      Generate N grams
    --skips arg                      Generate skips in N grams. This in conjunction 
                                     with the ngram tag can be used to generate 
                                     generalized n-skip-k-gram.
    --hash arg                       how to hash the features. Available options:
                                     strings, all

`-t` makes VW run in testing mode. The labels are ignored so this is useful for assessing the generalization performance of the learned model on a test set.

`-q` is a very powerful option. It takes as an argument a pair of two letters. Its effect is to create interactions between the features of two namespaces. Suppose each example has a namespace `user` and a namespace `document`, then specifying `-q ud` will create an interaction feature for every pair of features `(x,y)` where `x` is a feature from the `user` namespace and `y` is a feature from the `document` namespace. If a letter matches more than one namespace then all the matching namespaces are used. In our example if there is another namespace `url` then interactions between `url` and `document` will also be modeled.

`--ngram` and `--skip` can be used to generate ngram features possibly with skips (a.k.a. don't cares). For example `--ngram 2` will generate (unigram and) bigram features by creating new features from features that appear next to each other, and  `--ngram 2 --skip 1` will generate (unigram, bigram, and) trigram features plus trigram features where we don't care about the identity of the middle token.

Unlike `--ngram` where the order of the features matters, `--sort_features` destroys the order in which features are presented and writes them in cache in a way that minimizes the cache size. `--sort_features` and `--ngram` are mutually exclusive

By default VW hashes string features and does not hash integer features. `--hash all` hashes all feature identifiers. This is useful if your features are integers and you want to use parallelization as it will spread the features almost equally among the threads or cluster nodes, having a load balancing effect.

# Update Rule Options
    --adaptive                       use adaptive, individual learning rates.
    --conjugate_gradient             use conjugate gradient based optimization
    --regularization arg (=0.001)    minimize weight magnitude
    --decay_learning_rate arg (=1)   Set Decay factor for learning_rate between passes
    --initial_t arg (=1)             initial t value
    --power_t arg (=0.5)             t power value
    -l [ --learning_rate ] arg (=10) Set Learning Rate
    --loss_function arg (=squared)   Specify the loss function to be used, uses 
                                     squared by default. Currently available ones
                                     are squared, hinge, logistic and quantile.
    --quantile_tau arg (=0.5)        Parameter \tau associated with Quantile loss
                                     Defaults to 0.5
    --minibatch arg (=1)             Minibatch size

`--adaptive` turns on an individual learning rate for each feature. These learning rates are adjusted automatically according to a data-dependent schedule. For details the relevant papers are
[Adaptive Bound Optimization for Online Convex Optimization](http://arxiv.org/abs/1002.4908)
and [Adaptive Subgradient Methods for Online Learning
and Stochastic Optimization](http://www.cs.berkeley.edu/~jduchi/projects/DuchiHaSi10.pdf). These learning rates give an improvement when the data have many features, but they can be slightly slower especially when used in conjunction with options that cause examples to have many non-zero features such as `-q` and `--ngram`. 

`--conjugate_gradient` uses a batch optimizer based on the nonlinear conjugate gradient method. To avoid overfitting, the objective that is being minimized is a tradeoff between empirical loss and the norm of the learned weight vector. `--regularization r`  controls this tradeoff. By default \(r=0.001\) so the penalty is \(0.001 ||w||_2^2\).   

`-l \(\lambda\)`, `--initial_t \(\tau\)`, `--power_t p`, and `--decay_learning_rate d` specify the learning rate schedule whose generic form in the \(k+1\)-th epoch is 
\[
\eta_t = \frac{\lambda d^k\tau^p}{(\tau+t)^p}
\]
There is no single rule for the best learning rate form. For standard learning from an i.i.d. sample, typically \(p \in \{0, 0.5, 1\}\), \(d \in (0.5,1]\) and \(\lambda,\tau\) are searched in a logarithmic scale. Very often, the defaults are reasonable and only the -l option needs to be explored. For other problems the defaults may be inadequate, e.g. for tracking p=0 is more sensible.

To specify a loss function use `--loss_function` followed by either `squared`, `logistic`, `hinge`, or `quantile`. The latter is parametrized by \(\tau \in (0,1)\) whose value can be specified by `--quantile_tau`. By default this is 0.5. For more information see [[Loss functions]]

To average the gradient from \(k\) examples and update the weights once every \(k\) examples use `--minibatch \(k\)`. Minibatch updates make a big difference for Latent Dirichlet Allocation.   
  
# Weight Options
    -b [ --bit_precision ] arg       number of bits in the feature table
    -i [ --initial_regressor ] arg   Initial regressor(s)
    -f [ --final_regressor ] arg     Final regressor
    --random_weights arg             make initial weights random
    --initial_weight arg (=0)        Set all weights to an initial value of 1.

VW hashes all features to a predetermined range \([0,2^b-1]\) and uses a fixed weight vector with \(2^b\) components. The argument of `-b` option determines the value of \(b\) which is 18 by default. Hashing the features allows the algorithm to work with very raw data (since there's no need to assign a unique id to each feature) and has only a negligible effect on generalization performance (see for example 
[Feature Hashing for Large Scale Multitask Learning](http://arxiv.org/abs/0902.2206).

Use the `-f` option to write the weight vector to a file named after its argument. For testing purposes or to resume training, one can load a weight vector using  the `-i` option. Specifying this option multiple times with different arguments will cause the initial weight vector to be the average of the ones specified at the command line.

By default VW starts with the zero vector as its hypothesis. The `--random_weights` option initializes with random weights. This is useful if starting with a zero weight would cause the algorithm to be stuck as could happen in a backpropagation implementation, or with VW's LDA implementation. It's also possible to initialize with a fixed value such as the all-ones vector using `--initial_weight`.


# Parallelization Options
    --thread_bits arg (=0)           log_2 threads
    --multisource arg                multiple sources for daemon input
    --predictto arg                  host to send predictions to

## Experimental Parallelization Options
    --unique_id arg (=0)             unique id used for cluster parallel
    --backprop                       turn on delayed backprop
    --corrective                     turn on corrective updates
    --global_multiplier arg (=1)     Global update multiplier
    --delayed_global                 Do delayed global updates

# Latent Dirichlet Allocation Options
    --lda arg                        Run lda with <int> topics
    --lda_alpha arg (=0.100000001)   Prior on sparsity of per-document topic weights
    --lda_rho arg (=0.100000001)     Prior on sparsity of topic distributions
    --lda_D arg (=10000)             Number of documents

# Active Learning Options
    --active_learning                active learning mode
    --active_simulation              active learning simulation mode
    --active_mellowness arg (=8)     active learning mellowness parameter c_0. 
                                     Default 8
# Other options
    --noop                           do no learning
    -h [ --help ]                    Output Arguments
    --version                        Version information

  

 




  




  


 
  

 




 