Running VW with the`-h` or `--help` option produces a message which briefly explains each argument. Below arguments are grouped according to their function and each argument is explained in more detail.

# VW options
    -h [ --help ]                    Look here: http://hunch.net/~vw/ and
                                     click on Tutorial.
    --version                        Version information
    --random_seed arg                seed random number generator
    --noop                           do no learning

# Input options
    -d [ --data ] arg                Example Set
    --ring_size arg                  size of example ring
    --examples arg                   number of examples to parse
    --daemon                         read data from port 26542
    --port arg                       port to listen on
    --num_children arg (=10)         number of children for 
                                     persistent daemon mode
    --pid_file arg                   Write pid file in 
                                     persistent daemon mode
    --passes arg (=1)                Number of Training Passes
    -c [ --cache ]                   Use a cache.  The default is <data>.cache
    --cache_file arg                 The location(s) of cache_file.
    --compressed                     use gzip format whenever 
                                     possible. If a cache file 
                                     is being created, this 
                                     option creates a compressed
                                     cache file. A mixture of 
                                     raw-text & compressed 
                                     inputs are supported with autodetection.
    --no_stdin                       do not default to reading from stdin
    --save_resume                    save extra state so learning can be resumed
                                     later with new data

Raw training/testing data (in the proper plain text [[input format]]) can be passed to VW in a number of ways:

* Using the `-d` or `--data` options which expect a file name as an argument (specifying a file name that is not associated with any option also works);
* Via `stdin`;
* Via a TCP/IP port if the `--daemon` option is specified. The port itself is specified by `--port` otherwise the default port 26542 is used.  The daemon by default creates 10 child processes which share the model state, allowing answering multiple simultaneous queries.  The number of child processes can be controlled with `--num_children`, and you can create a file with the jobid using `--pid_file` which is later useful for killing the job.

Parsing raw data is slow so there are options to create or load data in VW's native format. Files containing data in VW's native format are called caches. It is important to understand that the exact contents of a cache file depend both on the input as well as the other options that are passed to VW during the creation of the cache. This implies that using the cache file with different options might cause VW to rebuild the cache. The easiest way to use a cache is to always specify the `-c` option. This way, VW will first look for a cache file and create it if it doesn't exist. To override the default cache file name use `--cache_file` followed by the file name.

`--compressed` can be used for reading gzipped raw training data, writing gzipped caches, and reading gzipped caches.

`--passes` takes as an argument the number of times the algorithm will cycle over the data (epochs). 

# Output options
    -a [ --audit ]                   print weights of features  
    -p [ --predictions ] arg         File to output predictions to
    -r [ --raw_predictions ] arg     File to output unnormalized predictions to
    --sendto arg                     send compressed examples to <host>
    --quiet                          Don't output diagnostics
    -P [ --progress ] arg            Progress update frequency. int: additive, float: multiplicative
    --min_prediction arg             Smallest prediction to output
    --max_prediction arg             Largest prediction to output

`-p /dev/stdout` is often a handy trick for seeing outputs. 

`-r` is rarely used.

`--quiet` shuts off the normal diagnostic printout of progress updates.

`--progress arg` changes the frequency of the diagnostic progress-update printouts.  If `arg` is an integer, the printouts happen every `arg` (fixed) interval, e.g: `arg` is 10, we get printouts at 10, 20, 30, ... Alternatively, if `arg` has a dot in it, it is interpreted as a floating point number, and the printouts happen on a multiplicative schedule: e.g. when `arg` is 2.0 (the default) progress updates will be printed on examples numbered: 1, 2, 4, 8, ..., 2^n

`--sendto` is used with another VW using `--daemon` to send examples and get back predictions from the daemon VW.

`--min_prediction` and `--max_prediction` control the range of the output prediction by clipping.  By default, it autoadjusts to the range of labels observed.  If you set this, there is no autoadjusting.

The `-a` or `--audit` option is useful for debugging and for accessing the features and values for each example as well as the values in VW's weight vector. The format depends on the mode VW is running on. The format used for the non-LDA case is: 

    `prediction tag (namespace^feature:hashindex:value:weight[@ssgrad] )*`

