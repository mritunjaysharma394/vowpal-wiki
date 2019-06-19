## _Work in progress_

Conditional Contextual Bandit (CCB) is an extension over Contextual Bandit (CB), where there are multiple slots in which an action can be chosen. There is a shared context, as well as features for each action and slot. The CCB reduction (`ccb_explore_adf`) calls into `cb_sample` (see [Sampling](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Conditional-Contextual-Bandit#sampling)) and then `cb_explore_adf`, and so it essentially reduces into sequential CB operations. There is an id assigned to each slot automatically in a reserved namespace. Interactions are then added for every namespace and interaction with this reserved namespace to learn the slot interactions. Rewards can be specified for any slot.

## Sampling
Since there are several calls to CB, in order for exploration to work each CB result must be sampled between each call. This is done automatically by the `cb_sample` reduction by potentially swapping the top action based on the pdf produced by the underlying `cb_explore_adf` call. Since exploration is done for every slot it is recommended to divide your epsilon by the number of slots in order to maintain a similar exploration amount.(`epsilon/num_slots`)

## Input Format
### VW text format
CCB format is a multi line example format with 3 different example/line types. Lines are identified by explicit types as part of the label. This is different to the previous implicit action example type.
```
ccb shared | ...
ccb action | ...
ccb slot [<chosen_action>:<cost>:<probability>[,<action>:<probability,...] [action_ids_to_include,...] | ...
```
- Both additional sections in the `slot` label are optional
- If `action_ids_to_include` is excluded then all actions are implicitly included
  - This is currently unsupported
- Action ids are zero indexed
- The list of action probability pairs in the first section is optional
  - If included, the entire collection of probabilities must sum to 1.0
- Test labels omit the entire `chosen_action:cost:probability`section

For example:
```
ccb shared | s_1 s_2
ccb action | a:1 b:1 c:1
ccb action | a:0.5 b:2 c:1
ccb slot | d:4
ccb slot 1:0.8:0.8,0:0.2 1,2 | d:7
```
### JSON format

### DSJSON format

## Label type
The label type of CCB is `CCB::label`. It contains the example type as one of `shared, action, slot`. An outcome if it was supplied (for labelled examples) and the currently `unused explicitly_included_actions`. The outcome is the cost associated with this example and all action probability pairs for this slot. You can see that this information directly corresponds to the information encoded in the text format section.

```C++
  struct conditional_contexual_bandit_outcome
  {
    float cost;
    ACTION_SCORE::action_scores probabilities;
  };

  enum example_type : uint8_t
  {
    unset = 0,
    shared = 1,
    action = 2,
    slot = 3
  };

  struct label {
    example_type type;
    conditional_contexual_bandit_outcome* outcome;
    v_array<uint32_t> explicit_included_actions;
};
```

## Prediction type
The prediction type for CCB is `CCB::decision_scores_t`, defined as follows:
```C++
typedef v_array<ACTION_SCORE::action_scores> decision_scores_t;
```

This prediction contains an array of action scores for every slot. Therefore, the chosen actions are the items in index 0 of every array. The rest of the contents of each of these arrays are the results of each CB call. The probability values can be used to determine if the top action is an explore or exploit action by observing if it is the largest and unique probability or a smaller and duplicated probability.