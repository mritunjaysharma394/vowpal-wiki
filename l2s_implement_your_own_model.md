### Introduction

    Algorithm 1 MYRUN(X) % for sequence tagging, X: input sequence, Y: output
    1 Y <- []
    2 for t = 1 to LEN(X) do
    3 ref <- X[t].true_label
    4 Y[t] <- PREDICT(x=examples[t], y=ref , tag=t, condition=[1:t-1])
    5 LOSS(number of Y[t] != X[t].true_label)
    6 return Y

We first describe the programmable joint prediction paradigm. 
Algorithm 1 shows sample code for a generic sequence labeler under Hamming loss. The algorithm takes as input a sequence of examples (e.g., words), and predicts the meaning of each element in turn. The ith prediction depends on previous predictions (in this example, we use the library’s support for generating implicit features based on previous predictions). It uses two underlying library functions, `PREDICT(...)` and `LOSS(...)`. The function `PREDICT(...)` returns individual predictions based on `x` while `LOSS(...)` allows the declaration of an arbitrary loss for the point set of predictions. The `LOSS(...)` function and the reference `y` inputted to `PREDICT(...)` are only used in the training phase and it has no effect in the test phase.

This single library interface is sufficient for both testing and training, when augmented to include label “advice” from a training set as a reference decision (by the parameter y). This means that a developer only has to specify the desired test time behavior and gets training with minor additional decoration. The underlying system works as a credit assignment compiler to translate the user-specified decoding function and labeled data into updates of the learning model.

How can you learn a good PREDICT function given just an imperative program like Algorithm 1? It is essential to run the `MYRUN(...)` function (e.g., Algorithm 1)  many times, “trying out” different versions of `PREDICT(...)` to learn one that yields low `LOSS(...)`. The reduction framework implemented in VW automatically translate Algorithm 1 and training data into model updates. 


### Implementation of MYRUN(x) for the sequntial tagging task
Now, let's look at a C++ implementation of `MYRUN(x)` in VW for Algorithm 1: [search_sequencetask.cc](https://github.com/JohnLangford/vowpal_wabbit/blob/master/vowpalwabbit/search_sequencetask.cc)
 

    1  namespace SequenceTask
    2  {
    3  void initialize(Search::search& sch, size_t& /*num_actions*/, po::variables_map& /*vm*/)
    4  { sch.set_options( Search::AUTO_CONDITION_FEATURES  |    // automatically add history features (previous predictions) to our examples, please
    5                     Search::AUTO_HAMMING_LOSS        |    // use hamming loss on individual predictions -- we won't declare loss
    6                     Search::EXAMPLES_DONT_CHANGE     |    // we don't do any internal example munging
                   0);
    7  }
    8
    9  void run(Search::search& sch, vector<example*>& ec)
    10 { Search::predictor P(sch, (ptag)0);
    11   for (size_t i=0; i<ec.size(); i++)
    12   { action oracle     = ec[i]->l.multi.label;
    13     size_t prediction = P.set_tag((ptag)i+1).set_input(*ec[i]).set_oracle(oracle).set_condition_range((ptag)i, sch.get_history_length(), 'p').predict();
    14     if (sch.output().good())
    15       sch.output() << sch.pretty_label((uint32_t)prediction) << ' ';
    16   }
    17 }
    18 }

The `initialize()` function is called at the begging of the program and is used to initialize variables and to set flags (see the comments in code for the usage). The `run()` function defines the search space as Algorithm 1. 
`Search::search&` is a reference to the environment variables.  `vector<example*>& ec` is a sequence of examples. The `PREDICT()` function is provided through a class `Search::predictor`. `(ptag)` is used to decorate calls to the `PREDICT()` function. This is especially useful for graphical models, where a tag is effectively the “name” of a particular variable in the graphical model. In line 4, oracle is the true label and is used as a reference for the prediction during the training. Line 5 calls the `PREDICT()` function and obtain a local prediction. Note that, we do not specify the loss function in the code because it is enabled by a flag.


