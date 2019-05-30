## _Work in progress_

# Conditional Contextual Bandit

Conditional Contextual Bandit (CCB) is an extension over contextual bandits, where there are multiple slots in which an action can be chosen. There is a shared context, as well as features for each action and slot. The CCB reduction (`ccb_explore_adf`) calls into `cb_sample` (see [[Sampling]]) and then `cb_explore_adf`, and so it essentially reduces into sequential cb operations. In order to learn relationships between the slots for each action that is taken its features are inserted into a special history namespace for subsequent slots. Additionally, there is an automatic quartic interaction between shared, action, slot and history features.

## Sampling
## Input Format
#### Input format
CCB format is a multi line example format with 3 different example/line types. Lines are identified by explicit types as part of the label. This is different to the previous implicit action example type.
```
ccb shared | ...
ccb action | ...
ccb slot [<chosen_action>:<cost>:<probability>[,<action>:<probability,...] [action_ids_to_include,...] | ...
```
- Both additional sections in the `slot` label are optional
- If `action_ids_to_include` is excluded then all actions are implicitly included
- Action ids are zero indexed
- The list of action probability pairs in the first section is optional
  - If included, the entire collection of probabilities must sum to 1.0
- Test labels omit the cost section

For example:
```
ccb shared | s_1 s_2
ccb action | a:1 b:1 c:1
ccb action | a:0.5 b:2 c:1
ccb slot | d:4
ccb slot 1:0.8:0.8,0:0.2 1,2 | d:7
```