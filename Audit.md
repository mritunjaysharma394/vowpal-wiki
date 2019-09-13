```
-a [ --audit ]              Turn on audit output for feature weight debugging
```

The audit option is useful for debugging and for accessing the features and values for each example as well as the values in VW's weight vector. The format depends on the mode VW is running on. The format used for the non-LDA case is:

`prediction tag (namespace^feature[\[offset\]]:hashindex:value:weight[@ssgrad] )*`

- `prediction` is VW's prediction on the example with tag `tag`
- Then there's a list of feature information:
    - `namespace` is the namespace where the feature belongs
    - `feature` is the name of the feature
        - `[offset]` is present when there are multiple learners in use. The weights for the zeroth learner are implicit when the `[offset]` is not shown, the first non-zero offset therefore starts from index 1. You'll notice the `hashindex` are sequential for the subsequent offsets.
    - `hashindex` is the position where it hashes
    - `value` is the value of the feature
    - `weight` is the current learned weight associated with that feature
    - `ssgrad` is the sum of squared gradients (plus 1) if adaptive updates are used