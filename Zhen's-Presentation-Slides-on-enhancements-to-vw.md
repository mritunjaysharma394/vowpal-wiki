[**Zhen Qin's slides**](https://github.com/JohnLangford/vowpal_wabbit/wiki/Zhen.pdf) describe Zhen's contributions to vowpal wabbit.  These include:

* `--invert_hash`: create readable model including the original feature names
* `--holdout_on/off` and `--holdout_period <N>` (default when multiple passes are used)
* `--early_terminate`: to stop multiple passes when held-out examples loss stops decreasing.
* `--bs <N>` bootstrap aggregation: model averaging for decreasing model variance and generalization loss
* `--top <k>`: reduction to top k recommendations
* `-q a:`, `-q ::`, `-q :a`: generalization for "any" in cubic feature crossing
* `--feature_mask`

See more details in the slides above.