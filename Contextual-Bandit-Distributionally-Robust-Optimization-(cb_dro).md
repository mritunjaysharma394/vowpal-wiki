`--cb_dro` activates distributionally robust optimization for contextual bandits.

## What is Distributionally Robust Optimization (DRO) ?

DRO is an approach to regularizing machine learning models via optimization against the worst-case distribution which is &ldquo;plausible&rdquo; given the empirical data[[1]](https://arxiv.org/abs/1810.08750).  The hope is better generalization.  Contrasted with Empirical Risk Minimization (ERM), which optimizes directly against the empirical distribution, DRO can be viewed as optimizing a lower bound.

Depending upon how &ldquo;plausible&rdquo; is defined in the previous paragraph, different DRO schemes result.  The one we use here is related to the material in [Empirical Likelihood for Contextual Bandits](https://arxiv.org/abs/1906.03323) but with important differences: 
* We utilize squared divergence rather than log to get an exact closed-form dual solution.
* We operate over a data stream rather than in batches.
* To track non-stationary we maintain the sufficient statistics for the dual solution with geometric decay (c.f., `--cb_dro_tau` below).

## How to use it

Specifying `--cb_dro` on the command line activates the code, which is intended to work in conjunction with any other ADF contextual bandit command line arguments (e.g., `--cb_adf` or `--cb_explore_adf` with any `--cb_type` etc.).  As an example, `RunTests -c 205` does the following

    % ./RunTests -c 205
    ...
    Test 205: (/usr/bin/timeout 80 ../build/vowpalwabbit/vw --cb_dro --cb_adf --rank_all -d train-sets/cb_adf_sm.data -p cb_dro_adf_sm.predict --cb_type sm) >/dev/null 2>cb_dro_adf_sm.stderr
    RunTests: test 205: stderr OK
    RunTests: test 205: cb_dro_adf_sm.predict OK

This is based upon test 188 which is the same command line without `--cb_dro`

    % ./RunTests -c 188
    ...
    Test 188: (/usr/bin/timeout 80 ../build/vowpalwabbit/vw --cb_adf --rank_all -d train-sets/cb_adf_sm.data -p cb_adf_sm.predict --cb_type sm) >/dev/null 2>cb_adf_sm.stderr
    RunTests: test 188: stderr OK
    RunTests: test 188: cb_adf_sm.predict OK

## Extra arguments

The defaults are reasonable but you might find better performance by tuning the parameters, especially `--cb_dro_tau`.

* `--cb_dro_tau float_between_0_and_1` (default: 0.999): This is the time constant for geometric decay of the sufficient statistics maintained by the algorithm.  The default value corresponds to a time constant of 1000 examples.  
* `--cb_dro_wmax positive_float` (default: inf): This is the largest possible importance weight you expect.  If your logging policy has a minimum probability `minp` for all actions then `wmax = 1/minp` and specifying it will tighten the estimates.  
* `--cb_dro_alpha float_between_0_and_1` (default: 0.05) This is the confidence level for the lower bound.  As alpha approaches 0 the confidence interval becomes extremely wide, and as alpha approaches 1 the optimization becomes ERM.

## Tips

* It doesn't always improve things.  YMMV, so experiment with the switch on and off.
* Increase the learning rate when using `--cb_dro`.  More aggressive optimization is required because the optimization objective is more conservative.  