`prediction` is VW's prediction on the example with tag `tag`. Then there's a list of feature information. `namespace` is the namespace where the feature belongs, `feature` is the name of the feature, `hashindex` is the position where it hashes, `value` is the value of the feature, `weight` is the current learned weight associated with that feature and finally `ssgrad` is the sum of squared gradients (plus 1) if adaptive updates are used.

# Example Manipulation options
    -t [ --testonly ]                Ignore label information and just test
    -q [ --quadratic ] arg           Create and use quadratic features
    --cubic arg                      Create and use cubic features
    --ignore arg                     ignore namespaces beginning
                                     with character <arg>
    --keep arg                       keep namespaces beginning with 
                                     character <arg>
    --holdout_off                    no holdout data in multiple passes
    --holdout_period                 holdout period for test only, default 10
    --sort_features                  turn this on to disregard order in which 
                                     features have been defined. This will lead 
                                     to smaller cache sizes
    --noconstant                     Don't add a constant feature
    -C [ --constant ] arg            Set initial value of the constant feature to arg
                                     (Useful for faster convergence on data-sets
                                      where the label isn't centered around zero)
    --ngram arg                      Generate N grams
    --skips arg                      Generate skips in N grams. This in conjunction 
                                     with the ngram tag can be used to generate 
                                     generalized n-skip-k-gram.
    --hash arg                       how to hash the features. Available options:
                                     strings, all

`-t` and `--testonly` makes VW run in testing mode. The labels are ignored so this is useful for assessing the generalization performance of the learned model on a test set. This has the same effect as passing a 0 importance weight on every example.

`-q` is a very powerful option. It takes as an argument a pair of two letters. Its effect is to create interactions between the features of two namespaces. Suppose each example has a namespace `user` and a namespace `document`, then specifying `-q ud` will create an interaction feature for every pair of features `(x,y)` where `x` is a feature from the `user` namespace and `y` is a feature from the `document` namespace. If a letter matches more than one namespace then all the matching namespaces are used. In our example if there is another namespace `url` then interactions between `url` and `document` will also be modeled. The letter `:` is a wildcard to interact with all namespaces. `-q a:` (or `-q :a`) will create an interaction feature for every pair of features `(x,y)` where `x` is a feature from the namespaces starting with `a` and `y` is a feature from the all namespaces. `-q ::` would interact any combination of pairs of features.

`--cubic` is similar to `-q`, but it takes three letters as the argument, thus enabling interaction among the features of three namespaces. 

`--ignore` ignores a namespace, effectively making the features not there.  You can use it multiple times.

`--keep` keeps namespace(s) ignoring those not listed, it is a counterpart to `--ignore`.  You can use it multiple times. Useful for example to train a baseline using just a single namespace.

`--holdout_off` disables holdout validation for multiple pass learning. By default, VW holds out a (controllable default = 1/10th) subset of examples whenever --passes > 1 and reports the test loss on the print out. This is used to prevent overfitting in multiple pass learning. An extra `h` is printed at the end of the line to specify the reported losses are holdout validation loss, instead of progressive validation loss. 

`--holdout_period` specifies the period of holdout example used for holdout validation in multiple pass learning. For example, if user specifies `--holdout_period 5`, every one in 5 examples is used for holdout validation. In other words, 80% of the data is used for training.   

`--noconstant` eliminates the constant feature that exists by default in VW.

`--ngram` and `--skip` can be used to generate ngram features possibly with skips (a.k.a. don't cares). For example `--ngram 2` will generate (unigram and) bigram features by creating new features from features that appear next to each other, and  `--ngram 2 --skip 1` will generate (unigram, bigram, and) trigram features plus trigram features where we don't care about the identity of the middle token.

Unlike `--ngram` where the order of the features matters, `--sort_features` destroys the order in which features are presented and writes them in cache in a way that minimizes the cache size. `--sort_features` and `--ngram` are mutually exclusive

By default VW hashes string features and does not hash integer features. `--hash all` hashes all feature identifiers. This is useful if your features are integers and you want to use parallelization as it will spread the features almost equally among the threads or cluster nodes, having a load balancing effect.

# Update Rule options
    --sgd                            use regular/classic/simple stochastic
                                     gradient descent update (this is no longer
                                     the default since it is often sub-optimal)
    --adaptive                       use adaptive, individual learning rates
                                     (on by default)
    --normalized                     use per feature normalized updates.
                                     (on by default)
    --invariant                      use safe/importance aware updates
                                     (on by default)
    --exact_adaptive_norm            use a more expensive exact norm for adaptive
                                     learning rates.
                                     (superseded by --adaptive --normalized --invariant
                                      which are on by default, so no longer needed.)
    --conjugate_gradient             use conjugate gradient based optimization (option in bfgs)
    --bfgs                           use bfgs optimization
    --mem arg (=15)                  memory in bfgs
    --termination arg (=0.001)       Termination threshold
    --hessian_on                     use second derivative in line search
    --initial_pass_length arg        initial number of examples per pass
    --l1 arg (=0)                    l_1 lambda (L1 regularization)
    --l2 arg (=0)                    l_2 lambda (L2 regularization)
    --decay_learning_rate arg (=1)   Set Decay factor for learning_rate between passes
    --initial_t arg (=0)             initial t value
    --power_t arg (=0.5)             t power value
    -l [ --learning_rate ] arg (=0.5) Set Learning Rate
    --loss_function arg (=squared)   Specify the loss function to be used, uses 
                                     squared by default. Currently available ones
                                     are squared, hinge, logistic and quantile.
    --quantile_tau arg (=0.5)        Parameter \tau associated with Quantile loss
                                     Defaults to 0.5
    --minibatch arg (=1)             Minibatch size
    --feature_mask arg               Use existing regressor to determine which 
                                     parameters may be updated

`--exact_adaptive_norm` and `--adaptive` turns on an individual learning rate for each feature. These learning rates are adjusted automatically according to a data-dependent schedule. For details the relevant papers are
[Adaptive Bound Optimization for Online Convex Optimization](http://arxiv.org/abs/1002.4908)
and [Adaptive Subgradient Methods for Online Learning
and Stochastic Optimization](http://www.cs.berkeley.edu/~jduchi/projects/DuchiHaSi10.pdf). These learning rates give an improvement when the data have many features, but they can be slightly slower especially when used in conjunction with options that cause examples to have many non-zero features such as `-q` and `--ngram`. Of the two `--exact_adaptive_norm` used to be recommended in the past, it is now the default so there's no need to specify it.

`--bfgs` and `--conjugate_gradient` uses a batch optimizer based on LBFGS or nonlinear conjugate gradient method.  Of the two, `--bfgs` is recommended.  To avoid overfitting, you should specify `--l2`.  You may also want to adjust `--mem` which controls the rank of an inverse hessian approximation used by LBFGS. `--termination` causes bfgs to terminate early when only a  very small gradient remains.

`--initial_pass_length` is a trick to make LBFGS quasi-online.  You must first create a cache file, and then it will treat initial_pass_length as the number of examples in a pass, resetting to the beginning of the file after each pass.  After running `--passes` many times, it starts over warmstarting from the final solution with twice as many examples.  

`--hessian_on` is a rarely used option for LBFGS which changes the way a step size is computed.  Instead of using the inverse hessian approximation directly, you compute a second derivative in the update direction and use that to compute the step size via a parabolic approximation.

`--l1` and `--l2` specify the level (lambda values) of L1 and L2 regularization, and can be nonzero at the same time.  These values are applied on a per-example basis in online learning (sgd),

![\sum_i \left(L(x_i,y_i,w) + \lambda_1 |w|_1 + 1/2 \cdot \lambda_2 |w|_2^2\right)](http://i.imgur.com/H01IthR.png?1)

but on an aggregate level in batch learning (conjugate gradient and bfgs).

![\left(\sum_i L(x_i,y_i,w)\right) + \lambda_1 \|w\|_1 + 1/2 \cdot\lambda_2 \|w\|_2^2 .](http://i.imgur.com/Cnt5IXS.png)


`-l <lambda>`,

 `--initial_t <t_0>`,

 `--power_t <p>`,

 and 

`--decay_learning_rate <d>`

 specify the learning rate schedule whose generic form in the ![(k+1)^{th}](http://i.imgur.com/9RejDsg.png) epoch is 

![\eta_t = \lambda d^k \left(\frac{t_0}{t_0 + w_t}\right)^p](http://i.imgur.com/Zawclow.png)

where ![\(w_t\)](http://i.imgur.com/asQbRe4.png) is the sum of the importance weights of all examples seen so far (![\(w_t = t\)](http://i.imgur.com/sBi5pNW.png) if all examples have importance weight 1).

There is no single rule for the best learning rate form. For standard learning from an i.i.d. sample, typically ![ p \in \{0, 0.5, 1\}, \   d \in (0.5,1\] ](http://i.imgur.com/jozy5Xw.png)

 and ![\lambda,t_0](http://i.imgur.com/Is1EDyk.png) are searched in a logarithmic scale. Very often, the defaults are reasonable and only the -l option (![\lambda](http://i.imgur.com/6Hw5vOo.png)) needs to be explored. For other problems the defaults may be inadequate, e.g. for tracking \(p=0\) is more sensible.

To specify a loss function use `--loss_function` followed by either `squared`, `logistic`, `hinge`, or `quantile`. The latter is parametrized by ![\tau \in (0,1)\)](http://i.imgur.com/EHfPa0T.png) 
whose value can be specified by `--quantile_tau`. By default this is 0.5. For more information see [[Loss functions]]

To average the gradient from _k_ examples and update the weights once every _k_ examples use `--minibatch <k>`. Minibatch updates make a big difference for Latent Dirichlet Allocation and it's only enabled there.

<a name="feature_mask"></a>
`--feature_mask` allows to specify directly a set of parameters which can update, from a model file. This is useful in combination with `--l1`. One can use `--l1` to discover which features should have a nonzero weight and do `-f model`, then use `--feature_mask model` without `--l1` to learn a better regressor.


# Weight options
    -b [ --bit_precision ] arg                 number of bits in the feature table
    -i [ --initial_regressor ] arg             Initial regressor(s) to load into memory (arg is filename)
    -f [ --final_regressor ] arg               Final regressor to save (arg is filename)
    --random_weights arg                       make initial weights random
    --initial_weight arg (=0)                  Set all weights to an initial value of 1.
    --readable_model arg                       Output human-readable final regressor
    --invert_hash arg                          Output human-readable final regressor with feature names
    --save_per_pass                            Save the model after every pass over data
    --input_feature_regularizer arg            Per feature regularization input file
    --output_feature_regularizer_binary arg    Per feature regularization output file
    --output_feature_regularizer_text arg      Per feature regularization output file, in text

VW hashes all features to a predetermined range ![ \[0,2^b-1\] ](http://i.imgur.com/ce7NfyJ.png) and uses a fixed weight vector with ![2^b](http://i.imgur.com/SuQvjP3.png) components. The argument of `-b` option determines the value of \(b\) which is 18 by default. Hashing the features allows the algorithm to work with very raw data (since there's no need to assign a unique id to each feature) and has only a negligible effect on generalization performance (see for example 
[Feature Hashing for Large Scale Multitask Learning](http://arxiv.org/abs/0902.2206).

Use the `-f` option to write the weight vector to a file named after its argument. For testing purposes or to resume training, one can load a weight vector using  the `-i` option.

`--readable_model` is identical to `-f`, except that the model is output in a human readable format.

`--invert_hash` is similar to `--readable_model`, but the model is output in a more human readable format with feature names followed by weights, instead of hash indexes and weights. Note that running vw with `--invert_hash` is **much slower** and needs much **more memory**. Feature names are not stored in the cache files (so if `-c` is on and the cache file exists and you want to use `--invert_hash`, either delete the cache or use `-k` to do it automatically). For multi-pass learning (where `-c` is necessary), it is recommended to first train the model without `--invert_hash` and then do another run with no learning (`-t`) which will just read the previously created binary model (`-i my.model`) and store it in human-readable format (`--invert_hash my.invert_hash`).

`--save_per_pass` saves the model after every pass over the data.  This is useful for early stopping.

`--input_feature_regularizer`, `--output_feature_regularizer_binary`, `--output_feature_regularizer_text` are analogs of `-i`, `-f`, and `--readable_model` for batch optimization where want to do _per feature_ regularization.  This is advanced, but allows efficient simulation of online learning with a batch optimizer.

By default VW starts with the zero vector as its hypothesis. The `--random_weights` option initializes with random weights. This is often useful for symmetry breaking in advanced models.  It's also possible to initialize with a fixed value such as the all-ones vector using `--initial_weight`.

# Holdout options
    --holdout_off             no holdout data in multiple passes
    --holdout_period arg      holdout period for test only, default 10
    --holdout_after arg       holdout after n training examples, default off 
                              (disables holdout_period)
    --early_terminate arg     Specify the number of passes tolerated when holdout 
                              loss doesn't decrease before early termination,
                              default is 3

# Feature namespace options
    --hash arg                how to hash the features. Available options: strings, all
    --ignore arg              ignore namespaces beginning with character <arg>
    --keep arg                keep namespaces beginning with character <arg>
    --noconstant              Don't add a constant feature
    -C [ --constant ] arg     Set initial value of constant
    --sort_features           turn this on to disregard order in which features have 
                              been defined. This will lead to smaller cache sizes
    --ngram arg               Generate N grams
    --skips arg               Generate skips in N grams. This in conjunction with the
                              ngram tag can be used to generate generalized n-skip-k-gram.
    --affix arg               generate prefixes/suffixes of features; argument '+2a,-3b,+1'
                              means generate 2-char prefixes for namespace a, 3-char suffixes
                              for b and 1 char prefixes for default namespace
    --spelling arg            compute spelling features for a give namespace (use '_'
                              for default namespace)

# Latent Dirichlet Allocation options
    --lda arg                        Run lda with <int> topics
    --lda_alpha arg (=0.100000001)   Prior on sparsity of per-document topic weights
    --lda_rho arg (=0.100000001)     Prior on sparsity of topic distributions
    --lda_D arg (=10000)             Number of documents

The `--lda` option switches VW to LDA mode. The argument is the number of topics. `--lda_alpha` and `--lda_rho` specify prior hyperparameters. `--lda_D` specifies the number of documents. VW will still work the same if this number is incorrect, just the diagnostic information will be wrong. For details see [Online Learning for Latent Dirichlet Allocation](http://books.nips.cc/papers/files/nips23/NIPS2010_1291.pdf)

# Matrix Factorization options
    --rank arg (=0)                                   rank for matrix factorization.

`--rank` sticks VW in matrix factorization mode.  You'll need a relatively small learning rate like `-l 0.01`.

# Low Rank Quadratic options
    --lrq arg             use low rank quadratic features
    --lrqdropout          use dropout training for low rank quadratic features

# Binary 
    --binary              Reports loss as binary classification with -1,1 labels

# Multiclass options
    --oaa arg             Use one-against-all multiclass learning with <k> labels
    --ect arg             Use error correcting tournament with <k> labels
    --csoaa arg           Use one-against-all multiclass learning with <k> costs
    --wap arg             Use weighted all-pairs multiclass learning with <k> costs
    --csoaa_ldf arg       Use one-against-all multiclass learning with label
                          dependent features.  Specify singleline or multiline.
    --wap_ldf arg         Use weighted all-pairs multiclass learning with label
                          dependent features.  Specify singleline or multiline.
    --log_multi arg       Use online (decision) trees for <arg> classes (in log(arg) time).
                          See http://arxiv.org/pdf/1406.1822

# Stagewise Polynomial options
    --stage_poly                     stagewise polynomial features
    --sched_exponent arg1 ( = 1.0)   exponent controlling quantity of included features
    --batch_sz arg2 ( = 1000)        multiplier on batch size before including more features
    --batch_sz_no_doubling           batch_sz does not double

`--stage_poly` tells VW to maintain polynomial features: training examples are augmented with features obtained by producting together subsets (and even sub-multisets) of features.  VW starts with the original feature set, and uses `--batch_sz` and (and `--batch_sz_no_doubling` if present) to determine when to include new features (otherwise, the feature set is held fixed), with `--sched_exponent` controlling the quantity of new features.

`--batch_sz arg2` (together with `--batch_sz_no_doubling`), on a single machine, causes three types of behaviors: `arg2 = 0` means features are constructed at the end of every non-final pass, `arg2 > 0` with `--batch_sz_no_doubling` means features are constructed every `arg2` examples, and `arg2 > 0` without `--batch_sz_no_doubling` means features are constructed when the number of examples seen so far is equal to `arg2`, then `2*arg2`, `4*arg2`, and so on.  When VW is run on multiple machines, then the options are similar, except that no feature set updates occur after the first pass (so that features have more time to stabilize across multiple machines).  The default setting is `arg2 = 1000` (and doubling is enabled).

`--sched_exponent arg1` tells VW to include `s^arg1` features every time it updates the feature set (as according to `--batch_sz` above), where `s` is the (running) average number of nonzero features (in the input representation).  The default is `arg1 = 1.0`.

While care was taken to choose sensible defaults, the choices do matter.  For instance, good performance was obtained by using `arg2 = #examples / 6` and `--batch_sz_no_doubling`, however `arg2 = 1000` (without `--batch_sz_no_doubling`) was made default since `#examples` is not available to VW a priori.  As usual, including too many features (by updating the support to frequently, or by including too many features each time) can lead to overfitting.

# Active Learning options
    --active_learning                active learning mode
    --active_simulation              active learning simulation mode
    --active_mellowness arg (=8)     active learning mellowness parameter c_0. 
                                     Default 8

Given a fully labeled dataset, experimenting with active learning can be done with `--active_simulation`. All active learning algorithms need a parameter that defines the trade off between label complexity and generalization performance. This is specified here with `--active_mellowness`. A value of 0 means that the algorithm will not ask for any label. A large value means that the algorithm will ask for all the labels. If instead of `--active_simulation`, `--active_learning` is specified (together with `--daemon`) real active learning is implemented (examples are passed to VW via a TCP/IP port and VW responds with its prediction as well as how much it wants this example to be labeled if at all). If this is confusing, watch Daniel's explanation at the VW tutorial. The active learning algorithm is described in detail in [Agnostic Active Learning without Constraints](http://books.nips.cc/papers/files/nips23/NIPS2010_0363.pdf).

# Parallelization options
    --span_server arg               Location of server for setting up spanning tree
    --unique_id arg (=0)            unique id used for cluster parallel job
    --total arg (=1)                total number of nodes used in cluster parallel job
    --node arg (=0)                 node number in cluster parallel job

VW supports cluster parallel learning, potentially on thousands of nodes (it's known to work well on 1000 nodes) using the algorithms <a href="http://arxiv.org/abs/1110.4198">discussed here</a>.  

`--span_server` specifies the network address of a little server that sets up spanning trees over the nodes.

`--unique_id` should be a number that is the same for all nodes executing a particular job and different for all others.

`--total` is the total number of nodes.

`--node` should be unique for each node and range from {0,total-1}.

More details are in the cluster directory.

**Warning:** Make sure to disable the holdout feature in parallel learning using `--holdout_off`. Otherwise, some nodes might attempt to terminate earlier while others continue running. If nodes become out of sync in this fashion, usually a deadlock will take place. You can detect this situation if you see all your `vw` instances hanging with a CPU usage of 0% for a long time.

# Label dependent features
    --csoaa_ldf multiline|singleline
    --wap_ldf multiline|singleline

See http://www.umiacs.umd.edu/~hal/tmp/multiclassVW.html and http://groups.yahoo.com/neo/groups/vowpal_wabbit/conversations/topics/626

# Learning algorithm / reduction options
    -B [ --bootstrap ] arg    bootstrap mode with k rounds by online importance resampling
    --top arg                 top k recommendation
    --bs_type arg             bootstrap mode - currently 'mean' or 'vote'
    --autolink arg            create link function with polynomial d
    --cb arg                  Use contextual bandit learning with <k> costs
    --lda arg                 Run LDA with <int> topics
    --nn arg              Use sigmoidal feedforward network with <k> hidden units
    --cbify arg           Convert multiclass on <k> classes into a contextual
                          bandit problem and solve
    --search arg          use search-based structured prediction (SEARN or DAgger), arg=maximum action id or 0 for LDF

 




  




  


 
  

 




 