The contextual bandit learning algorithms in VW consist of two broad classes. The first class consists of settings where the maximum number of actions is known ahead of time, and the semantics of these actions stay fixed across examples. A more advanced setting allows potentially changing semantics per example. In this latter setting, the actions are specified via features, different features associated with each action. We refer to this setting as the ADF setting for action dependent features.

### Stationary set of actions with fixed semantics

When the number of actions is known ahead of time, suppose we have a file train.dat consisting of examples in the contextual bandit format discussed [here](https://github.com/JohnLangford/vowpal_wabbit/wiki/Logged-Contextual-Bandit-Example). Then we train VW on this data by invoking:

    ./vw -d train.dat --cb_explore 4 

Here `--cb_explore` indicates that we are training a contextual bandit algorithm, and its argument is the (fixed) maximum number of actions. Providing no further arguments uses defaults for the exploration strategy, which is epsilon-greedy (`epsilon = 0.05`). In general, we support a number of exploration algorithms defined below:

1. **Explore-first:** This simplest scheme takes in a parameter `tau`. On the first `tau` examples, we take each of the `k` actions with probability `1/k`. This data is then used to learn a good predictor, which is used to pick actions for the remaining examples. 

        Usage: ./vw -d train.dat --cb_explore 4 --first 2

2. **Epsilon-greedy:** This default exploration strategy trains a good policy online as it sees examples. At each example, the prediction of the current learned policy is taken with probability `1-epsilon`, and with the remaining `epsilon` probability an action is chosen uniformly at random. 

        Usage: ./vw -d train.dat --cb_explore 4 --epsilon 0.1

3. **Bagging Explorer:** This exploration rule is based on an ensemble approach. It takes in an argument `m` and trains `m` different policies. The policies differ by being trained on different random subsets of the data, with each example going to a subset of the `m` policies. The precise details of training are done using the bootstrap feature of VW described [here](https://github.com/JohnLangford/vowpal_wabbit/wiki/Zhen's-Presentation-Slides-on-enhancements-to-vw). The exploration algorithm picks the action from one of these policies uniformly at random. _This is a simple and effective approach that rules out obviously bad actions, while exploring amongst the plausibly good ones_ when the variation amongst the `m` policies is adequate. 

        Usage: ./vw -d train.dat --cb_explore 4 --bag 5

4. **Online Cover:** This is a theoretically optimal exploration algorithm based on this [paper](http://arxiv.org/abs/1402.0555). Like bagging, many different policies are trained, with the number specified as an argument `m`. Unlike bagging, the training of these policies is explicitly optimized to result in a diverse set of predictions, choosing all the actions which are not already learned to be bad in a given context. _This is the most sophisticated of the available options for contextual bandit learning_. 

        Usage: ./vw -d train.dat --cb_explore 4 --cover 3

### Changing action set or featurized actions

In many applications, the set of actions is richer in that it changes over time or we have rich information regarding each action. In either of these cases, it is a good idea to create features for every `(context, action)` pair rather than features associated only with context and shared across all actions. We call this setting ADF, and it builds on the `csoaa_ldf` learner that supports cost-sensitive classification with label-dependent features. An example dataset for this setting might look like:

    | a:1 b:0.5
    0:0.1:0.75 | a:0.5 b:1 c:2
     
    shared | s_1 s_2
    0:1.0:0.5 | a:1 b:1 c:1
    | a:0.5 b:2 c:1

Each example now spans multiple lines, with one line per action. For each action, we have the label information `(a,c,p)`, if known, as before. The action field `a` is ignored now since actions are identified by line numbers, and typically set to 0. The semantics of cost and probability are same as before. Each example is also allowed to specify the label information on precisely one action. A newline signals end of a multiline example. Additionally, we can specify contextual features which are shared across all actions in a line at the beginning of an example, which always has a tag `shared`, as in the second multiline example above. Since the shared line is not associated with any action, it should never contain the label information. 

The ADF learning mode is enabled by invoking VW as follows on the above file train_adf.dat:

    ./vw -d train_adf.dat --cb_explore_adf 

Note that unlike `--cb_explore`, we do not specify the number of actions in `--cb_explore_adf` as they are inferred from the number of lines in each example. We again have a number of choices for the exploration algorithms:

1. **Explore-first:** Same as before, it takes in a parameter `tau`. On the first `tau` examples, we take each of the actions with equal probability. This data is then used to learn a good predictor, which is used to pick actions for the remaining examples. 

        Usage: ./vw -d train.dat --cb_explore_adf --first 2

2. **Epsilon-greedy:** _This is the default strategy again._ At each example, the prediction of the current learned policy is taken with probability `1-epsilon`, and with the remaining `epsilon` probability an action is chosen uniformly at random. 

        Usage: ./vw -d train.dat --cb_explore_adf --epsilon 0.1

3. **Bagging Explorer:** Again same as before

        Usage: ./vw -d train.dat --cb_explore_adf --bag 5

4. **Softmax Explorer:** This is a different explorer, which uses the policy to not only predict an action but also predict a score indicating the quality of each action. A distribution is then created with the probability of action `a` being is proportional to `exp(lambda*score(x,a))`. Here `lambda` is a parameter, which leads to uniform exploration for `lambda = 0`, and stops exploring as `lambda` approaches infinity. _In general, this provides another nice knob for controlled exploration based on the uncertainty in the learned policy._

        Usage: ./vw -d train.dat --cb_explore_adf --softmax --lambda 10

_The online cover algorithm is not currently supported in the ADF mode._

### Conditional Contextual Bandit

#### Input format
CCB format is a multi line example format with 3 different example/line types. Lines are identified by explicit types as part of the label. This is different to the previous implicit action example type.
```
ccb shared | ...
ccb action | ...
ccb decision [<action>:<probability>:<cost>[,<action>:<probability,<action>:<probability>,...] [action_ids_to_include,...] | ...

```
- Both additional sections in the `decision` label are optional
- If `action_ids_to_include` is excluded then all actions are implicitly included
- Action ids are zero indexed
- The list of action probability pairs in the first section is optional
  - If included, the entire collection of probabilities must sum to 1.0

For example:
```
ccb shared | s_1 s_2
ccb action | a:1 b:1 c:1
ccb action | a:0.5 b:2 c:1
ccb decision | d:4
ccb decision 1:0.8:0.8,0:0.2 1,2 | d:7
```