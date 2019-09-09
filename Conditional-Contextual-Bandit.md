Conditional Contextual Bandit (CCB) is an extension over Contextual Bandit (CB), where there are multiple slots in which an action can be chosen. There is a shared context, as well as features for each action and slot. The CCB reduction (`ccb_explore_adf`) calls into `cb_sample` (see [Sampling](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Conditional-Contextual-Bandit#sampling)) and then `cb_explore_adf`, and so it essentially reduces into sequential CB operations. There is an id assigned to each slot automatically in a reserved namespace. Interactions are then added for every namespace and interaction with this reserved namespace to learn the slot interactions. Rewards can be specified for any slot.

CCB provides several improvements:
- Ability to learn from any slot, not just the top action
- Diversity in predictions
- Richer learning around slot dependent situations

## Usage
```
vw --ccb_explore_adf -d <data_file>
```

To use CCB, invoke VW with `--ccb_explore_adf`. All of the normal parameters are valid such as data file and prediction output file. The data file should be in the input format described below.

## Sampling
Since there are several calls to CB, in order for exploration to work each CB result must be sampled between each call. This is done automatically by the `cb_sample` reduction by potentially swapping the top action based on the pdf produced by the underlying `cb_explore_adf` call. Since exploration is done for every slot it is recommended to divide your epsilon by the number of slots in order to maintain a similar exploration amount.(`epsilon/num_slots`)

## Label Type
The label type of CCB is CCB::label. It contains the example type as one of shared, action, slot. An outcome if it was supplied (for labelled examples) and the currently unused explicitly_included_actions. The outcome is the cost associated with this example and all action probability pairs for this slot. You can see that this information directly corresponds to the information encoded in the text format section.

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

#### Example
```
ccb shared | s_1 s_2
ccb action | a:1 b:1 c:1
ccb action | a:0.5 b:2 c:1
ccb action | a:0.5 
ccb action | c:1
ccb slot  | d:4
ccb slot 1:0.8:0.8,0:0.2 0,1,3 | d:7
```
### JSON format
The JSON format is identical to the CB format, with the addition of `_slots` field. The `_slots` field contains all of the slot information similar to `_multi` for actions. It is an array of objects, where each object is one slot. `_a` can be supplied to specify the explicit included actions but this is not included in the reduction yet.

#### Example
```json
{
    "shared_feature": "feature",
    "_multi": [
        {
            "feature1": 3.0,
            "feature2": "name1"
        },
        {
            "feature1": 2.0,
            "feature2": "name2"
        },
        {
            "feature1": 3.0,
            "feature2": "name3"
        }
    ],
    "_slots": [
        {
            "size": "small",
            "_a": [
                0,
                2
            ]
        },
        {
            "size": "large"
        }
    ]
}
```
### DSJSON format
The DSJSON format for CCB is also similar to CB. The context field, `c`, is the same as for CB, where it is a valid object in VW JSON format. Therefore the slots are defined in the context field. The `_outcomes` field contains an object per slot. This specifies the cost associated with this slot, the outcomes reported for this slot as well as either an array or single value for both actions and probabilities.
#### Example
```json
{
  "Timestamp": "2019-08-27T12:45:53.6300000Z",
  "Version": "1",
  "c": {
    "GUser": {
      "shared_feature": "feature"
    },
    "_multi": [
      {
        "TAction": {
          "feature1": 3.0,
          "feature2": "name1"
        }
      },
      {
        "TAction": {
          "feature1": 3.0,
          "feature2": "name1"
        }
      },
      {
        "TAction": {
          "feature1": 3.0,
          "feature2": "name1"
        }
      }
    ],
    "_slots": [
      {
        "size": "small",
        "_a": [0, 2]
      },
      {
        "size": "large"
      }
    ]
  },
  "_outcomes": [
    {
      "_id": "62ddd79e-4d75-4c64-94f1-a5e13a75c2e4",
      "_label_cost": 0,
      "_a": [2, 0],
      "_p": [0.9, 0.1],
      "_o": []
    },
    {
      "_id": "042661c4-d433-4b05-83d6-d51a2d1c68be",
      "_label_cost": 0,
      "_a": [1, 0],
      "_p": [0.1, 0.9],
      "_o": [-1.0, 0.0]
    }
  ],
  "VWState": {
    "m": "da63c529-018b-44b1-ad0f-c2b13056832c/195fc8ed-224f-471a-90c4-d3e60b336f8f"
  }
}
```