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


  --active_learning                active learning mode
  --active_simulation              active learning simulation mode
  --active_mellowness arg (=8)     active learning mellowness parameter c_0. 
                                   Default 8
  --adaptive                       use adaptive, individual learning rates.
  -a [ --audit ]                   print weights of features
  -b [ --bit_precision ] arg       number of bits in the feature table
  --backprop                       turn on delayed backprop

  --conjugate_gradient             use conjugate gradient based optimization
  --regularization arg (=0)        minimize weight magnitude
  --corrective                     turn on corrective updates

   --decay_learning_rate arg (=1)   Set Decay factor for learning_rate between 
                                   passes
  -f [ --final_regressor ] arg     Final regressor
  --global_multiplier arg (=1)     Global update multiplier
  --delayed_global                 Do delayed global updates
  --hash arg                       how to hash the features. Available options:
                                   strings, all
  -h [ --help ]                    Output Arguments
  --version                        Version information
  --initial_weight arg (=0)        Set all weights to an initial value of 1.
  -i [ --initial_regressor ] arg   Initial regressor(s)
  --initial_t arg (=1)             initial t value
  --lda arg                        Run lda with <int> topics
  --lda_alpha arg (=0.100000001)   Prior on sparsity of per-document topic weig
                                   hts
  --lda_rho arg (=0.100000001)     Prior on sparsity of topic distributions
  --lda_D arg (=10000)             Number of documents
  --minibatch arg (=1)             Minibatch size, for LDA
  --min_prediction arg             Smallest prediction to output
  --max_prediction arg             Largest prediction to output
  --multisource arg                multiple sources for daemon input
  --noop                           do no learning

  --power_t arg (=0.5)             t power value
  --predictto arg                  host to send predictions to
  -l [ --learning_rate ] arg (=10) Set Learning Rate
  -p [ --predictions ] arg         File to output predictions to
  -q [ --quadratic ] arg           Create and use quadratic features
  --quiet                          Don't output diagnostics
  --random_weights arg             make initial weights random
  -r [ --raw_predictions ] arg     File to output unnormalized predictions to
  --sendto arg                     send example to <hosts>
  -t [ --testonly ]                Ignore label information and just test
  --thread_bits arg (=0)           log_2 threads
  --loss_function arg (=squared)   Specify the loss function to be used, uses 
                                   squared by default. Currently available ones
                                   are squared, hinge, logistic and quantile.
  --quantile_tau arg (=0.5)        Parameter \tau associated with Quantile loss
                                   . Defaults to 0.5
  --unique_id arg (=0)             unique id used for cluster parallel
  --sort_features                  turn this on to disregard order in which 
                                   features have been defined. This will lead 
                                   to smaller cache sizes
  --ngram arg                      Generate N grams
  --skips arg                      Generate skips in N grams. This in conjuncti
                                   on with the ngram tag can be used to generat
                                   e generalized n-skip-k-gram.


 