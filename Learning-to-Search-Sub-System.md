The learning to search sub-system deals with problems that involve jointly predicting multiple mutual dependent output variables. To use the system, a developer has to define the search space as an imperative program (or use a predefined module in VW). Detailed usage  is described below. 

## Overview

### Joint Prediction
In joint prediction problems, the goal is creating good joint predictions. As an example, consider recognizing a
handwritten word where each character might be recognized in turn to understand the word. Here, it is commonly
observed that exposing information from related predictions (i.e. adjacent letters) aids individual predictions.
Furthermore, optimizing a joint loss function can improve the gracefulness of error recovery. 
Despite these advantages, it is empirically common to build independent predictors, because they are simpler to implement and faster to run. The goal of L2S sub-system is to make structured prediction algorithms as easy and fast as possible to program and to compute. Specifically, the L2S sub-system works as a compiler. A developer need only code desired test time behavior. Then, the compiler automatically translates the decoding function and labeled data into updates of the learning model, which learns to avoid compounding errors and makes a sequence of coherent decisions.

### System Architecture and Background
We first describe the system architecture in an abstract fashion. More details will be shown later. The system provides two library functions, `PREDICT(...)` and `LOSS(...)`. We explain the functionality of these two library functions using the following code for a part of speech tagger (or generic sequence labeler) under Hamming loss.

    Def MYRUN(X): % for sequence tagging, X: input sequence, Y: output
    1: Y ← []
    2: for t = 1 to LEN(X) do
    3: ref ← X[t].true_label
    4: Y[t] ← PREDICT(x=examples[t], y=ref , tag=t, condition=[1:t-1])
    5: LOSS(number of Y[t] != X[t].true_label)
    6: return Y

The algorithm takes as input a sequence of examples (e.g., words), and predicts the meaning of each element in
turn. The i-th prediction depends on previous predictions.  The function `PREDICT(...)` returns individual predictions based on x while `LOSS(...)` allows the declaration of an arbitrary loss for the point set of predictions. The
`LOSS(...)` function and the reference y inputted to `PREDICT(...)` are only used in the training phase and it has no effect in the test phase. This single library interface is sufficient for both testing and training, when augmented to include label “advice” from a training set as a reference decision (by the parameter y). This means that a developer only has to specify the desired test time behavior and gets training with minor additional decoration. As we mentioned, the underlying system works as a credit assignment compiler to translate the user-specified decoding function and labeled data into updates of the learning model. How can you learn a good PREDICT function given just an imperative program like Algorithm 1? It is essential to run the `MYRUN(...)` function many times, “trying out” different versions of `PREDICT(...)` to learn one that yields low `LOSS(...)`. Details can be found in the paper.

### References
For more information, please refer to the following tutorial and papers
* Tutorial: [Learning to Search](http://hunch.net/~l2s/)
* L2s System - [A Credit Assignment Compiler for Joint Prediction](http://arxiv.org/pdf/1406.1837v5.pdf)
* L2S approach - [LOLS](http://www.jmlr.org/proceedings/papers/v37/changb15.pdf)
* L2S appraoch - [AggreVate](http://arxiv.org/abs/1406.5979)
* L2S approach - [Dagger](http://www.jmlr.org/proceedings/papers/v15/ross11a/ross11a.pdf)
* L2S approach - [Searn](http://hunch.net/~jl/projects/reductions/searn/searn.pdf)

## Command-Line Usage 
VW provides predefined structured prediction models for various applications, including sequential tagging, dependency parsing, and an entity-relation joint prediction task. 

Please see [dependency parsing demo](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/dependencyparsing), [Entity-Relation extraction demo](https://github.com/JohnLangford/vowpal_wabbit/tree/master/demo/entityrelation), and [Test scripts](https://github.com/JohnLangford/vowpal_wabbit/blob/master/test/RunTests) for using the pre-defined structured prediction models.

## Implementing Your Own Structured Prediction Model
- For implementing your own model using the C++ library, please see this [tutorial](https://github.com/JohnLangford/vowpal_wabbit/wiki/Implement-Your-Own-Joint-Prediction-Model)
- For python implementation, please see this [ipython notebook](https://github.com/JohnLangford/vowpal_wabbit/blob/master/python/examples/Learning_to_Search.ipynb)
