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

# Update Rule Options
    --adaptive                       use adaptive, individual learning rates.
    --conjugate_gradient             use conjugate gradient based optimization
    --regularization arg (=0)        minimize weight magnitude
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
   
# Weight Options
    -b [ --bit_precision ] arg       number of bits in the feature table
    -i [ --initial_regressor ] arg   Initial regressor(s)
    -f [ --final_regressor ] arg     Final regressor
    --random_weights arg             make initial weights random
    --initial_weight arg (=0)        Set all weights to an initial value of 1.

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

  

 




  




  


 
  

 




 