**Contextual Bandit algorithms in VW**

The contextual bandit learning algorithms in VW consist of two broad classes. The first class consists of settings where the number of actions is fixed and known ahead of time, and the semantics of these actions stay fixed across examples. A more advanced setting allows there to be variable number of actions, with potentially changing semantics per example. In this latter setting, the actions are specified via features, different features associated with each action. We refer to this setting as the ADF setting for action dependent features.

When the number of actions is known ahead of time, suppose we have a file train.dat consisting of examples in the contextual bandit format discussed [here](https://github.com/JohnLangford/vowpal_wabbit/wiki/Contextual-Bandit-Example). Then we train VW on this data by invoking:

    ./vw -d train.dat --cb_explore 4 

Here `--cb_explore` indicates that we are training a contextual bandit algorithm, and its argument is the (fixed) number of actions. Providing no further arguments uses defaults for the exploration strategy, which is epsilon-greedy (`epsilon = 0.05`). In general, we support a number of exploration algorithms defined below:

1. **Explore-first:** This simplest scheme takes in a parameter `tau`. On the first `tau` examples, we take each of the `k` actions with probability `1/k`. This data is then used to learn a good predictor, which is used to pick actions for the remaining examples. Usage:
    ./vw -d train.dat --cb_explore 4 --first 2

2. **Epsilon-greedy:** This default exploration strategy trains a good policy online as it sees examples. At each example, the prediction of the current learned policy is taken with probability `1-epsilon`, and with the remaining `epsilon` probability an action is chosen uniformly at random. Usage:
    ./vw -d train.dat --cb_explore 5 --epsilon 0.1

3. **Bagging Explorer:** This exploration rule is based on an ensemble approach. It takes in an argument `m` and trains `m` different policies. The policies differ by being trained on different random subsets of the data, with each example going to a subset of the `m` policies. The precise details of training are done using the bootstrap feature of VW described [here](https://github.com/JohnLangford/vowpal_wabbit/wiki/Zhen's-Presentation-Slides-on-enhancements-to-vw). The exploration algorithm picks the action from one of these policies uniformly at random. _This is a simple and effective approach that rules out obviously bad actions, while exploring amongst the plausibly good ones_ when the variation amongst the `m` policies is adequate. Usage:
    ./vw -d train.dat --cb_explore 4 --bag 5

4. **Online Cover:** This is a theoretically optimal exploration algorithm based on this [paper](http://arxiv.org/abs/1402.0555). Like bagging, many different policies are trained, with the number specified as an argument `m`. Unlike bagging, the training of these policies is explicitly optimized to result in a diverse set of predictions, choosing all the actions which are not already learned to be bad in a given context. _This is the most sophisticated of the available options for contextual bandit learning_. Usage:
    ./vw -d train.dat --cb_explore 4 --cover 3