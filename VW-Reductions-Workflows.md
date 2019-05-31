# CB
![CB Reductions Graph](https://user-images.githubusercontent.com/49209570/58733227-f131a480-83c1-11e9-93b9-aade1c6a386c.png)

This graph does not display the first possible reduction (which is not a part of the CB workflow), audit_regressor. audit_regressor's valid downstream reductions include any reduction that takes in a single-line example (nodes with incoming black lines).Additionally, this graph only displays valid reduction chains that will not result in program exceptions.

The valid entry nodes have not been marked as they haven't yet been mapped out.

## Known issues
Most reductions will attempt to force data to flow into required reductions; for all example multi-line examples must pass through the csldf reduction. However, these checks are not comprehensive, and several known gaps exist in their coverage.

The following reductions do not perform any validity checks
* audit_regressor
* expreplay_c
* warm_cb

Additionally, Search can bypass the csldf reduction if the cs_active option was provided.